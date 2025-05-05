# Tensor Parallelism vs Pipeline Parallelism

### Tensor Parallelism

Tensor Parallelism (TP) generally provides better performance for inference compared to Pipeline Parallelism. It works by splitting individual operations across multiple GPUs, with each GPU handling a portion of the model's parameters for each operation.

In vLLM, tensor parallelism is recommended when your model can fit in a single node with multiple GPUs. For example, if you have 4 GPUs in a single node, you would set the tensor parallel size to 4.

Benchmarking studies consistently show that tensor parallelism achieves higher throughput and lower latency compared to equivalent pipeline parallel configurations. For example, one comparison with LLaMa 13B showed that a TP=4 configuration achieved 6.4s p90th latency, while a PP=2, TP=2 configuration resulted in 10.2s p90th latency at the same request rate.

### Pipeline Parallelism

Pipeline parallelism distributes groups of model layers across different GPUs. The model's layers are grouped into continuous blocks forming stages, typically done vertically through the network's architecture.

Pipeline parallelism is primarily useful when your model is too large to fit in a single node, requiring you to use tensor parallelism together with pipeline parallelism. The tensor parallel size would be the number of GPUs in each node, while the pipeline parallel size is the number of nodes you want to use.

Pipeline parallelism is most effective in scenarios where high throughput is prioritized over low latency, such as large batch processing. It excels in multi-node setups where communication must occur without high-speed interconnects.

### Why Tensor Parallelism Is Generally Better for Inference

The primary challenge with tensor parallelism is communication overhead between GPUs. However, with high-speed interconnects like NVLink, this overhead is significantly reduced, making tensor parallelism even more effective.

On NVIDIA A100 GPUs with NVLink, tensor parallelism consistently outperforms pipeline parallelism in both vLLM and TensorRT-LLM. The high-speed interconnect reduces communication overhead associated with frequent tensor operations in TP configurations.

Pipeline parallelism can face challenges with the dynamic nature of LLM serving, making it difficult to eliminate computation bubbles entirely. Variability in input sizes can lead to uneven workload distribution, increasing delays especially if certain model segments are more computationally demanding.

In summary, for inference speed, tensor parallelism typically provides better performance than pipeline parallelism, especially when high-speed interconnects like NVLink are available. Pipeline parallelism is primarily useful when your model is too large to fit on a single node, requiring distribution across multiple nodes.