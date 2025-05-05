# **Distributed Inference and Serving**

[Distributed Inference and Serving — vLLM](https://docs.vllm.ai/en/stable/serving/distributed_serving.html)

This Note is based on the vLLM New Official Documentation: Inference and Serving - Distributed Inference and Serving

## Step Snap 1 [Understanding Parallel Computing Strategies in vLLM]

### Parallelism Types in vLLM

vLLM supports two main parallelization strategies for distributed inference:

1. **Tensor Parallelism**: This approach splits individual tensors (like weight matrices) across multiple GPUs. Yes, vLLM implements tensor parallelism using algorithms from **Megatron-LM**, which is a framework developed by NVIDIA specifically for training and serving large language models. Tensor parallelism works by partitioning the neural network layers horizontally - dividing attention heads or feed-forward network units across GPUs while maintaining the model's sequential structure.
2. **Pipeline Parallelism**: This divides the model vertically by assigning different layers to different devices. Each GPU processes a complete segment of the model, passing activations to the next stage in the pipeline.

### What is Megatron-LM?

Megatron-LM is a framework developed by NVIDIA that pioneered efficient techniques for training and deploying massive language models. Its tensor parallelism strategy is particularly important because it:

- Carefully partitions matrix multiplications across devices
- Minimizes communication overhead between GPUs
- Maintains computational efficiency at scale

### Distributed Runtime Management

For orchestrating these parallelism strategies across hardware, vLLM offers two different backends:

1. **Python Multiprocessing**:
    - Used by default in single-node deployments with sufficient GPUs for the requested `tensor_parallel_size`
    - Each GPU gets its own Python process
    - Simple setup with no additional dependencies
    - Efficient for single-machine environments
2. **Ray**:
    - Used when GPU requirements exceed single-node capacity or when running in Ray placement groups
    - Better suited for multi-node deployments
    - Provides advanced scheduling, fault tolerance, and resource management

To clarify an important point in your question: Both tensor parallelism and pipeline parallelism are strategies for distributing the model itself. The choice between Ray and Multiprocessing is about how to manage the distributed runtime environment - they are the "orchestrators" that coordinate how these parallelism strategies execute across your hardware.

Users can manually select which runtime (Ray or Multiprocessing) to use via the `distributed_executor_backend` parameter in the LLM class or the `--distributed-executor-backend` parameter for the API server. For multiprocessing, Ray installation is not required.

## Step Snap 2 [Understanding vLLM Distributed Execution Backends: Multiprocessing vs. Ray]

### Single-Node Deployment Options

When deploying vLLM on a single node, the execution backend choice depends on GPU availability and tensor parallelism configuration:

1. **Multiprocessing Mode**: By default, vLLM uses Python's multiprocessing when there are sufficient GPUs on a single node to satisfy the configured `tensor_parallel_size`. This approach doesn't require Ray installation.
2. **Ray Mode**: vLLM switches to Ray when either:
    - The node lacks sufficient GPUs for the requested `tensor_parallel_size`
    - The execution occurs within a Ray placement group

Users can manually select either backend through the `distributed_executor_backend` parameter in the LLM class or via the API server parameter `--distributed-executor-backend`. For the multiprocessing approach, Ray installation is not required.

### Python Multiprocessing Implementation in vLLM

Python's multiprocessing module serves as vLLM's default choice for single-node multi-GPU scenarios:

1. **Operating Principle**:
    - vLLM deploys model shards across multiple GPUs using separate processes
    - Each GPU corresponds to an independent Python process, effectively bypassing GIL limitations
    - Inter-process communication utilizes shared memory or pipe mechanisms
2. **Memory Architecture**:
    - Each process maintains its own memory space, reducing memory contention
    - Shared memory technology optimizes data transfer efficiency
    - Supports efficient tensor passing between processes
3. **Ideal Use Cases**:
    - Single-machine multi-GPU deployments (e.g., servers with 4/8 GPUs)
    - Scenarios where GPU resources satisfy tensor parallelism requirements
    - Environments not requiring complex distributed scheduling
4. **Key Advantages**:
    - Simple configuration without additional dependencies
    - Quick startup with minimal process creation overhead
    - Low inter-process communication latency, suitable for computation-intensive tasks

### Ray Framework in vLLM Applications

As a comprehensive distributed computing framework, Ray primarily serves these vLLM scenarios:

1. **Architectural Design**:
    - Based on the Actor model, with each task unit (like model shards) encapsulated as Actors
    - Utilizes Ray's scheduler for cross-node resource coordination
    - Supports elastic scaling and fault tolerance mechanisms
2. **Resource Management**:
    - Dynamically allocates computing resources (GPU, CPU, memory)
    - Supports granular resource request and allocation strategies
    - Implements affinity scheduling through placement groups to reduce network overhead
3. **Typical Applications**:
    - Multi-node distributed deployments (across multiple servers)
    - Single-node scenarios with insufficient GPU resources for tensor parallelism
    - Production environments requiring dynamic scaling or high availability
4. **Advanced Capabilities**:
    - Handles complex dependency management and workflow orchestration
    - Provides built-in failure recovery and task retry mechanisms
    - Integrates monitoring and performance analysis tools

## Step Snap 3 [How does vLLM handle tensor parallelism on a single machine with multiple GPUs?]

When running vLLM on a single machine with multiple GPUs where the configured `tensor_parallel_size` is less than or equal to the available GPU count, vLLM automatically uses Python's native multiprocessing module rather than Ray for distributed execution.

Despite using multiprocessing as the orchestration backend, vLLM still implements tensor parallelism (based on Megatron-LM algorithms) to distribute the model across GPUs. This tensor parallelism strategy horizontally partitions model layers—dividing attention heads and neural network parameters across GPUs while maintaining the model's structure.

Each GPU runs its own Python process, handling a specific portion of the model's computation. These processes coordinate through efficient inter-process communication mechanisms like shared memory. This approach provides an optimal balance between implementation simplicity and computational efficiency for single-node deployments, eliminating the need for Ray installation while still leveraging sophisticated model parallelism techniques.

## Step Snap 4 [Runtime: From Program Lifecycle Manager to AI Model Orchestration Conductor]

Imagine watching a magnificent symphony orchestra. The conductor waves their baton, ensuring each musician plays the right notes at the right time, coordinating the entire orchestra to produce beautiful music. In the computing world, a "runtime" plays a similar role to this conductor, but it directs program code rather than musicians.

### Origin and Evolution of the Concept

The "runtime" concept was initially very simple—it merely referred to the time period from when a program starts until it ends. Like saying "this program has a runtime of 5 minutes." But as computer science evolved, this concept gradually became richer:

- **Early Stage**: In early languages like C, runtime referred to a set of support libraries (like the C runtime library) that provided basic functions such as memory management and input/output.
- **Middle Stage**: By the Java era, runtime evolved into more complex environments like the JVM (Java Virtual Machine), which not only executed programs but also handled memory management, garbage collection, and just-in-time compilation.
- **Modern Stage**: In today's distributed computing environments, runtime has further developed into sophisticated software systems that coordinate work across multiple machines and cores.

### Runtime in vLLM: The Conductor of AI Models

In large language model inference frameworks like vLLM, the runtime's role becomes even more critical. Imagine a massive AI model too large to fit into a single GPU's memory. The runtime acts as a seasoned conductor that needs to:

- **Split the Model**: Just as a conductor assigns different parts of a symphony to different instrument groups, the runtime divides the model across multiple GPUs.
- **Coordinate Communication**: Ensure these model parts distributed across different GPUs or even different servers can efficiently exchange data, just as a conductor ensures coordination between violin and cello sections.
- **Optimize Execution**: Carefully arrange the computation order and data flow to maximize hardware resource utilization, similar to how a conductor controls the rhythm and dynamics of a piece.
- **Handle Batch Requests**: Efficiently package multiple user requests, just as a conductor ensures the entire orchestra can simultaneously serve an audience in a concert hall.

vLLM offers two "conductor" options:

1. **Ray**: This is an experienced international conductor capable of directing orchestra members (multiple servers) located around the world to play complex symphonies together. It provides fault tolerance (if one musician makes a mistake, the performance continues), dynamic resource scheduling (bringing in more musicians as needed), and distributed state management.
2. **Python Multiprocessing**: This is a local conductor specializing in directing multiple instrument groups (multiple GPUs) in a single concert hall (single machine). It's simple to set up, has low overhead, but cannot conduct orchestras across multiple venues.

At the lower level, these "conductors" use communication protocols like NCCL as their "conducting language" and shared memory or networks as their "conducting gestures" to ensure the entire AI model works harmoniously, like a perfectly performed symphony.

It's these invisible "conductors" that allow us to run massive AI models that couldn't otherwise execute on a single device, providing users with a smooth experience—just like we enjoy a perfectly performed symphony without needing to understand the complex conducting techniques behind it.

## Step Snap 5 [vLLM Distributed Inference: Single Node vs. Multi-Node Deployment]

### Single Node Deployment

When running vLLM on a single machine with multiple GPUs, you're operating in what's called a "single node" configuration. Think of this as having all your computational power under one roof:

vLLM supports distributed tensor-parallel and pipeline-parallel inference and serving. Currently, we support Megatron-LM's tensor parallel algorithm. We manage the distributed runtime with either Ray or python native multiprocessing. Multiprocessing can be used when deploying on a single node, multi-node inference currently requires Ray.

**Key Characteristics of Single Node Deployment:**

1. **Runtime Options:**
Multiprocessing will be used by default when not running in a Ray placement group and if there are sufficient GPUs available on the same node for the configured tensor_parallel_size, otherwise Ray will be used. This default can be overridden via the LLM class distributed_executor_backend argument or --distributed-executor-backend API server argument.
2. **Implementation Methods:**
    - Using Python API:
    To run multi-GPU inference with the LLM class, set the tensor_parallel_size argument to the number of GPUs you want to use. For example, to run inference on 4 GPUs:
    from vllm import LLM
    llm = LLM("facebook/opt-13b", tensor_parallel_size=4)
    output = llm.generate("San Francisco is a")
    - Using Command Line:
    To run multi-GPU serving, pass in the --tensor-parallel-size argument when starting the server. For example, to run API server on 4 GPUs:
    vllm serve facebook/opt-13b 
    --tensor-parallel-size 4
3. **Advanced Configuration:**
You can also additionally specify --pipeline-parallel-size to enable pipeline parallelism. For example, to run API server on 8 GPUs with pipeline parallelism and tensor parallelism:
vllm serve gpt2 
--tensor-parallel-size 4 
--pipeline-parallel-size 2

### Multi-Node Deployment

When a single machine doesn't have enough GPUs, you can scale horizontally across multiple machines in a "multi-node" configuration:

If a single node does not have enough GPUs to hold the model, you can run the model using multiple nodes. It is important to make sure the execution environment is the same on all nodes, including the model path, the Python environment.

**Key Steps for Multi-Node Deployment:**

1. **Setting Up the Cluster:**
The recommended way is to use docker images to ensure the same environment, and hide the heterogeneity of the host machines via mapping them into the same docker configuration.
2. **Head Node Configuration:**
The first step, is to start containers and organize them into a cluster. We have provided the helper script examples/online_serving/run_cluster.sh to start the cluster.
Pick a node as the head node, and run the following command:
bash run_cluster.sh 
vllm/vllm-openai 
ip_of_head_node 
--head 
/path/to/the/huggingface/home/in/this/node 
-e VLLM_HOST_IP=ip_of_this_node
3. **Worker Node Configuration:**
On the rest of the worker nodes, run the following command:
bash run_cluster.sh 
vllm/vllm-openai 
ip_of_head_node 
--worker 
/path/to/the/huggingface/home/in/this/node 
-e VLLM_HOST_IP=ip_of_this_node
4. **Running the Model:**
The common practice is to set the tensor parallel size to the number of GPUs in each node, and the pipeline parallel size to the number of nodes. For example, if you have 16 GPUs in 2 nodes (8 GPUs per node), you can set the tensor parallel size to 8 and the pipeline parallel size to 2:
vllm serve /path/to/the/model/in/the/container 
--tensor-parallel-size 8 
--pipeline-parallel-size 2

### Key Differences and Considerations

1. **Runtime Requirements:**
    - Single node: Can use either multiprocessing (simpler) or Ray
    - Multi-node: Must use Ray for coordination
2. **Communication Efficiency:**
To make tensor parallel performant, you should make sure the communication between nodes is efficient, e.g. using high-speed network cards like Infiniband.
3. **Model Distribution:**
Please make sure you downloaded the model to all the nodes (with the same path), or the model is downloaded to some distributed file system that is accessible by all nodes.
4. **Parallelism Strategy:**
    - Single node: Often just tensor parallelism is sufficient
    - Multi-node: Often combines tensor parallelism (within nodes) and pipeline parallelism (across nodes)
5. **Environment Consistency:**
Multi-node deployments require careful attention to ensuring identical environments across all machines.

By understanding these approaches, you can scale vLLM from a single powerful workstation all the way to a distributed cluster, allowing you to run increasingly larger language models as your hardware resources expand.

## Step Snap 6 [Understanding Model Parallelism Strategies in vLLM]

Let me explain the model parallelism strategies in vLLM, with special focus on when to use pipeline parallelism even on a single node.

### Model Parallelism Strategies in vLLM

You've correctly identified an important edge case! Let me clarify how these two parallelism strategies work:

**Tensor Parallelism (Horizontal Slicing)**

- **Division Method**: Slices horizontally across the model's hidden dimension, with each GPU processing different "slices" of the model
- **Requirements**: Typically needs to evenly divide model layers, as each GPU handles different parts of the same layer
- **Runtime Management**: Can use multiprocessing or Ray on a single node, but multi-node requires Ray
- **Typical Application**: Most effective when the model can be divided evenly by the number of GPUs

**Pipeline Parallelism (Vertical Slicing)**

- **Division Method**: Slices vertically across the model's layers, with each GPU processing different layers or stages of the model
- **Flexibility**: Supports uneven splits, can handle cases where even division isn't possible
- **Runtime Requirements**: Yes, even on a single node, using pipeline parallelism typically requires Ray as the distributed runtime
- **Special Use Case**: Particularly useful when model size cannot be evenly divided by the number of GPUs

### Single Node Edge Case

When a model fits into a single node with multiple GPUs, but the number of GPUs cannot evenly divide the model size, you can use pipeline parallelism, which splits the model along layers and supports uneven splits. In this case:

- Tensor parallel size should be set to 1
- Pipeline parallel size should be set to the number of GPUs
- This configuration typically requires Ray as the distributed runtime, even on a single node

For example, if you have a model that cannot be evenly divided among 4 GPUs, you could set:

```bash
vllm serve model_name \
    --tensor-parallel-size 1 \
    --pipeline-parallel-size 4 \
    --distributed-executor-backend ray
```

This approach allows vLLM to allocate model layers to GPUs in a non-uniform way, which is especially useful when the model has layers of different sizes or when you need to maximize the use of available GPUs but the model doesn't divide perfectly.

In summary, pipeline parallelism indeed provides a method for vertically splitting the model, which is valuable in cases where tensor parallelism isn't suitable, and it does require Ray as the runtime, even when running on just a single node.

## Step Snap 7 [InfiniBand vs NVLink]

**InfiniBand:**

- A networking technology designed for high-performance computing (HPC) environments
- Used primarily for node-to-node communication in clusters (connecting separate servers)
- Provides high bandwidth (up to 200 Gbps or more) and extremely low latency
- Used in supercomputers and data centers for connecting separate machines
- Requires special network cards and switches

**NVLink:**

- A proprietary NVIDIA interconnect technology primarily designed for GPU-to-GPU communication
- Used mainly within a single server to connect multiple GPUs together
- Provides extremely high bandwidth (up to 600 GB/s in newer generations)
- Built directly into NVIDIA GPUs and doesn't require separate network cards
- Optimized specifically for AI and HPC workloads that need to transfer large amounts of data between GPUs

**Similarities and Differences:**

- Both technologies provide high-bandwidth, low-latency connections for transferring large amounts of data
- NVLink is optimized for GPU-to-GPU communication within a server, while InfiniBand is designed for server-to-server communication in a cluster
- In a distributed AI system, you might use NVLink for communication between GPUs on the same node and InfiniBand for communication between different nodes
- Both technologies significantly reduce the communication bottleneck in distributed computing compared to standard Ethernet

**In the context of vLLM:**

- For tensor parallelism on a single node, NVLink provides optimal performance for GPU-to-GPU communication
- For tensor parallelism across multiple nodes, high-speed networks like InfiniBand become crucial to maintain performance
- Without these high-speed interconnects, the communication overhead can significantly impact the efficiency of tensor parallelism

So while both are high-speed interconnect technologies, they operate at different levels of the system architecture and are complementary rather than alternatives to each other in a full-scale distributed AI infrastructure.

## Step Snap 8 [GPU Interconnect Technologies: Beyond NVLink]

NVLink is the primary method for fast connections between GPUs, but there are several other implementation methods:

1. **PCIe (PCI Express)**:
    - This is the most basic connection method, supported by all GPUs
    - Much slower than NVLink, but universally available
    - Bandwidth typically limited to around 16GB/s (PCIe 4.0 x16)
    - Becomes a bottleneck for distributed training
2. **NVLink**:
    - NVIDIA proprietary technology offering the highest bandwidth
    - Latest versions provide up to 600GB/s bandwidth
    - Directly integrated into GPU chips
    - Supports GPU memory sharing and direct access
3. **NVSwitch**:
    - NVIDIA's dedicated switching chip
    - Allows multiple GPUs to form a fully connected network
    - Used in high-end servers like DGX systems
    - Enables NVLink-level connections between all GPUs
4. **AMD Infinity Fabric**:
    - AMD's equivalent technology
    - Used for connections between AMD Instinct accelerators
    - Provides high-bandwidth connections similar to NVLink
5. **SXM Connection**:
    - NVIDIA's server-grade GPU connection method
    - Connects GPUs directly to the motherboard instead of via PCIe slots
    - Provides higher power limits and better cooling
    - Typically used in conjunction with NVLink and NVSwitch
6. **Intel CXL (Compute Express Link)**:
    - A newer open standard
    - Based on PCIe but with additional capabilities
    - Supports memory sharing and coherency
    - May become an important standard for GPU interconnects in the future

In vLLM's tensor parallel computing, the speed of these connection technologies directly impacts performance because different parts of the model need to frequently exchange data. NVLink remains the best choice for GPU-to-GPU communication within the same node, while high-speed network technologies like InfiniBand are needed between nodes.

## Step Snap 9 [Why we need InfiniBand to speed up GPU communication? Isn't it for server connection rather than tensor parallelism?]

Yes, You've raised an excellent question about distributed vs. tensor parallel approaches. And here is the answer: 

Even in distributed multi-node setups, both tensor parallelism and pipeline parallelism can be used, sometimes in combination. They're not mutually exclusive.

### Tensor Parallelism Across Nodes

When vLLM documentation mentions needing high-speed interconnects like InfiniBand for tensor parallelism across nodes, it's because:

- Tensor parallelism splits the **same layer** of the model across different GPUs, which may be distributed across different nodes
- During forward and backward passes, these distributed layer portions need to frequently exchange activation values and gradients
- This exchange is intensive and frequent, requiring high bandwidth and low latency
- If the node-to-node connection is slow, this exchange becomes a severe performance bottleneck

For example, if you set `--tensor-parallel-size 16` across two nodes (8 GPUs per node), different parts of the same layer would be distributed across GPUs on different nodes, requiring efficient communication.

### Pipeline Parallelism Across Nodes

Pipeline parallelism is different:

- Pipeline parallelism assigns **different layers** of the model to different GPUs
- Communication mainly happens between layers, only passing activations at layer boundaries
- The communication volume is typically smaller than tensor parallelism, but still requires reasonable bandwidth

### Why Tensor Parallelism Across Nodes Needs High-Speed Networks

The communication pattern in tensor parallelism is "all-to-all," meaning all GPUs splitting the same layer need to exchange data with each other. This communication pattern especially requires efficient networking between nodes because:

1. Large volume: The amount of data to be exchanged is large
2. High frequency: Exchanges are needed at almost every computation step
3. All-to-all: Multiple GPUs across nodes need to exchange data with each other simultaneously

So even when using tensor parallelism in distributed deployments (not just pipeline parallelism), high-speed node-to-node connections remain crucial. This is why the documentation specifically mentions high-speed network technologies like InfiniBand.

In practice, many large-scale deployments use both types of parallelism simultaneously: tensor parallelism within nodes and pipeline parallelism between nodes, to achieve optimal performance.