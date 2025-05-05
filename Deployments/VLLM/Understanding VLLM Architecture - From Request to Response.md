# Understanding VLLM Architecture: From Request to Response

## Step Snap 1 [The Request Journey: High-Level Overview]

VLLM's architecture is designed for high-throughput, memory-efficient inference of large language models. Here's how it processes requests:

```
┌──────────────────┐     ┌───────────────┐     ┌────────────────┐     ┌──────────────┐
│                  │     │               │     │                │     │              │
│  User Request    │────▶│  LLM Engine   │────▶│   Scheduler    │────▶│    Worker    │
│  (API/Interface) │     │ (Orchestrator)│     │ (Queue Manager)│     │(GPU Executor)│
│                  │     │               │     │                │     │              │
└──────────────────┘     └───────────────┘     └────────────────┘     └──────────────┘
                                                       │                      │
                                                       ▼                      ▼
                                               ┌────────────────┐     ┌──────────────┐
                                               │                │     │              │
                                               │ Block Manager  │◀───▶│ Cache Engine │
                                               │ (Memory Blocks)│     │(Memory Alloc.)│
                                               │                │     │              │
                                               └────────────────┘     └──────────────┘

```

When a request arrives, it flows through these components:

- **Entry Points**: API server, CLI, or direct library calls
- **LLM Engine**: Converts requests into SequenceGroups and orchestrates processing
- **Scheduler**: Assigns priorities and manages request queues
- **Worker**: Executes the actual model computations on GPU
- **Block Manager & Cache Engine**: Handle memory allocation and KV cache management

The modular design allows VLLM to process multiple requests in parallel while efficiently utilizing GPU memory.

## Step Snap 2 [Core Components: Under the Hood]

Diving deeper into the key components and their interactions:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              LLM Engine                                  │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │ step()
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                               Scheduler                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐               │
│  │   Waiting    │    │   Running    │    │   Swapped    │               │
│  │    Queue     │───▶│    Queue     │◀──▶│    Queue     │               │
│  └──────────────┘    └──────────────┘    └──────────────┘               │
│               │               │                   │                      │
│               │               │                   │                      │
│               ▼               ▼                   ▼                      │
│        ┌──────────────────────────────────────────────────────┐         │
│        │                   Block Manager                       │         │
│        │   (Physical Blocks ↔ Logical Blocks Mapping)          │         │
│        └──────────────────────────────────────────────────────┘         │
└─────────────────────────────────┬─────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                                Worker                                    │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐   │
│  │   Model Runner   │───▶│  KV Cache Manager│───▶│  Output Processor│   │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘   │
│             │                      │                       │             │
│             ▼                      ▼                       ▼             │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                       Cache Engine                                │   │
│  │         (GPU Memory Management and Allocation)                    │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘

```

Key components and their roles:

1. **Scheduler (core/scheduler.py)**:
    - Maintains three queues: waiting, running, and swapped
    - Implements scheduling policies (FIFO by default)
    - Decides which SequenceGroups to process in each step
2. **Block Manager (core/block_manager.py)**:
    - Divides memory into equal-sized blocks
    - Maps logical blocks (sequence tokens) to physical memory blocks
    - Handles block allocation, deallocation, and swapping
3. **Worker (worker/worker.py)**:
    - Abstracts GPU computation
    - Manages model execution and KV cache
    - One worker typically corresponds to one GPU
4. **Cache Engine (worker/cache_engine.py)**:
    - Handles low-level memory allocation
    - Manages physical memory across devices (GPU and CPU)

## Step Snap 3 [The Step Function: VLLM's Heartbeat]

The `step()` function in LLM Engine is the core execution unit of VLLM, orchestrating all components:

```
┌─────────────────────────────────────────────────────────────────┐
│                         LLM Engine step()                        │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
                ▼                             ▼
┌───────────────────────────┐   ┌───────────────────────────┐
│      Scheduler step 1     │   │      Worker execute       │
│ (Schedule next sequences) │──▶│    (Model computation)    │
└───────────────────────────┘   └─────────────┬─────────────┘
                                              │
                                              ▼
                               ┌───────────────────────────┐
                               │     Scheduler step 2      │
                               │ (Process model outputs)   │
                               └─────────────┬─────────────┘
                                             │
                                             ▼
                               ┌───────────────────────────┐
                               │   Return outputs/tokens   │
                               │  (Back to user/API call)  │
                               └───────────────────────────┘

