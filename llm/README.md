# LLM 知识库

## llama.cpp 参数与推理性能

整理日期：2026-09-23。源码基线：`f5e85d43a048f3d5adefb4c5e29867d8077fba62`。

- [完整图解手册（HTML，推荐阅读）](llamacpp-parameters-handbook.zh-CN.html)：正文、全部附录、4 张内嵌流程图，可离线阅读。
- [完整手册（Markdown）](llamacpp-parameters-handbook.zh-CN.md)：便于检索、编辑和版本管理。
- [交互图解：上下文、并发与 KV 内存](llamacpp-kv-budget.html)：调整容量规划和并发路数，比较不同 KV 精度的理论体积。

### 分篇查阅

- [性能机制、调参顺序与案例](llamacpp-parameters-performance-guide.md)
- [公共参数全集：357 个注册定义、551 个参数拼写](llamacpp-all-parameters-catalog.md)
- [独立工具：benchmark、量化、GGUF 拆分、RPC](llamacpp-tool-parameters.md)
- [HTTP completion 请求参数](llamacpp-request-parameters.md)
- [性能机制的官方源码证据](llamacpp-parameter-performance-evidence.md)
- [参数审计数据（JSON）](llamacpp-parameter-catalog.json)

图形源文件位于 `llamacpp-handbook-assets/`。手册绑定上述源码版本；弃用项、已移除项和适用范围均已注明。数值案例为明确假设下的估算，本次整理未进行新的模型性能实测。
