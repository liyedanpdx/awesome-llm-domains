# Qwen3-235B-A22B and Qwen3-30B-A3B

On April 29, 2025, Alibaba released its latest generation of open-source large language models, Qwen3, marking another significant milestone in the field of AI foundation models. This report delves into the advantages and key details of Qwen3, providing a comprehensive reference for developers, researchers, and enterprises.

30B-A3B = MoE with 30 billion parameters where 3 billion parameters are active (=A3B)

235B-A22B = MoE with 235 billion parameters where 22 billion parameters are active (=A22B)

### Background and Release Overview

According to today's official announcement, Qwen3 is the newest member of the Qwen series, launched on April 29, 2025, encompassing a range of model sizes from small to ultra-large. This release is seen as a milestone in the open-source large model domain, particularly for its breakthroughs in performance and cost-effectiveness.

### Model Advantages

Qwen3 stands out in several key areas:

1. **High Performance and Competitiveness**
    - In international benchmarks for coding, mathematical reasoning, and comprehensive language understanding, Qwen3 performs on par with top models like DeepSeek-R1, Grok-3, and Gemini-2.5-Pro. For instance, the flagship Qwen3-235B-A22B shows highly competitive results across multiple tests.
    - Even smaller models, such as Qwen3-4B, match the performance of Qwen2.5-72B-Instruct, demonstrating efficient parameter utilization.
2. **Hybrid Reasoning Mode**
    - Qwen3 innovatively introduces "thinking mode" and "non-thinking mode," allowing users to switch dynamically via the enable_thinking parameter (enabled by default) or commands like /think and /no_think.
    - Thinking mode is suited for complex problems, providing step-by-step reasoning, while non-thinking mode is for quick responses to simple tasks, enhancing the model's flexibility.
3. **Multi-Language Support**
    - Supports 119 languages and dialects, covering Indo-European, Sino-Tibetan, Afro-Asiatic, and more, greatly expanding global application scenarios. For example, it can be used in multilingual dialogue systems or cross-language content generation.
4. **Open-Source and Ecosystem Development**
    - All models are fully open-source under the Apache 2.0 license, supporting both research and commercial development. This not only lowers the barrier to entry but also fosters innovation and collaboration in the global developer community.
5. **Cost-Effectiveness and Deployment Efficiency**
    - MoE (Mixture of Experts) models significantly reduce training and inference costs. For example, Qwen3-30B-A3B activates only 30 billion parameters to achieve high performance, with inference costs reportedly reduced by 70%, ideal for resource-constrained environments.
    - Dense models like Qwen3-0.6B are suited for lightweight deployment, meeting the needs of edge devices.
6. **Enhanced Agent and Coding Capabilities**
    - Optimized for agent tasks and code generation, supporting MCP (Multi-Context Processing), recommended for use with [Qwen-Agent](https://github.com/QwenLM/Qwen-Agent) for intelligent agent development.

### Key Details

Below are the core technical details of Qwen3 models, summarized in a table for clarity:

| **Model Type** | **Parameter Scale** | **Context Length** | **Training Data** | **Open-Source Status** |
| --- | --- | --- | --- | --- |
| MoE Models | Qwen3-235B-A22B (235B total, 22B active)<br>Qwen3-30B-A3B (300B total, 30B active) | 128K | ~360T tokens | Apache 2.0 |
| Dense Models | Qwen3-32B, 14B, 8B, 4B, 1.7B, 0.6B | 32K (0.6B-4B)<br>128K (8B+) | ~360T tokens | Apache 2.0 |
- **Training Data and Performance**:
    - Qwen3 is trained on approximately 360T tokens, double that of Qwen2.5 (180T tokens), including knowledge-intensive and long-context data (Stage S1 >300T tokens, 4K context; S2 additional 50T tokens, knowledge-intensive; S3 extended to 32K context).
    - Performance: Qwen3-1.7B/4B/8B/14B/32B-Base models match the performance of Qwen2.5-3B/7B/14B/32B/72B-Base, respectively. MoE models achieve similar performance by activating only 10% of parameters (e.g., Qwen3-30B-A3B).
- **Architecture and Deployment**:
    - All models are decoder-only architectures, suitable for generation tasks.
    - Recommended deployment frameworks include SGLang (>=0.4.6.post1), vLLM (>=0.8.4), and local tools like Ollama, LMStudio, MLX, llama.cpp, KTransformers.
    - Usage Examples:
        - Hugging Face: Model name "Qwen/Qwen3-30B-A3B".
        - SGLang deployment: python -m sglang.launch_server --model-path Qwen/Qwen3-30B-A3B --reasoning-parser qwen3.
        - vLLM deployment: vllm serve Qwen/Qwen3-30B-A3B --enable-reasoning --reasoning-parser deepseek_r1.
        - Ollama local run: ollama run qwen3:30b-a3b.

### Application Scenarios and Future Potential

Qwen3's diverse model sizes and flexible deployment options make it suitable for various scenarios:

- **Lightweight Applications**: Qwen3-0.6B is ideal for edge devices, with a 32K context length for small tasks.
- **High-Performance Inference**: Qwen3-235B-A22B is suited for complex tasks like scientific computing, code generation, and multimodal processing.
- **Global Market**: Multi-language support gives it a competitive edge in international markets, especially for cross-language AI applications.

Moreover, Qwen3's open-source ecosystem (e.g., support on [Hugging Face](https://huggingface.co/collections/Qwen/qwen3-67dd247413f0e2e4f653967f) and [ModelScope](https://modelscope.cn/collections/Qwen3-9743180bdc6b48)) is expected to accelerate its adoption in the developer community.

### Conclusion

The release of Qwen3 not only showcases Alibaba's technological breakthroughs in AI but also injects new vitality into the global AI industry through its combination of open-source and cost-effectiveness. Its multi-language support, flexible reasoning modes, and high performance make it an ideal choice for developers, enterprises, and research institutions.

### Model Training
https://huggingface.co/Qwen/Qwen3-8B/discussions/3