```

The step function:

1. **Planning Phase (Scheduler Step 1)**:
    - Determines which sequences to run in this iteration
    - Plans token swapping operations (in/out/copy)
    - May preempt/reorder sequences based on scheduling policy
2. **Execution Phase (Worker Execute)**:
    - Performs the actual model forward pass
    - Processes batched inputs efficiently
    - Manages attention computation with the KV cache
3. **Processing Phase (Scheduler Step 2)**:
    - Decodes model outputs
    - Updates scheduler state based on sampling results
    - Releases resources for completed requests
4. **Output Phase**:
    - Creates and returns generation results
    - Updates request status (complete/partial)

This step function executes repeatedly until all requests are fulfilled or timeout occurs.

## Step Snap 4 [Memory Management: VLLM's Secret Sauce]

VLLM's exceptional performance comes from its innovative memory management techniques:

```
┌───────────────────────────────────────────────────────────────────┐
│                      GPU Memory Organization                       │
│                                                                   │
│  ┌─────────────┐  ┌─────────────────────────────────────────────┐ │
│  │ Model Params│  │                  KV Cache                    │ │
│  │ (weights)   │  │                                             │ │
│  └─────────────┘  │  ┌──────┐ ┌──────┐ ┌──────┐      ┌──────┐  │ │
│  ┌─────────────┐  │  │Block │ │Block │ │Block │ .... │Block │  │ │
│  │ Activations │  │  │  1   │ │  2   │ │  3   │      │  N   │  │ │
│  └─────────────┘  │  └──────┘ └──────┘ └──────┘      └──────┘  │ │
│                   └─────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────┘
                                   ▲
                                   │
                                   ▼
┌───────────────────────────────────────────────────────────────────┐
│                           CPU Memory                               │
│                                                                   │
│  ┌─────────────────────────────────────────────────┐              │
│  │                   Swap Space                     │              │
│  │  ┌──────┐ ┌──────┐ ┌──────┐      ┌──────┐       │              │
│  │  │Block │ │Block │ │Block │ .... │Block │       │              │
│  │  │  X   │ │  Y   │ │  Z   │      │  M   │       │              │
│  │  └──────┘ └──────┘ └──────┘      └──────┘       │              │
│  └─────────────────────────────────────────────────┘              │
└───────────────────────────────────────────────────────────────────┘

```

VLLM's memory management innovations include:

1. **PagedAttention**:
    - Divides KV cache into fixed-size blocks
    - Enables non-contiguous memory allocation for sequences
    - Eliminates memory fragmentation issues
2. **Continuous Batching**:
    - Dynamically adds new requests to ongoing batches
    - Maintains high GPU utilization
    - Eliminates need to wait for full batches
3. **Block-level GPU-CPU Swapping**:
    - Moves less active sequences to CPU memory
    - Prioritizes active sequences in GPU memory
    - Efficiently handles context switching
4. **Prefix Caching**:
    - Reuses computation for shared prefix tokens
    - Optimizes performance for similar prompts
    - Implemented via the Evictor component

These memory management techniques allow VLLM to handle more concurrent requests and longer sequences than traditional inference engines.

## Step Snap 5 [Model Support: The Extensible Framework]

VLLM provides a flexible architecture to support various model types:

```
┌─────────────────────────────────────────────────────────────────┐
│                     model_executor                              │
│                                                                 │
│  ┌─────────────────────────┐    ┌─────────────────────────────┐ │
│  │         layers          │    │           models            │ │
│  │                         │    │                             │ │
│  │  ┌─────────────────┐    │    │  ┌─────────────────────┐    │ │
│  │  │ Attention       │    │    │  │ Llama               │    │ │
│  │  ├─────────────────┤    │    │  ├─────────────────────┤    │ │
│  │  │ MLP             │    │    │  │ Mistral             │    │ │
│  │  ├─────────────────┤    │    │  ├─────────────────────┤    │ │
│  │  │ Embedding       │────┼───▶│  │ Falcon              │    │ │
│  │  ├─────────────────┤    │    │  ├─────────────────────┤    │ │
│  │  │ Normalization   │    │    │  │ GPT-NeoX            │    │ │
│  │  ├─────────────────┤    │    │  ├─────────────────────┤    │ │
│  │  │ Quantized Layers│    │    │  │ Qwen                │    │ │
│  │  └─────────────────┘    │    │  └─────────────────────┘    │ │
│  └─────────────────────────┘    └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

```

The model support framework consists of:

1. **Layer Implementations (model_executor/layers/)**:
    - Optimized implementations of common model components
    - Special attention layers for PagedAttention
    - Support for various quantization methods
2. **Model Architectures (model_executor/models/)**:
    - Architecture-specific implementations
    - Maps HuggingFace models to VLLM-optimized versions
    - Supports parameter adaptation for quantized models
3. **Worker Abstraction**:
    - Provides a consistent interface for different model types
    - Handles differences in input/output processing
    - Manages tensor parallelism for multi-GPU execution

To add support for a new model architecture, developers typically need to:

- Create a new model implementation in model_executor/models/
- Implement any unique layers in model_executor/layers/
- Register the model with VLLM's model loader system

This extensible architecture has enabled VLLM to rapidly support a wide range of model families, including Llama, Mistral, Falcon, GPT-NeoX, and Qwen models.