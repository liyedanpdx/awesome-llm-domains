# Open Source LLMs

## Introduction

Open source Large Language Models (LLMs) have democratized access to advanced AI capabilities, enabling researchers, developers, and organizations to leverage, customize, and deploy state-of-the-art language models without the limitations of proprietary alternatives. This document provides an overview of key open source LLMs, their capabilities, and deployment considerations.

## Major Open Source Models

### LLaMA Family (Meta AI)

#### LLaMA 2
- Released: July 2023
- Sizes: 7B, 13B, 70B parameters
- Context: 4K tokens (extended versions available)
- License: Custom permissive for research and commercial use
- Notable features: Chat-tuned variants, improved safety compared to LLaMA 1

#### LLaMA 3
- Released: April 2024
- Sizes: 8B, 70B parameters (as of initial release)
- Context: 8K tokens standard
- License: Similar to LLaMA 2 with some restrictions
- Notable features: Improved reasoning, reduced hallucinations, stronger multilingual abilities

### Mistral AI Models

#### Mistral 7B
- Released: September 2023
- Size: 7B parameters
- Context: 8K tokens
- License: Apache 2.0
- Notable features: Strong performance despite smaller size, sliding window attention

#### Mixtral 8x7B
- Released: December 2023
- Architecture: Mixture of Experts (8 experts, 7B each)
- Context: 32K tokens
- License: Apache 2.0
- Notable features: Sparse MoE architecture, multilingual capabilities, strong reasoning

### Falcon (Technology Innovation Institute)

- Released: March/May 2023
- Sizes: 1B, 7B, 40B, 180B parameters
- Context: 2K-8K tokens
- License: Custom permissive license (TII Falcon License)
- Training: Trained on 1.5 trillion tokens of RefinedWeb and other datasets

### BLOOM (BigScience)

- Released: July 2022
- Sizes: 560M to 176B parameters
- Context: 2K tokens
- License: RAIL (Responsible AI License)
- Notable features: Built by a collaborative open science effort, 46 languages supported

## Specialized Open Source Models

### Code Models
- **CodeLlama**: Specialized for code generation and understanding
- **StarCoder/StarCoderBase**: Trained on permissively licensed source code
- **DeepSeek Coder**: Optimized for code completion and generation

### Multimodal Models
- **LLaVA**: Vision-language model combining LLaMA with visual encoders
- **BLIP-2**: Bootstrapping Language-Image Pre-training for unified vision-language understanding

### Small Efficient Models
- **TinyLlama**: 1.1B parameter model trained on 3 trillion tokens
- **Phi-2**: 2.7B parameter model with strong reasoning capabilities
- **Gemma**: Google's lightweight open models (2B and 7B variants)

## Deployment Considerations

### Quantization
Many open source models can be quantized to reduce their memory footprint:
- **GGUF Format**: Used by llama.cpp, supports various quantization levels (4-bit, 5-bit, 8-bit)
- **GPTQ**: Post-training quantization method for transformer models
- **AWQ**: Activation-aware weight quantization

### Frameworks and Tools
Popular frameworks for deploying open source LLMs:
- **Hugging Face Transformers**: Python library for working with pretrained models
- **llama.cpp**: C/C++ inference engine for efficient deployment
- **vLLM**: High-throughput and memory-efficient inference engine
- **text-generation-inference**: Optimized serving for Hugging Face models

## Fine-tuning and Adaptation

### Techniques
- **LoRA**: Low-Rank Adaptation for efficient fine-tuning
- **QLoRA**: Quantized Low-Rank Adaptation for memory-efficient fine-tuning
- **PEFT**: Parameter-Efficient Fine-Tuning methods

### Datasets
Popular datasets for fine-tuning open source models:
- **OpenOrca**: High-quality synthetic conversations aligned with GPT-4
- **Alpaca**: Stanford's dataset for instruction-following
- **LIMA**: Meta's "Less Is More for Alignment" dataset

## References

1. Meta AI. (2023). LLaMA 2: Open Foundation and Fine-Tuned Chat Models. https://ai.meta.com/llama/
2. Mistral AI. (2023). Mistral 7B. https://mistral.ai/news/announcing-mistral-7b/
3. TII. (2023). Falcon LLM. https://falconllm.tii.ae/
4. BigScience Workshop. (2022). BLOOM. https://bigscience.huggingface.co/blog/bloom 