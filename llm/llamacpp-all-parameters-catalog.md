# llama.cpp 完整公共参数注册表

版本：`f5e85d43a048f3d5adefb4c5e29867d8077fba62`。本表逐项覆盖 `common/arg.cpp` 的 **357 个注册定义**，共 **551 个不重复命令行拼写**。同名但按工具分别注册的参数保留两行；同一注册项的所有正向、反向及短别名放在同一行。各行编号是本表审计编号，不是 CLI 参数。

本表覆盖公共解析器以及在其中注册的工具专属选项；`llama-bench`、`llama-quantize` 等使用独立解析器的参数不在这 357 项里，需结合主指南的独立工具附录。编译时未构建某个工具或后端时，相关参数不能凭空提供该功能。

**适用工具读法：** `COMMON` 指继承公共参数的工具集合，并不保证每个工具实际使用每个字段；`DOWNLOAD` 不继承 COMMON。`CLI` 为 llama-cli，`COMPLETION` 为 llama-completion，`SERVER` 为 llama-server，`BENCH` 在此表指使用公共解析器的 llama-batched-bench，不能当成 llama-bench 的参数清单。其余大写标签保留源码枚举名称，通常对应同名工具；`MTMD` 为多模态 CLI。标记“排除”表示该工具不接受该项。过滤规则见[固定版本源码](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1425)。

**默认值与值格式：** 表中仅在明确处写默认值；其余默认由工具初始化、模型元数据、构建与自动适配共同决定，按“自动/该运行时帮助”为准，不能把某台机器的数字当全局默认。`N/F/P` 等为源码值占位符；“无值开关”不再跟 true/false，成对的正反向拼写用来开启或关闭。`--flag value` 与 `--flag=value` 是否等价以工具解析器为准。JSON 附件保存完整帮助表达式、环境变量与源码供核对。

**性能列读法：** 作用与适用范围直接来自每行链接的注册定义；性能列为结合计算与存储机制的定性分析，不是此硬件上的实测承诺。优化要区分加载、预填充 PP、逐 token 解码 TG、首 token 延迟 TTFT、总吞吐、峰值内存与质量。更多线程、更多 GPU、更多并发都不保证同时改善所有指标。

K/V 类型（第 113/114/292/293 行）：`f32, f16, bf16, q8_0, q4_0, q4_1, iq4_nl, q5_0, q5_1`，接受类型与当前后端实际可运行组合是两回事。见[类型枚举](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L304)。

推测类型（第 307 行）：`none, draft-simple, draft-eagle3, draft-mtp, draft-dflash, draft-dspark, ngram-simple, ngram-map-k, ngram-map-k4v, ngram-mod, ngram-cache`。见[算法名称注册](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/speculative.cpp#L33)。


## 帮助、终端与连接

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [1](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1440) | `-h`<br>`--help`<br>`--usage` | `无值开关` | COMMON, DOWNLOAD | 打印帮助后退出。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [2](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1447) | `--version` | `无值开关` | COMMON | 显示版本和构建信息。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [3](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1455) | `-cl`<br>`--cache-list` | `无值开关` | COMMON | 列出本地下载缓存中的模型。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [4](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1467) | `--completion-bash` | `无值开关` | COMMON | 输出 Bash 自动补全脚本。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [5](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1474) | `--server-base` | `URL` | CLI | CLI 连接指定已有服务器。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [6](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1481) | `--verbose-prompt` | `无值开关` | COMPLETION, CLI, EMBEDDING, RETRIEVAL | 生成前打印详细提示词。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [7](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1488) | `--display-prompt`<br>`--no-display-prompt` | `无值开关` | COMPLETION, CLI | 开关提示词显示。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [8](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1496) | `-co`<br>`--color` | `[on\|off\|auto]` | COMPLETION, CLI, SPECULATIVE, LOOKUP | 终端输出着色；on/off/auto，默认 auto。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |

## CPU 线程与调度

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [9](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1513) | `-t`<br>`--threads` | `N` | COMMON | 生成阶段 CPU 线程数；非正数使用硬件并发数。 | 影响 CPU 算力利用率与同步成本；过多线程可能受内存带宽或调度开销限制而变慢。 |
| [10](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1523) | `-tb`<br>`--threads-batch` | `N` | COMMON | 提示词与批处理 CPU 线程数；未指定时沿用生成线程数。 | 影响 CPU 算力利用率与同步成本；过多线程可能受内存带宽或调度开销限制而变慢。 |
| [11](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1533) | `-C`<br>`--cpu-mask` | `M` | COMMON | 生成线程 CPU 亲和性十六进制掩码。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [12](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1543) | `-Cr`<br>`--cpu-range` | `lo-hi` | COMMON | 生成线程 CPU 亲和性编号范围。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [13](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1553) | `--cpu-strict` | `<0\|1>` | COMMON | 是否严格按指定 CPU 放置线程。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [14](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1560) | `--prio` | `N` | COMMON | 生成线程/进程优先级：-1 低，0 普通，1 中，2 高，3 实时。 | 改变与其他进程竞争 CPU 时的调度延迟；不增加硬件算力。 |
| [15](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1570) | `--poll` | `<0...100>` | COMMON | 生成线程等待工作时的忙轮询程度；0 禁用。 | 忙轮询可减少唤醒延迟，但增加 CPU 占用和功耗。 |
| [16](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1577) | `-Cb`<br>`--cpu-mask-batch` | `M` | COMMON | 批处理线程 CPU 掩码；缺省继承生成设置。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [17](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1587) | `-Crb`<br>`--cpu-range-batch` | `lo-hi` | COMMON | 批处理线程 CPU 编号范围。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [18](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1597) | `--cpu-strict-batch` | `<0\|1>` | COMMON | 批处理线程严格放置开关。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [19](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1604) | `--prio-batch` | `N` | COMMON | 批处理线程优先级：0 至 3。 | 改变与其他进程竞争 CPU 时的调度延迟；不增加硬件算力。 |
| [20](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1614) | `--poll-batch` | `<0\|1>` | COMMON | 批处理线程轮询开关；缺省继承生成设置。 | 忙轮询可减少唤醒延迟，但增加 CPU 占用和功耗。 |

## 查询缓存、上下文与批处理

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [21](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1621) | `-lcs`<br>`--lookup-cache-static` | `FNAME` | LOOKUP, SERVER | 静态 n-gram 查询缓存文件；生成时不更新。 | 匹配重复片段可减少串行目标模型步骤；命中不足时查找和验证可能成为额外成本。 |
| [22](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1628) | `-lcd`<br>`--lookup-cache-dynamic` | `FNAME` | LOOKUP, SERVER | 动态 n-gram 查询缓存文件；生成时更新。 | 匹配重复片段可减少串行目标模型步骤；命中不足时查找和验证可能成为额外成本。 |
| [23](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1635) | `-c`<br>`--ctx-size` | `N` | COMMON | 上下文容量；0 从模型读取。 | 决定可用上下文与 KV 容量；容量增加通常增大内存，实际长历史增加注意力成本。 |
| [24](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1646) | `--kv-unified-per-slot` | `N` | SERVER | 每个并行槽位的上下文上限；未显式设置 -c 时共享 KV 池按 槽位数×此值 配置。 | 决定可用上下文与 KV 容量；容量增加通常增大内存，实际长历史增加注意力成本。 |
| [25](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1654) | `-n`<br>`--predict`<br>`--n-predict` | `N` | COMMON | 最大生成 token 数；-1 不限；部分工具支持 -2 生成至上下文满。 | 直接改变生成 token 总量及总等待时间；不能据此判断每 token 速度。 |
| [26](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1665) | `-b`<br>`--batch-size` | `N` | COMMON | 逻辑批大小上限；决定一次向解码器提交多少 token。 | 较大逻辑批允许更多提示词/并发 token 一起处理；受微批、上下文与内存约束。 |
| [27](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1672) | `-ub`<br>`--ubatch-size` | `N` | COMMON | 物理微批大小上限；决定实际计算分块。 | 较大微批常提高 GPU 利用率与预填充吞吐，但提高临时计算缓冲区峰值。 |
| [28](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1679) | `--keep` | `N` | COMMON | 上下文移位时保留初始提示词的 token 数；-1 全保留。 | 影响上下文移位及重算成本、保留信息和长对话行为。 |
| [29](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1686) | `--swa-full` | `无值开关` | COMMON | 为滑动窗口注意力分配完整大小 SWA 缓存。 | 完整 SWA 缓存消耗更多内存；缓存范围改变可复用历史能力。 |
| [30](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1694) | `-ctxcp`<br>`--ctx-checkpoints`<br>`--swa-checkpoints` | `N` | SERVER, CLI | 每槽位最多保存的上下文检查点数。 | 更多检查点增加状态存储，可能减少回退后的重算。 |
| [31](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1702) | `-cms`<br>`--checkpoint-min-step` | `N` | SERVER | 上下文检查点之间最小 token 间隔；0 无最小间隔。 | 间隔更小可更精细回退，但增加检查点维护与存储成本。 |
| [32](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1712) | `-cram`<br>`--cache-ram` | `N` | SERVER, CLI | RAM 中提示词缓存大小上限，单位 MiB；-1 不限，0 禁用。 | 使用更多 RAM 保存状态可减少重复预填充；受命中率与状态复制开销影响。 |
| [33](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1720) | `-kvu`<br>`--kv-unified`<br>`-no-kvu`<br>`--no-kv-unified` | `无值开关` | SERVER, PERPLEXITY, BATCHED, BENCH, PARALLEL | 开关所有序列共用的统一 KV 缓冲区；槽位数自动时默认启用。 | 改变多序列 KV 容量共享和分配，影响并发容量与显存利用率。 |
| [34](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1728) | `--cache-idle-slots`<br>`--no-cache-idle-slots` | `无值开关` | SERVER | 新任务到来时将空闲槽位存入提示词缓存，并在统一 KV 模式下清空；要求启用 cache-ram。 | 使用更多 RAM 保存状态可减少重复预填充；受命中率与状态复制开销影响。 |
| [35](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1736) | `--context-shift`<br>`--no-context-shift` | `无值开关` | COMPLETION, CLI, SERVER, IMATRIX, PERPLEXITY | 开关长时间生成时的上下文移位。 | 影响上下文移位及重算成本、保留信息和长对话行为。 |
| [36](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1744) | `--chunks` | `N` | IMATRIX, PERPLEXITY, RETRIEVAL | 最大处理文本块数；-1 全部。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [37](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1751) | `-fa`<br>`--flash-attn` | `[on\|off\|auto]` | COMMON | Flash Attention 开关：on/off/auto。 | 融合/分块注意力常降低中间矩阵流量及内存，尤其长上下文；收益依赖后端和模型支持。 |

## 输入、计时与会话

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [38](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1766) | `-p`<br>`--prompt` | `PROMPT` | COMMON；排除 SERVER | 初始生成提示词。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [39](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1773) | `-sys`<br>`--system-prompt` | `PROMPT` | COMPLETION, CLI, DIFFUSION, MTMD | 系统提示词；是否生效取决于聊天模板。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [40](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1780) | `--perf`<br>`--no-perf` | `无值开关` | COMMON | 开关 libllama 内部性能计时。 | 改变计时采集或展示；主要用于测量，可能有少量观测开销。 |
| [41](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1789) | `--show-timings`<br>`--no-show-timings` | `无值开关` | CLI | 开关每次回答后的计时信息显示。 | 改变计时采集或展示；主要用于测量，可能有少量观测开销。 |
| [42](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1797) | `-f`<br>`--file` | `FNAME` | COMMON；排除 SERVER | 从文件读提示词。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [43](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1809) | `-sysf`<br>`--system-prompt-file` | `FNAME` | COMPLETION, CLI, DIFFUSION | 从文件读系统提示词。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [44](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1819) | `--in-file` | `FNAME` | IMATRIX | 输入文件列表，以逗号分隔。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [45](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1832) | `-bf`<br>`--binary-file` | `FNAME` | COMMON；排除 SERVER | 从二进制文件读提示词。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [46](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1848) | `-e`<br>`--escape`<br>`--no-escape` | `无值开关` | COMMON | 开关换行、制表等转义序列处理。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [47](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1856) | `-ptc`<br>`--print-token-count` | `N` | COMPLETION | 每 N 个 token 打印一次 token 计数。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [48](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1863) | `--prompt-cache` | `FNAME` | COMPLETION | 提示词状态缓存文件，用于减少后续启动的重复预填充。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [49](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1870) | `--prompt-cache-all` | `无值开关` | COMPLETION | 也将用户输入和生成结果保存到提示词状态缓存。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [50](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1877) | `--prompt-cache-ro` | `无值开关` | COMPLETION | 只读取提示词状态缓存，不写回。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [51](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1884) | `-r`<br>`--reverse-prompt` | `PROMPT` | COMPLETION, CLI, SERVER | 遇到指定反向提示词时停止生成并交还控制。 | 直接改变生成 token 总量及总等待时间；不能据此判断每 token 速度。 |
| [52](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1891) | `-sp`<br>`--special` | `无值开关` | COMPLETION, CLI, SERVER | 允许输出特殊 token。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [53](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1898) | `-cnv`<br>`--conversation`<br>`-no-cnv`<br>`--no-conversation` | `无值开关` | COMPLETION | 开关会话模式；有聊天模板时默认自动开启，启用交互且不显示特殊 token 及前后缀。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [54](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1909) | `-st`<br>`--single-turn` | `无值开关` | COMPLETION, CLI | 会话仅执行一轮后退出。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [55](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1918) | `-i`<br>`--interactive` | `无值开关` | COMPLETION | 开启交互模式。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [56](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1925) | `-if`<br>`--interactive-first` | `无值开关` | COMPLETION | 开启交互模式并先等待用户输入。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [57](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1932) | `-mli`<br>`--multiline-input` | `无值开关` | COMPLETION, CLI | 支持多行输入或粘贴。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [58](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1939) | `--in-prefix-bos` | `无值开关` | COMPLETION | 在用户输入前添加 BOS token，再添加输入前缀。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [59](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1947) | `--in-prefix` | `STRING` | COMPLETION | 用户输入前缀。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [60](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1955) | `--in-suffix` | `STRING` | COMPLETION | 用户输入后缀。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [61](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1963) | `--warmup`<br>`--no-warmup` | `无值开关` | COMPLETION, CLI, SERVER, MTMD, EMBEDDING, RETRIEVAL, PERPLEXITY, DEBUG | 开关正式运行前的空跑预热。 | 预热增加启动时间，但可将首次图构建/分配等成本移出首个请求。 |
| [62](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1971) | `--spm-infill` | `无值开关` | SERVER | 使用 Suffix/Prefix/Middle 代码补全排列。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |

## 采样与结构化输出

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [63](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1981) | `--samplers` | `SAMPLERS` | COMMON | 按分号分隔指定完整采样器执行顺序。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [64](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1990) | `-s`<br>`--seed` | `SEED` | COMMON | 采样随机种子。 | 影响随机结果和复现实验；通常不直接提升吞吐。 |
| [65](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1997) | `--sampler-seq`<br>`--sampling-seq` | `SEQUENCE` | COMMON | 使用简写指定采样器序列。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [66](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2004) | `--ignore-eos` | `无值开关` | COMMON | 忽略 EOS，允许继续生成。 | 忽略终止符可显著增加生成 token 总量与等待时间，需同时设置生成上限。 |
| [67](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2011) | `--temp`<br>`--temperature` | `N` | COMMON | 采样温度。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [68](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2020) | `--top-k` | `N` | COMMON | 仅保留概率最高的 K 个候选；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [69](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2028) | `--top-p` | `N` | COMMON | nucleus/top-p 累积概率截断；1.0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [70](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2036) | `--min-p` | `N` | COMMON | 以最高概率为基准的 min-p 相对阈值；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [71](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2044) | `--top-nsigma`<br>`--top-n-sigma` | `N` | COMMON | top-n-sigma 采样阈值；-1 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [72](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2051) | `--xtc-probability` | `N` | COMMON | XTC 采样触发概率；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [73](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2059) | `--xtc-threshold` | `N` | COMMON | XTC 候选概率阈值；1 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [74](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2067) | `--typical`<br>`--typical-p` | `N` | COMMON | 局部典型性采样概率 p；1 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [75](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2074) | `--repeat-last-n` | `N` | COMMON | 重复惩罚检查的最近 token 数；0 禁用。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [76](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2086) | `--repeat-penalty` | `N` | COMMON | 重复 token 的乘性惩罚；1 不惩罚。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [77](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2100) | `--presence-penalty` | `N` | COMMON | token 出现即施加的 presence 惩罚；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [78](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2111) | `--frequency-penalty` | `N` | COMMON | 随 token 出现次数累加的 frequency 惩罚；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [79](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2122) | `--dry-multiplier` | `N` | COMMON | DRY 长序列重复惩罚乘数；0 禁用。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [80](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2129) | `--dry-base` | `N` | COMMON | DRY 惩罚指数底数。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [81](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2140) | `--dry-allowed-length` | `N` | COMMON | DRY 允许的无惩罚重复长度。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [82](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2147) | `--dry-penalty-last-n` | `N` | COMMON | DRY 检查的最近 token 范围；0 禁用。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [83](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2157) | `--dry-sequence-breaker` | `STRING` | COMMON | 添加 DRY 序列分隔符并清空默认集合；none 表示不使用分隔符。 | 改变重复检测和惩罚成本及输出质量；更长检查范围可能增加采样负担。 |
| [84](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2183) | `--adaptive-target` | `N` | COMMON | adaptive-p 目标概率；负数禁用，启用时取 0 至 1。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [85](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2193) | `--adaptive-decay` | `N` | COMMON | adaptive-p 目标自适应衰减率，0 至 0.99；小值响应快，大值更稳定。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [86](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2203) | `--dynatemp-range` | `N` | COMMON | 动态温度变化范围；0 禁用。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [87](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2210) | `--dynatemp-exp` | `N` | COMMON | 动态温度指数。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [88](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2217) | `--mirostat` | `N` | COMMON | Mirostat：0 禁用、1 原版、2 第二版；启用时忽略 top-k、top-p 和 typical。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [89](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2226) | `--mirostat-lr` | `N` | COMMON | Mirostat 学习率 eta。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [90](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2234) | `--mirostat-ent` | `N` | COMMON | Mirostat 目标熵 tau。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [91](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2242) | `-l`<br>`--logit-bias` | `TOKEN_ID(+/-)BIAS` | COMMON | 按 token ID 增减 logit；格式如 15043+1 或 15043-1。 | 主要影响输出分布、重复率与质量；采样/约束本身有 CPU 成本，并可间接改变输出长度。 |
| [92](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2264) | `--grammar` | `GRAMMAR` | COMMON | 用类似 BNF 的语法约束生成。 | 筛选合法 token 可保证格式，但复杂语法会增加逐 token 约束计算成本。 |
| [93](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2271) | `--grammar-file` | `FNAME` | COMMON | 从文件加载生成语法。 | 筛选合法 token 可保证格式，但复杂语法会增加逐 token 约束计算成本。 |
| [94](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2278) | `-j`<br>`--json-schema` | `SCHEMA` | COMMON | 用 JSON Schema 约束生成；外部引用需先转换为语法。 | 筛选合法 token 可保证格式，但复杂语法会增加逐 token 约束计算成本。 |
| [95](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2285) | `-jf`<br>`--json-schema-file` | `FILE` | COMMON | 从文件加载 JSON Schema 约束。 | 筛选合法 token 可保证格式，但复杂语法会增加逐 token 约束计算成本。 |
| [96](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2302) | `-bs`<br>`--backend-sampling` | `无值开关` | COMMON | 实验性后端采样，将采样计算放到后端执行。 | 可减少 logits 传回 CPU 的同步/传输；实验性，收益与支持依赖后端及采样配置。 |

## Embedding、位置编码与 KV

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [97](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2309) | `--pooling` | `{none,mean,cls,last,rank}` | EMBEDDING, RETRIEVAL, SERVER, DEBUG | embedding 池化方式：none/mean/cls/last/rank；缺省沿用模型。 | 改变 embedding 计算/聚合与语义；注意力类型也影响计算布局，不能仅按速度选择。 |
| [98](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2321) | `--attention` | `{causal,non-causal}` | EMBEDDING | embedding 注意力类型：causal/non-causal；缺省沿用模型。 | 改变 embedding 计算/聚合与语义；注意力类型也影响计算布局，不能仅按速度选择。 |
| [99](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2330) | `--rope-scaling` | `{none,linear,yarn}` | COMMON | RoPE 缩放方法：none/linear/yarn；优先遵循模型配置。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [100](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2340) | `--rope-scale` | `N` | COMMON | 上下文 RoPE 扩展倍数 N。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [101](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2347) | `--rope-freq-base` | `N` | COMMON | RoPE 基础频率；缺省从模型读取。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [102](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2354) | `--rope-freq-scale` | `N` | COMMON | RoPE 频率缩放因子；上下文扩展倍数为 1/N。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [103](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2361) | `--yarn-orig-ctx` | `N` | COMMON | YaRN 原始训练上下文长度。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [104](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2368) | `--yarn-ext-factor` | `N` | COMMON | YaRN 外推混合因子；0 为完全插值。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [105](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2375) | `--yarn-attn-factor` | `N` | COMMON | YaRN 注意力幅值缩放。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [106](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2382) | `--yarn-beta-slow` | `N` | COMMON | YaRN 慢变化频率校正参数。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [107](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2389) | `--yarn-beta-fast` | `N` | COMMON | YaRN 快变化频率校正参数。 | 主要改变长上下文位置编码和质量；不是免费扩容，实际长上下文仍需 KV 与注意力计算。 |
| [108](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2396) | `-gan`<br>`--grp-attn-n` | `N` | COMPLETION, PASSKEY | 分组注意力的组因子。 | 改变长上下文分组注意力行为；影响上下文表达与相关计算。 |
| [109](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2403) | `-gaw`<br>`--grp-attn-w` | `N` | COMPLETION | 分组注意力窗口宽度。 | 改变长上下文分组注意力行为；影响上下文表达与相关计算。 |
| [110](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2410) | `-kvo`<br>`--kv-offload`<br>`-nkvo`<br>`--no-kv-offload` | `无值开关` | COMMON | 开关 KV 缓存卸载到设备。 | KV 放设备可减少 CPU 路径，消耗设备内存；禁用可能减显存但增加传输/主机工作。 |
| [111](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2418) | `--repack`<br>`-nr`<br>`--no-repack` | `无值开关` | COMMON | 开关权重重排。 | 加载时重排耗时/占内存，可能换取 CPU 推理更高效的数据布局。 |
| [112](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2426) | `--no-host` | `无值开关` | COMMON | 绕过 host buffer，允许使用额外缓冲区。 | 改变张量缓冲区或算子位置，影响内存容量、计算后端和跨设备传输。 |
| [113](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2433) | `-ctk`<br>`--cache-type-k` | `TYPE` | COMMON | K 缓存元素的数据类型。 | 较低位缓存减少 KV 容量/带宽，但引入量化误差和转换成本；支持及加速因后端而异。 |
| [114](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2446) | `-ctv`<br>`--cache-type-v` | `TYPE` | COMMON | V 缓存元素的数据类型。 | 较低位缓存减少 KV 容量/带宽，但引入量化误差和转换成本；支持及加速因后端而异。 |

## 质量评测与并发

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [115](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2459) | `--hellaswag` | `无值开关` | PERPLEXITY | 使用 -f 数据集运行 HellaSwag 评测。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [116](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2466) | `--hellaswag-tasks` | `N` | PERPLEXITY | HellaSwag 随机任务数量。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [117](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2473) | `--winogrande` | `无值开关` | PERPLEXITY | 使用 -f 数据集运行 Winogrande 评测。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [118](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2480) | `--winogrande-tasks` | `N` | PERPLEXITY | Winogrande 随机任务数量。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [119](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2487) | `--multiple-choice` | `无值开关` | PERPLEXITY | 使用 -f 数据集运行多选题评测。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [120](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2494) | `--multiple-choice-tasks` | `N` | PERPLEXITY | 多选题评测任务数量。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [121](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2501) | `--kl-divergence` | `无值开关` | PERPLEXITY | 与指定基础 logits 计算 KL 散度。 | 额外 logits/统计/质量评测计算增加运行时间或存储，不是普通生成加速参数。 |
| [122](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2508) | `--save-all-logits`<br>`--kl-divergence-base` | `FNAME` | PERPLEXITY | 保存所有 logits 或指定 KL 比较的 logits 文件。 | 额外 logits/统计/质量评测计算增加运行时间或存储，不是普通生成加速参数。 |
| [123](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2515) | `--ppl-stride` | `N` | PERPLEXITY | perplexity 计算滑动步幅。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [124](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2522) | `--ppl-output-type` | `<0\|1>` | PERPLEXITY | perplexity 结果输出类型。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [125](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2529) | `-dt`<br>`--defrag-thold` | `N` | COMMON | 已弃用的 KV 碎片整理阈值；当前处理器仅发出警告，不改变参数。 | 无性能作用；仅保留兼容解析并警告。 |
| [126](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2540) | `-np`<br>`--parallel` | `N` | SERVER | server 槽位数；-1 自动，0 非法。 | 增加可同时服务序列及批利用率可提高总吞吐，但可能增加单请求延迟和 KV 需求。 |
| [127](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2551) | `-np`<br>`--parallel` | `N` | COMMON；仅非 SERVER 分支 | 非 server 工具的并行解码序列数；与 server 同名但另行注册。 | 增加可同时服务序列及批利用率可提高总吞吐，但可能增加单请求延迟和 KV 需求。 |
| [128](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2559) | `-ns`<br>`--sequences` | `N` | PARALLEL | 总解码序列数。 | 增加可同时服务序列及批利用率可提高总吞吐，但可能增加单请求延迟和 KV 需求。 |
| [129](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2566) | `-cb`<br>`--cont-batching`<br>`-nocb`<br>`--no-cont-batching` | `无值开关` | SERVER | 开关连续/动态批处理。 | 增加可同时服务序列及批利用率可提高总吞吐，但可能增加单请求延迟和 KV 需求。 |

## 多模态输入与编码

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [130](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2574) | `-mm`<br>`--mmproj` | `FILE` | MTMD, SERVER, CLI, TTS | 多模态投影器模型文件；使用 -hf 时可自动选择。 | 决定是否加载多模态模块，改变模型驻留内存与媒体编码工作量。 |
| [131](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2582) | `-mmu`<br>`--mmproj-url` | `URL` | MTMD, SERVER, CLI, TTS | 多模态投影器下载 URL。 | 决定是否加载多模态模块，改变模型驻留内存与媒体编码工作量。 |
| [132](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2589) | `--mmproj-auto`<br>`--no-mmproj`<br>`--no-mmproj-auto` | `无值开关` | MTMD, SERVER, CLI, DOWNLOAD | 开关自动使用可用的多模态投影器。 | 决定是否加载多模态模块，改变模型驻留内存与媒体编码工作量。 |
| [133](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2597) | `--mmproj-offload`<br>`--no-mmproj-offload` | `无值开关` | MTMD, SERVER, CLI, TTS | 开关多模态投影器 GPU 卸载。 | 改变视觉/音频编码所在设备，影响编码速度、显存及跨设备传输。 |
| [134](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2605) | `-mmdev`<br>`--mmproj-device` | `DEVICE` | MTMD, SERVER, CLI, TTS | 指定多模态投影器设备；只能指定一个设备，none 禁止卸载。 | 改变视觉/音频编码所在设备，影响编码速度、显存及跨设备传输。 |
| [135](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2625) | `--image`<br>`--audio`<br>`--video` | `FILE` | MTMD, CLI | 图片、音频或视频输入文件；可用逗号分隔多个文件。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [136](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2634) | `--image-min-tokens` | `N` | MTMD, SERVER, CLI, TTS | 动态分辨率视觉模型每幅图像的最少 token 数。 | 图像 token 越多通常保留更多细节，也增加编码、预填充、KV 和后续注意力成本。 |
| [137](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2641) | `--image-max-tokens` | `N` | MTMD, SERVER, CLI, TTS | 动态分辨率视觉模型每幅图像的最多 token 数。 | 图像 token 越多通常保留更多细节，也增加编码、预填充、KV 和后续注意力成本。 |
| [138](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2648) | `--mtmd-batch-max-tokens` | `N` | SERVER | 图像编码单批最多图像 token 数。 | 图像编码批量增大会改变利用率与峰值显存；超大批可能内存不足。 |
| [139](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2655) | `--video-fps` | `N` | MTMD, SERVER, CLI, TTS | 视频目标采样帧率。 | 更多帧或时间戳增大媒体编码或文本 token 工作量与上下文占用。 |
| [140](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2662) | `--video-timestamp-interval` | `N` | MTMD, SERVER, CLI, TTS | 视频文本时间戳间隔，单位毫秒。 | 更多帧或时间戳增大媒体编码或文本 token 工作量与上下文占用。 |
| [141](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2669) | `--video-ffmpeg-dir` | `DIR` | MTMD, SERVER, CLI, TTS | ffmpeg 和 ffprobe 所在目录；缺省搜索 PATH。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |

## 加载、设备、卸载与内存适配

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [142](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2677) | `--rpc` | `SERVERS` | COMMON | 远程 RPC 设备服务器列表，host:port，以逗号分隔。 | 引入远端算力及网络传输；高延迟/低带宽链路可能抵消卸载收益。 |
| [143](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2686) | `--mlock` | `无值开关` | COMMON | 已弃用；锁住模型内存，替代项为 --load-mode。 | 改变加载时间、页缓存、换页和内存驻留；不能简单认定加载快就生成快。 |
| [144](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2694) | `--mmap`<br>`--no-mmap` | `无值开关` | COMMON | 已弃用；开关内存映射加载，替代项为 --load-mode。 | 改变加载时间、页缓存、换页和内存驻留；不能简单认定加载快就生成快。 |
| [145](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2703) | `-dio`<br>`--direct-io`<br>`-ndio`<br>`--no-direct-io` | `无值开关` | COMMON | 已弃用；开关 DirectIO 加载，替代项为 --load-mode。 | 改变加载时间、页缓存、换页和内存驻留；不能简单认定加载快就生成快。 |
| [146](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2712) | `-lm`<br>`--load-mode` | `MODE` | COMMON | 加载模式：auto/none/mmap/mlock/mmap+mlock/dio；默认 auto。 | 改变加载时间、页缓存、换页和内存驻留；不能简单认定加载快就生成快。 |
| [147](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2731) | `--tensor-read-lazy` | `MODE` | COMMON | 对部分张量按需读盘：on/off/auto；on 要求 mmap，auto 仅对大于 4 GiB 的适用张量开启。 | 减少某些大张量常驻内存，代价是按需磁盘 I/O；磁盘随机访问可能增大延迟。 |
| [148](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2744) | `--numa` | `TYPE` | COMMON | NUMA 策略：distribute 分散、isolate 局部节点、numactl 使用外部 CPU 映射。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [149](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2759) | `-dev`<br>`--device` | `<dev1,dev2,..>` | COMMON | GPU/加速器卸载设备列表；none 不卸载。 | 设备卸载改变计算/带宽来源；更多 GPU 层通常有利但受显存、后端与传输约束。 |
| [150](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2767) | `--list-devices` | `无值开关` | COMMON | 列出当前构建可见的设备后退出。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [151](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2775) | `-ot`<br>`--override-tensor` | `<tensor name pattern>=<buffer type>,...` | COMMON | 按张量名称模式覆盖缓冲区类型。 | 改变张量缓冲区或算子位置，影响内存容量、计算后端和跨设备传输。 |
| [152](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2781) | `-cmoe`<br>`--cpu-moe` | `无值开关` | COMMON | 所有 MoE 专家权重放在 CPU。 | 减少显存需求，但增加 CPU 计算/权重带宽及设备间成本；适合容量不足时折衷。 |
| [153](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2788) | `-ncmoe`<br>`--n-cpu-moe` | `N` | COMMON | 前 N 层的 MoE 专家权重放在 CPU。 | 减少显存需求，但增加 CPU 计算/权重带宽及设备间成本；适合容量不足时折衷。 |
| [154](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2798) | `-ncffn`<br>`--n-cpu-ffn` | `N` | COMMON | 前 N 层稠密 FFN 权重放在 CPU；MoE 请用对应专家选项。 | 减少显存需求，但增加 CPU 计算/权重带宽及设备间成本；适合容量不足时折衷。 |
| [155](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2810) | `-ngl`<br>`--gpu-layers`<br>`--n-gpu-layers` | `N` | COMMON | 存放于显存的最大层数；支持整数、auto、all。 | 设备卸载改变计算/带宽来源；更多 GPU 层通常有利但受显存、后端与传输约束。 |
| [156](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2828) | `-sm`<br>`--split-mode` | `{none,layer,row,tensor}` | COMMON | 多 GPU 拆分：none 单卡，layer 按层及 KV，row 按权重行，tensor 按张量并行拆分权重及 KV（实验性）。 | 平衡显存与设备负载；跨卡同步/互联带宽可能限制多卡加速，需对比实际负载。 |
| [157](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2852) | `-ts`<br>`--tensor-split` | `N0,N1,N2,...` | COMMON | 各 GPU 的模型分配比例，如 3,1。 | 平衡显存与设备负载；跨卡同步/互联带宽可能限制多卡加速，需对比实际负载。 |
| [158](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2879) | `-mg`<br>`--main-gpu` | `INDEX` | COMMON | 主 GPU；none 模式用于整模型，row 模式用于中间结果和 KV。 | 平衡显存与设备负载；跨卡同步/互联带宽可能限制多卡加速，需对比实际负载。 |
| [159](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2889) | `-fit`<br>`--fit` | `[on\|off]` | COMMON | 开关根据设备内存调整尚未显式指定的参数。 | 间接改变未显式设置的上下文或卸载配置；先核对启动日志，避免比较时配置悄然变化。 |
| [160](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2903) | `-fitp`<br>`--fit-print` | `[on\|off]` | FIT_PARAMS | 开关打印预计需要的内存。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [161](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2917) | `-fitt`<br>`--fit-target` | `MiB0,MiB1,MiB2,...` | COMMON | 自动适配时各设备保留的内存余量 MiB；单值广播到所有设备。 | 间接改变未显式设置的上下文或卸载配置；先核对启动日志，避免比较时配置悄然变化。 |
| [162](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2942) | `-fitc`<br>`--fit-ctx` | `N` | COMMON | 自动适配可设置的最小上下文容量。 | 间接改变未显式设置的上下文或卸载配置；先核对启动日志，避免比较时配置悄然变化。 |
| [163](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2949) | `--check-tensors` | `无值开关` | COMMON | 加载时检查张量数据是否有非法值。 | 增加加载检查成本，不是稳态推理加速项。 |
| [164](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2956) | `--override-kv` | `KEY=TYPE:VALUE,...` | COMMON | 覆盖模型元数据；支持 int/float/bool/str，可逗号分隔多个覆盖项。 | 元数据可改变 tokenizer、架构配置或上下文解释；错误覆盖可能损害正确性。 |
| [165](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2968) | `--op-offload`<br>`--no-op-offload` | `无值开关` | COMMON | 开关将 host 张量操作卸载到设备。 | 改变张量缓冲区或算子位置，影响内存容量、计算后端和跨设备传输。 |

## 适配器、模型选择与下载

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [166](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3046) | `--lora` | `FNAME` | COMMON, EXPORT_LORA | 加载 LoRA 适配器文件，可逗号分隔多个。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [167](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2986) | `--lora-scaled` | `FNAME:SCALE,...` | COMMON, EXPORT_LORA | 加载带自定义权重系数的 LoRA 适配器。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [168](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3001) | `--control-vector` | `FNAME` | COMMON | 添加控制向量文件，可逗号分隔多个。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [169](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3010) | `--control-vector-scaled` | `FNAME:SCALE,...` | COMMON | 添加带自定义缩放系数的控制向量。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [170](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3024) | `--control-vector-layer-range` | `START END` | COMMON | 控制向量生效层范围，起止均包含。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [171](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3032) | `-a`<br>`--alias` | `STRING` | SERVER | API 模型名称别名，可逗号分隔多个。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [172](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3044) | `--tags` | `STRING` | SERVER | 模型信息标签；不参与路由。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [173](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3056) | `-m`<br>`--model` | `FNAME` | COMMON, EXPORT_LORA, DOWNLOAD, TOKENIZE | 主模型文件路径；export-lora 中指基础模型。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [174](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3065) | `-mu`<br>`--model-url` | `MODEL_URL` | COMMON, DOWNLOAD, TOKENIZE | 模型下载 URL。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [175](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3072) | `-dr`<br>`--docker-repo` | `[<repo>/]<model>[:quant]` | COMMON, DOWNLOAD, TOKENIZE | Docker Hub 模型仓库；repo 缺省 ai/，quant 缺省 latest。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [176](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3081) | `-hf`<br>`-hfr`<br>`--hf-repo` | `<user>/<model>[:quant]` | COMMON, DOWNLOAD, TOKENIZE | Hugging Face 仓库及可选量化；缺省优先 Q4_K_M，不存在时使用仓库首个文件；可自动下载 mmproj。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [177](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3091) | `-hff`<br>`--hf-file` | `FILE` | COMMON, DOWNLOAD, TOKENIZE | Hugging Face 文件名；覆盖 -hf 的量化选择。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [178](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3098) | `-hft`<br>`--hf-token` | `TOKEN` | COMMON, DOWNLOAD, TOKENIZE | Hugging Face 访问令牌；缺省可从 HF_TOKEN 读取。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [179](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3105) | `--mtp` | `无值开关` | DOWNLOAD | 下载工具同时下载可用的 MTP 预测头。 | 这里只控制下载附件；不等于已经在服务中启用相应推测算法。 |
| [180](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3112) | `--dflash` | `无值开关` | DOWNLOAD | 下载工具同时下载可用的 DFlash 辅助模型。 | 这里只控制下载附件；不等于已经在服务中启用相应推测算法。 |
| [181](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3119) | `--eagle3` | `无值开关` | DOWNLOAD | 下载工具同时下载可用的 Eagle3 辅助模型。 | 这里只控制下载附件；不等于已经在服务中启用相应推测算法。 |

## 检索、校准、分词及基准测试

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [182](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3126) | `--context-file` | `FNAME` | RETRIEVAL | 检索工具的上下文文件，可逗号分隔多个。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [183](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3139) | `--chunk-size` | `N` | RETRIEVAL | 检索 embedding 文本块的最小长度。 | 文本块尺寸改变 embedding 批次、文本长度与检索粒度。 |
| [184](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3146) | `--chunk-separator` | `STRING` | RETRIEVAL | 文本块分隔符。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [185](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3153) | `--junk` | `N` | PASSKEY, PARALLEL | passkey 测试中无关文本重复次数。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [186](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3160) | `--pos` | `N` | PASSKEY | passkey 在无关文本中的位置。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [187](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3167) | `-o`<br>`--output`<br>`--output-file` | `FNAME` | IMATRIX, CVECTOR_GENERATOR, EXPORT_LORA, TTS, FINETUNE, RESULTS, EXPORT_GRAPH_OPS, CLI | 工具输出文件路径。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [188](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3175) | `-ofreq`<br>`--output-frequency` | `N` | IMATRIX | 每 N 次迭代输出一次重要性矩阵。 | 更频繁保存增加磁盘 I/O；影响校准流程耗时，不直接改变在线生成速度。 |
| [189](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3182) | `--output-format` | `{gguf,dat}` | IMATRIX | 重要性矩阵文件格式：gguf/dat。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [190](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3191) | `--save-frequency` | `N` | IMATRIX | 每 N 次迭代另存一份重要性矩阵。 | 更频繁保存增加磁盘 I/O；影响校准流程耗时，不直接改变在线生成速度。 |
| [191](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3198) | `--process-output` | `无值开关` | IMATRIX | 重要性矩阵统计包含输出张量。 | 额外 logits/统计/质量评测计算增加运行时间或存储，不是普通生成加速参数。 |
| [192](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3205) | `--ppl`<br>`--no-ppl` | `无值开关` | IMATRIX | 开关 perplexity 计算。 | 额外 logits/统计/质量评测计算增加运行时间或存储，不是普通生成加速参数。 |
| [193](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3213) | `--chunk`<br>`--from-chunk` | `N` | IMATRIX | 从第 N 个数据块开始处理。 | 改变评测工作量或输入布局，影响评测耗时及可比较性。 |
| [194](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3220) | `--show-statistics` | `无值开关` | IMATRIX | 显示重要性矩阵统计信息后退出。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [195](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3227) | `--parse-special` | `无值开关` | IMATRIX | 解析聊天、工具等特殊 token。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [196](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3234) | `--ids` | `无值开关` | TOKENIZE | 只打印可被 Python 解析的 token ID 列表。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [197](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3241) | `--stdin` | `无值开关` | TOKENIZE | 从标准输入读取提示词，优先于 -f 和 -p。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [198](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3248) | `--no-bos` | `无值开关` | TOKENIZE | 即便模型通常需要，也不自动添加 BOS。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [199](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3255) | `--no-parse-special` | `无值开关` | TOKENIZE | 不解析特殊 token。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [200](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3262) | `--show-count` | `无值开关` | TOKENIZE | 打印 token 总数。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [201](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3269) | `-pps` | `无值开关` | BENCH, PARALLEL | 基准测试中各并行序列共享提示词。 | 改变基准测试工作负载与共享方式；必须固定这些条件才能比较 PP/TG/并发性能。 |
| [202](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3276) | `-tgs` | `无值开关` | BENCH, PARALLEL | 基准测试中各序列分开执行文本生成。 | 改变基准测试工作负载与共享方式；必须固定这些条件才能比较 PP/TG/并发性能。 |
| [203](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3283) | `-npp` | `n0,n1,...` | BENCH | 基准测试提示词 token 长度列表。 | 改变基准测试工作负载与共享方式；必须固定这些条件才能比较 PP/TG/并发性能。 |
| [204](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3291) | `-ntg` | `n0,n1,...` | BENCH | 基准测试生成 token 长度列表。 | 改变基准测试工作负载与共享方式；必须固定这些条件才能比较 PP/TG/并发性能。 |
| [205](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3299) | `-npl` | `n0,n1,...` | BENCH | 基准测试并行提示词数量列表。 | 改变基准测试工作负载与共享方式；必须固定这些条件才能比较 PP/TG/并发性能。 |

## Embedding 输出与 HTTP 服务

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [206](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3307) | `--embd-normalize` | `N` | EMBEDDING, SERVER, DEBUG | embedding 归一化：-1 不归一化，0 最大绝对值 int16，1 L1，2 L2，>2 p 范数。 | 增加 embedding 后处理；json+ 的成对相似度矩阵有额外计算及输出成本。 |
| [207](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3314) | `--embd-output-format` | `FORMAT` | EMBEDDING | embedding 输出格式；json+ 额外生成余弦相似度矩阵。 | 增加 embedding 后处理；json+ 的成对相似度矩阵有额外计算及输出成本。 |
| [208](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3321) | `--embd-separator` | `STRING` | EMBEDDING | embedding 输入分隔符。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [209](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3328) | `--cls-separator` | `STRING` | EMBEDDING | 分类序列分隔符。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [210](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3335) | `--host` | `HOST` | SERVER | 服务监听地址；以 .sock 结尾可使用 UNIX socket。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [211](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3342) | `--port` | `PORT` | SERVER | 服务监听端口。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [212](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3349) | `--reuse-port` | `无值开关` | SERVER | 允许多个 socket 绑定同一端口。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [213](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3356) | `--path` | `PATH` | SERVER | 静态网页文件目录。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [214](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3363) | `--cors-origins` | `ORIGINS` | SERVER | CORS 允许来源列表；localhost 为特殊本地来源规则。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [215](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3374) | `--cors-methods` | `METHODS` | SERVER | CORS 允许 HTTP 方法列表。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [216](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3381) | `--cors-headers` | `HEADERS` | SERVER | CORS 允许请求头列表。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [217](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3388) | `--cors-credentials`<br>`--no-cors-credentials` | `无值开关` | SERVER | CORS 是否允许凭证；配合通配来源时会回显请求 Origin。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [218](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3399) | `--api-prefix` | `PREFIX` | SERVER | API URL 前缀，不带末尾斜杠。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [219](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3406) | `--ui-config`<br>`--webui-config` | `JSON` | SERVER | 内联 JSON 设置 Web UI 默认配置。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [220](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3413) | `--ui-config-file`<br>`--webui-config-file` | `PATH` | SERVER | 从 JSON 文件设置 Web UI 默认配置。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [221](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3420) | `--ui-mcp-proxy`<br>`--webui-mcp-proxy`<br>`--no-ui-mcp-proxy`<br>`--no-webui-mcp-proxy` | `无值开关` | SERVER | 开关实验性 MCP CORS 代理。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [222](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3428) | `--tools` | `TOOL1,TOOL2,...` | SERVER | 开启指定内建工具或 all；可涉及文件读写与命令执行。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [223](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3438) | `--tools-runtime` | `OPTION` | SERVER | 工具执行环境：宿主机、Docker/Podman 新建或既有容器、SSH 远端。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [224](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3449) | `--mcp-servers-config` | `PATH` | SERVER | 从文件加载 MCP 服务定义。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [225](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3457) | `--mcp-servers-json` | `JSON` | SERVER | 内联 JSON 定义 MCP 服务。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [226](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3465) | `-ag`<br>`--agent`<br>`-no-ag`<br>`--no-agent` | `无值开关` | SERVER | 一并开启或关闭 CORS 代理及全部内建工具的 agent 模式。 | 影响服务/工具执行或多进程资源使用；完整 agent 请求还包含工具延迟。 |
| [227](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3481) | `--ui`<br>`--webui`<br>`--no-ui`<br>`--no-webui` | `无值开关` | SERVER | 开关 Web UI。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [228](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3489) | `--embedding`<br>`--embeddings` | `无值开关` | SERVER, DEBUG | 服务器限制为 embedding 用途；需专用 embedding 模型。 | 切换任务类型与接口；embedding/rerank 的吞吐指标不能直接等同生成 token/s。 |
| [229](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3496) | `--rerank`<br>`--reranking` | `无值开关` | SERVER | 开启 reranking 接口。 | 切换任务类型与接口；embedding/rerank 的吞吐指标不能直接等同生成 token/s。 |
| [230](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3504) | `--api-key` | `KEY` | SERVER | API 身份认证密钥，可逗号分隔多个。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [231](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3515) | `--api-key-file` | `FNAME` | SERVER | 从文件读取 API 密钥，每行一个，# 行视为注释。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [232](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3532) | `--ssl-key-file` | `FNAME` | SERVER | TLS/SSL PEM 私钥文件。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [233](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3539) | `--ssl-cert-file` | `FNAME` | SERVER | TLS/SSL PEM 证书文件。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [234](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3546) | `--chat-template-kwargs` | `STRING` | SERVER, CLI | 以 JSON 对象传递聊天模板额外参数。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [235](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3560) | `-to`<br>`--timeout` | `N` | SERVER | 服务读写超时时间，单位秒。 | 影响连接行为与端到端体验，通常不改变模型算子速度。 |
| [236](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3568) | `--sse-ping-interval` | `N` | SERVER | SSE 保活 ping 间隔，单位秒；-1 禁用。 | 影响连接行为与端到端体验，通常不改变模型算子速度。 |
| [237](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3575) | `--threads-http` | `N` | SERVER | HTTP 请求处理线程数。 | 影响 HTTP 接入和请求处理吞吐；不等于模型计算线程数。 |

## 服务缓存、路由、模板与思考

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [238](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3582) | `--cache-prompt`<br>`--no-cache-prompt` | `无值开关` | SERVER | 开关服务端提示词缓存。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [239](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3590) | `--cache-reuse` | `N` | SERVER | 通过 KV 移位尝试复用缓存的最小文本块；依赖提示词缓存。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [240](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3600) | `--metrics` | `无值开关` | SERVER | 开启 Prometheus 指标接口。 | 监测接口/采集可能有少量开销，主要用于观测服务负载。 |
| [241](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3607) | `--props` | `无值开关` | SERVER | 允许 POST /props 修改全局属性。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [242](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3614) | `--slots`<br>`--no-slots` | `无值开关` | SERVER | 开关槽位监控接口。 | 监测接口/采集可能有少量开销，主要用于观测服务负载。 |
| [243](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3622) | `--slot-save-path` | `PATH` | SERVER | 槽位 KV 缓存保存目录。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [244](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3636) | `--media-path` | `PATH` | SERVER | 服务可读取的本地媒体目录，供相对 file:// URL 使用。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [245](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3650) | `--models-dir` | `PATH` | SERVER | 路由服务器扫描的模型目录。 | 影响模型加载/卸载、冷启动及多模型争用；同时驻留更多模型消耗更多内存。 |
| [246](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3657) | `--models-preset` | `PATH` | SERVER | 路由服务器模型预设 INI 文件。 | 影响模型加载/卸载、冷启动及多模型争用；同时驻留更多模型消耗更多内存。 |
| [247](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3664) | `--models-max` | `N` | SERVER | 路由服务器最多同时加载模型数；0 不限。 | 影响模型加载/卸载、冷启动及多模型争用；同时驻留更多模型消耗更多内存。 |
| [248](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3671) | `--models-autoload`<br>`--no-models-autoload` | `无值开关` | SERVER | 开关路由服务器自动加载模型。 | 影响模型加载/卸载、冷启动及多模型争用；同时驻留更多模型消耗更多内存。 |
| [249](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3679) | `--jinja`<br>`--no-jinja` | `无值开关` | SERVER, COMPLETION, CLI, MTMD | 开关 Jinja 聊天模板引擎。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [250](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3687) | `--reasoning-format` | `FORMAT` | SERVER, COMPLETION, CLI | 推理内容解析格式：none/deepseek/deepseek-legacy；默认自动选择。 | 主要改变响应解析/字段，通常不减少模型已生成的思考 token。 |
| [251](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3698) | `-rea`<br>`--reasoning` | `[on\|off\|auto]` | SERVER, COMPLETION, CLI | 模板支持时开启/关闭思考：on/off/auto。 | 模型/模板支持时改变思考 token 总量与质量，显著影响总响应时长；不是每 token 必然更快。 |
| [252](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3716) | `--reasoning-effort` | `LEVEL` | SERVER, COMPLETION, CLI | 传给聊天模板的推理强度；default 保留模板默认，其余等级由模板支持决定。 | 模型/模板支持时改变思考 token 总量与质量，显著影响总响应时长；不是每 token 必然更快。 |
| [253](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3728) | `--reasoning-budget` | `N` | SERVER, COMPLETION, CLI | 思考 token 预算：-1 不限，0 立即结束思考，正数为预算。 | 模型/模板支持时改变思考 token 总量与质量，显著影响总响应时长；不是每 token 必然更快。 |
| [254](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3736) | `--reasoning-budget-message` | `MESSAGE` | SERVER, COMPLETION, CLI | 思考预算耗尽时，在思考结束标签前注入的文本。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [255](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3743) | `--reasoning-preserve`<br>`--no-reasoning-preserve` | `无值开关` | SERVER, COMPLETION, CLI | 保留完整历史中的思考轨迹；要求模板 supports_preserve_reasoning 能力。 | 保留更多思考历史增大下轮输入及 KV 使用，影响预填充与长对话成本。 |
| [256](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3757) | `--chat-template` | `JINJA_TEMPLATE` | COMPLETION, CLI, SERVER, MTMD | 自定义聊天模板；缺省取模型元数据。前后缀可能禁用模板，非内建模板需先启用 Jinja。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [257](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3769) | `--chat-template-file` | `JINJA_TEMPLATE_FILE` | COMPLETION, CLI, SERVER | 从文件读取聊天模板。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [258](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3781) | `--skip-chat-parsing`<br>`--no-skip-chat-parsing` | `无值开关` | COMPLETION, CLI, SERVER | 开关跳过聊天解析，直接将思考及工具调用等作为 content 输出。 | 主要改变响应解析/字段，通常不减少模型已生成的思考 token。 |
| [259](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3792) | `--prefill-assistant`<br>`--no-prefill-assistant` | `无值开关` | SERVER | 开关将最后一条 assistant 消息作为未完成内容继续预填充。 | 通过实际输入 token、模板或媒体内容改变预填充工作量及剩余上下文。 |
| [260](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3803) | `-sps`<br>`--slot-prompt-similarity` | `SIMILARITY` | SERVER | 请求提示词与槽位提示词的匹配阈值；0 禁用相似度门槛。 | 提示词/状态匹配命中可跳过重复预填充；读写、拷贝或不命中会增加成本。 |
| [261](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3810) | `--lora-init-without-apply` | `无值开关` | SERVER | 加载但暂不应用 LoRA，之后可通过接口应用。 | 改变模型行为，并可能增加适配器/控制向量驻留与额外计算；需实测。 |
| [262](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3817) | `--sleep-idle-seconds` | `SECONDS` | SERVER | 空闲指定秒数后休眠；-1 禁用。 | 空闲时释放/降低资源占用，但唤醒请求可能有重新加载的冷启动延迟。 |
| [263](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3827) | `--simple-io` | `无值开关` | COMPLETION, CLI | 使用兼容子进程或受限终端的基础输入输出。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |

## 控制向量与日志

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [264](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3834) | `--positive-file` | `FNAME` | CVECTOR_GENERATOR | 控制向量生成用正例提示词文件，每行一条。 | 影响控制向量离线生成的工作量、内存与结果；不直接调节普通在线推理。 |
| [265](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3841) | `--negative-file` | `FNAME` | CVECTOR_GENERATOR | 控制向量生成用反例提示词文件，每行一条。 | 影响控制向量离线生成的工作量、内存与结果；不直接调节普通在线推理。 |
| [266](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3848) | `--pca-batch` | `N` | CVECTOR_GENERATOR | PCA 批大小；增大可加速但使用更多内存。 | 影响控制向量离线生成的工作量、内存与结果；不直接调节普通在线推理。 |
| [267](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3855) | `--pca-iter` | `N` | CVECTOR_GENERATOR | PCA 迭代次数。 | 影响控制向量离线生成的工作量、内存与结果；不直接调节普通在线推理。 |
| [268](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3862) | `--method` | `{pca, mean}` | CVECTOR_GENERATOR | 控制向量降维方法：pca/mean，默认 pca。 | 影响控制向量离线生成的工作量、内存与结果；不直接调节普通在线推理。 |
| [269](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3871) | `--output-format` | `{md,jsonl}` | BENCH | batched-bench 输出格式：md/jsonl，默认 md。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [270](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3880) | `--log-disable` | `无值开关` | COMMON | 关闭日志。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [271](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3887) | `--log-file` | `FNAME` | COMMON | 日志写入文件。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [272](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3894) | `--log-prompts-dir` | `PATH` | SERVER, CLI | 将提示词日志写入目录，供调试使用。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [273](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3906) | `--log-colors` | `[on\|off\|auto]` | COMMON | 日志颜色 on/off/auto，默认 auto。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [274](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3923) | `-v`<br>`--verbose`<br>`--log-verbose` | `无值开关` | COMMON | 开启全部级别的详细日志。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [275](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3931) | `--offline` | `无值开关` | COMMON, DOWNLOAD, TOKENIZE | 离线模式，只用本地缓存并阻止联网。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [276](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3938) | `-lv`<br>`--verbosity`<br>`--log-verbosity` | `N` | COMMON | 日志详细程度：0 通用，1 错误，2 警告，3 信息，4 跟踪，5 调试。 | 大量终端/磁盘输出可影响端到端时间；通常不改变模型算子。 |
| [277](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3953) | `--log-prefix`<br>`--no-log-prefix` | `无值开关` | COMMON | 开关日志前缀。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |
| [278](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3961) | `--log-timestamps`<br>`--no-log-timestamps` | `无值开关` | COMMON | 开关日志时间戳。 | 不直接改变模型计算；影响输入输出、管理或使用方式。 |

## 草稿模型与推测解码

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [279](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3974) | `--spec-draft-hf`<br>`-hfd`<br>`-hfrd`<br>`--hf-repo-draft` | `<user>/<model>[:quant]` | SPECULATIVE, SERVER, CLI | 草稿模型 Hugging Face 仓库，语法同主模型 -hf。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [280](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3981) | `--spec-draft-threads`<br>`-td`<br>`--threads-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿模型生成线程数；缺省沿用主模型生成线程。 | 影响 CPU 算力利用率与同步成本；过多线程可能受内存带宽或调度开销限制而变慢。 |
| [281](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3991) | `--spec-draft-threads-batch`<br>`-tbd`<br>`--threads-batch-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿模型批处理线程数；缺省沿用草稿生成线程。 | 影响 CPU 算力利用率与同步成本；过多线程可能受内存带宽或调度开销限制而变慢。 |
| [282](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4001) | `--spec-draft-cpu-mask`<br>`-Cd`<br>`--cpu-mask-draft` | `M` | SPECULATIVE, SERVER, CLI | 草稿生成 CPU 亲和性掩码。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [283](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4011) | `--spec-draft-cpu-range`<br>`-Crd`<br>`--cpu-range-draft` | `lo-hi` | SPECULATIVE, SERVER, CLI | 草稿生成 CPU 亲和性编号范围。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [284](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4021) | `--spec-draft-cpu-strict`<br>`--cpu-strict-draft` | `<0\|1>` | SPECULATIVE, SERVER, CLI | 草稿生成线程严格 CPU 放置。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [285](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4028) | `--spec-draft-prio`<br>`--prio-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿生成线程优先级。 | 改变与其他进程竞争 CPU 时的调度延迟；不增加硬件算力。 |
| [286](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4038) | `--spec-draft-poll`<br>`--poll-draft` | `<0\|1>` | SPECULATIVE, SERVER, CLI | 草稿生成线程忙轮询开关。 | 忙轮询可减少唤醒延迟，但增加 CPU 占用和功耗。 |
| [287](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4045) | `--spec-draft-cpu-mask-batch`<br>`-Cbd`<br>`--cpu-mask-batch-draft` | `M` | SPECULATIVE, SERVER, CLI | 草稿批处理 CPU 亲和性掩码。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [288](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4055) | `--spec-draft-cpu-range-batch`<br>`-Crbd`<br>`--cpu-range-batch-draft` | `lo-hi` | SPECULATIVE | 草稿批处理 CPU 亲和性编号范围。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [289](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4065) | `--spec-draft-cpu-strict-batch`<br>`--cpu-strict-batch-draft` | `<0\|1>` | SPECULATIVE, SERVER, CLI | 草稿批处理线程严格 CPU 放置。 | 改变 CPU/NUMA 局部性与线程迁移；可减少跨节点访问，也可能限制可用算力。 |
| [290](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4072) | `--spec-draft-prio-batch`<br>`--prio-batch-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿批处理线程优先级。 | 改变与其他进程竞争 CPU 时的调度延迟；不增加硬件算力。 |
| [291](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4082) | `--spec-draft-poll-batch`<br>`--poll-batch-draft` | `<0\|1>` | SPECULATIVE, SERVER, CLI | 草稿批处理线程忙轮询开关。 | 忙轮询可减少唤醒延迟，但增加 CPU 占用和功耗。 |
| [292](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4089) | `--spec-draft-type-k`<br>`-ctkd`<br>`--cache-type-k-draft` | `TYPE` | COMMON | 草稿模型 K 缓存数据类型。 | 较低位缓存减少 KV 容量/带宽，但引入量化误差和转换成本；支持及加速因后端而异。 |
| [293](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4102) | `--spec-draft-type-v`<br>`-ctvd`<br>`--cache-type-v-draft` | `TYPE` | COMMON | 草稿模型 V 缓存数据类型。 | 较低位缓存减少 KV 容量/带宽，但引入量化误差和转换成本；支持及加速因后端而异。 |
| [294](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4115) | `--spec-draft-override-tensor`<br>`-otd`<br>`--override-tensor-draft` | `<tensor name pattern>=<buffer type>,...` | SPECULATIVE, SERVER, CLI | 覆盖草稿模型张量缓冲区类型。 | 改变张量缓冲区或算子位置，影响内存容量、计算后端和跨设备传输。 |
| [295](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4121) | `--spec-draft-cpu-moe`<br>`-cmoed`<br>`--cpu-moe-draft` | `无值开关` | SPECULATIVE, SERVER, CLI | 草稿模型全部 MoE 专家权重放在 CPU。 | 减少显存需求，但增加 CPU 计算/权重带宽及设备间成本；适合容量不足时折衷。 |
| [296](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4128) | `--spec-draft-n-cpu-moe`<br>`--spec-draft-ncmoe`<br>`-ncmoed`<br>`--n-cpu-moe-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿模型前 N 层 MoE 专家权重放在 CPU。 | 减少显存需求，但增加 CPU 计算/权重带宽及设备间成本；适合容量不足时折衷。 |
| [297](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4139) | `--spec-draft-n-max` | `N` | SPECULATIVE, LOOKUP, SERVER, CLI | 每轮草稿最多 token 数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [298](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4149) | `--spec-draft-n-min` | `N` | SPECULATIVE, LOOKUP, SERVER, CLI | 启用草稿验证的最少草稿 token 数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [299](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4156) | `--spec-synth-len` | `L` | SERVER, CLI | 合成平均接受长度（含目标 token），仅用于基准测试。 | 人为控制接受行为，仅用于性能实验；不可当作实际模型质量或接受率结果。 |
| [300](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4169) | `--spec-synth-rates` | `P0,P1,...` | SERVER, CLI | 各位置无条件合成接受概率，仅用于基准测试。 | 人为控制接受行为，仅用于性能实验；不可当作实际模型质量或接受率结果。 |
| [301](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4189) | `--spec-draft-p-split`<br>`--draft-p-split` | `P` | SPECULATIVE, SERVER, CLI | 推测解码草稿分支拆分概率。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [302](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4196) | `--spec-draft-p-min`<br>`--draft-p-min` | `P` | SPECULATIVE, SERVER, CLI | 贪心草稿的最小接受候选概率阈值。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [303](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4203) | `--spec-draft-backend-sampling`<br>`--no-spec-draft-backend-sampling` | `无值开关` | SPECULATIVE, SERVER, CLI | 开关将草稿采样卸载到后端。 | 可减少 logits 传回 CPU 的同步/传输；实验性，收益与支持依赖后端及采样配置。 |
| [304](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4212) | `--spec-draft-device`<br>`-devd`<br>`--device-draft` | `<dev1,dev2,..>` | SPECULATIVE, SERVER, CLI | 草稿模型卸载设备列表；none 不卸载。 | 设备卸载改变计算/带宽来源；更多 GPU 层通常有利但受显存、后端与传输约束。 |
| [305](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4221) | `--spec-draft-ngl`<br>`-ngld`<br>`--gpu-layers-draft`<br>`--n-gpu-layers-draft` | `N` | SPECULATIVE, SERVER, CLI | 草稿模型显存层数；支持整数、auto、all。 | 设备卸载改变计算/带宽来源；更多 GPU 层通常有利但受显存、后端与传输约束。 |
| [306](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4240) | `--spec-draft-model`<br>`-md`<br>`--model-draft` | `FNAME` | SPECULATIVE, SERVER, CLI | 草稿模型文件。 | 所选模型架构、参数量、量化与草稿质量决定容量/带宽/算力；下载地址本身不提速。 |
| [307](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4248) | `--spec-type` | `TYPE1,TYPE2,...（合法名称见表前说明）` | SPECULATIVE, SERVER, CLI | 推测解码算法类型列表，以逗号分隔。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [308](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4258) | `--spec-ngram-mod-n-min` | `N` | SPECULATIVE, SERVER, CLI | ngram-mod 每轮草稿最少 token 数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [309](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4268) | `--spec-ngram-mod-n-max` | `N` | SPECULATIVE, SERVER, CLI | ngram-mod 每轮草稿最多 token 数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [310](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4278) | `--spec-ngram-mod-n-match` | `N` | SPECULATIVE, SERVER, CLI | ngram-mod 查找匹配长度。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [311](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4289) | `--spec-ngram-simple-size-n` | `N` | SPECULATIVE, SERVER, CLI | ngram-simple 查询 n-gram 长度 N。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [312](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4299) | `--spec-ngram-simple-size-m` | `N` | SPECULATIVE, SERVER, CLI | ngram-simple 草稿片段长度 M。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [313](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4309) | `--spec-ngram-simple-min-hits` | `N` | SPECULATIVE, SERVER, CLI | ngram-simple 启用草稿的最少命中次数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [314](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4320) | `--spec-ngram-map-k-size-n` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k 查询 n-gram 长度 N。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [315](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4330) | `--spec-ngram-map-k-size-m` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k 草稿片段长度 M。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [316](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4340) | `--spec-ngram-map-k-min-hits` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k 启用草稿的最少命中次数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [317](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4351) | `--spec-ngram-map-k4v-size-n` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k4v 查询 n-gram 长度 N。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [318](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4361) | `--spec-ngram-map-k4v-size-m` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k4v 草稿片段长度 M。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |
| [319](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4371) | `--spec-ngram-map-k4v-min-hits` | `N` | SPECULATIVE, SERVER, CLI | ngram-map-k4v 启用草稿的最少命中次数。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |

## 已移除参数

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [320](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4386) | `--draft`<br>`--draft-n`<br>`--draft-max` | `N` | SPECULATIVE, LOOKUP, SERVER, CLI | 已移除；改用 --spec-draft-n-max 或 --spec-ngram-mod-n-max。 | 传入会报已移除错误；不可作为当前可用调优选项。 |
| [321](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4393) | `--draft-min`<br>`--draft-n-min` | `N` | SPECULATIVE, LOOKUP, SERVER, CLI | 已移除；改用 --spec-draft-n-min 或 --spec-ngram-mod-n-min。 | 传入会报已移除错误；不可作为当前可用调优选项。 |
| [322](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4400) | `--spec-ngram-size-n` | `N` | SERVER | 已移除；改用对应 --spec-ngram-*-size-n 或 --spec-ngram-mod-n-match。 | 传入会报已移除错误；不可作为当前可用调优选项。 |
| [323](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4407) | `--spec-ngram-size-m` | `N` | SERVER | 已移除；改用对应 --spec-ngram-*-size-m。 | 传入会报已移除错误；不可作为当前可用调优选项。 |
| [324](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4414) | `--spec-ngram-min-hits` | `N` | SERVER | 已移除；改用对应 --spec-ngram-*-min-hits。 | 传入会报已移除错误；不可作为当前可用调优选项。 |

## 语音、扩散、训练与调试

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [325](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4426) | `--tts-lang` | `FNAME` | TTS | 语音生成语言代码，ISO 639-1；源码占位符虽为 FNAME，实际为语言。 | 改变语音任务内容/说话人条件；任务工作量和音频质量需单独测量。 |
| [326](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4434) | `--tts-speaker-file` | `FNAME` | TTS | 语音生成的 speaker 文件。 | 改变语音任务内容/说话人条件；任务工作量和音频质量需单独测量。 |
| [327](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4446) | `--diffusion-steps` | `N` | DIFFUSION | 扩散文本模型的扩散步数。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [328](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4451) | `--diffusion-visual` | `无值开关` | DIFFUSION | 显示渐进扩散生成过程。 | 逐步显示带来额外输出成本，主要用于观察扩散过程。 |
| [329](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4456) | `--diffusion-eps` | `F` | DIFFUSION | 扩散时间步 epsilon。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [330](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4461) | `--diffusion-algorithm` | `N` | DIFFUSION | 扩散算法编号：0 原始，1 熵，2 margin，3 随机，4 置信度。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [331](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4469) | `--diffusion-alg-temp` | `F` | DIFFUSION | Dream 扩散算法温度。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [332](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4474) | `--diffusion-block-length` | `N` | DIFFUSION | LLaDA 扩散生成块长度。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [333](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4479) | `--diffusion-cfg-scale` | `F` | DIFFUSION | LLaDA classifier-free guidance 强度。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [334](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4484) | `--diffusion-add-gumbel-noise` | `F` | DIFFUSION | 当温度大于 0 时是否加 Gumbel 噪声；此版本要求 F 数值，处理器转浮点再赋值。 | 改变扩散计算步数/分块/引导与质量；扩散模型需按对应流程测量，不能套用自回归 TG。 |
| [335](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4489) | `-lr`<br>`--learning-rate` | `ALPHA` | FINETUNE | 训练 AdamW/SGD 学习率 alpha。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [336](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4494) | `-lr-min`<br>`--learning-rate-min` | `ALPHA` | FINETUNE | 学习率衰减终点。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [337](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4499) | `-decay-epochs`<br>`--learning-rate-decay-epochs` | `ALPHA` | FINETUNE | 达到学习率终点的 epoch 数；源码占位符 ALPHA，但语义是 epoch 数。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [338](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4504) | `-wd`<br>`--weight-decay` | `WD` | FINETUNE | 优化器权重衰减；0 关闭。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [339](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4509) | `-val-split`<br>`--val-split` | `FRACTION` | FINETUNE | 验证集占训练数据比例。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [340](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4514) | `-epochs`<br>`--epochs` | `N` | FINETUNE | 最大训练 epoch 数。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [341](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4519) | `-opt`<br>`--optimizer` | `sgd\|adamw` | FINETUNE | 优化器类型 sgd/adamw。 | 训练专用，影响训练收敛、时间或优化器状态内存；不直接设置部署推理速度。 |
| [342](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4528) | `--check` | `无值开关` | RESULTS | 运行核对而非生成参考结果。 | 验证/调试计算及张量输出增加额外成本，不能代表正常生产推理。 |
| [343](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4535) | `--save-logits` | `无值开关` | DEBUG | 保存最终 logits 用于验证。 | 验证/调试计算及张量输出增加额外成本，不能代表正常生产推理。 |
| [344](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4542) | `--logits-output-dir` | `PATH` | DEBUG | 验证 logits 文件输出目录。 | 验证/调试计算及张量输出增加额外成本，不能代表正常生产推理。 |
| [345](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4549) | `--tensor-filter` | `REGEX` | DEBUG | 用正则筛选调试张量名称；可重复指定。 | 验证/调试计算及张量输出增加额外成本，不能代表正常生产推理。 |

## 内建预设

| 编号/源码 | 全部拼写 | 值格式 | 适用工具 | 中文作用 | 性能影响 |
|---|---|---|---|---|---|
| [346](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4559) | `--embd-gemma-default` | `无值开关` | EMBEDDING, SERVER | 采用默认 EmbeddingGemma 模型预设，可能下载模型。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [347](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4575) | `--fim-qwen-1.5b-default` | `无值开关` | SERVER | 采用 Qwen 2.5 Coder 1.5B 补全预设，可能下载模型。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [348](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4589) | `--fim-qwen-3b-default` | `无值开关` | SERVER | 采用 Qwen 2.5 Coder 3B 补全预设，可能下载模型。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [349](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4603) | `--fim-qwen-7b-default` | `无值开关` | SERVER | 采用 Qwen 2.5 Coder 7B 补全预设，可能下载模型。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [350](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4617) | `--fim-qwen-7b-spec` | `无值开关` | SERVER | 采用 Qwen 2.5 Coder 7B + 0.5B 草稿补全预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [351](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4633) | `--fim-qwen-14b-spec` | `无值开关` | SERVER | 采用 Qwen 2.5 Coder 14B + 0.5B 草稿补全预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [352](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4649) | `--fim-qwen-30b-default` | `无值开关` | SERVER | 采用 Qwen 3 Coder 30B A3B Instruct 补全预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [353](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4663) | `--gpt-oss-20b-default` | `无值开关` | SERVER, CLI | 采用 gpt-oss-20b 模型预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [354](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4682) | `--gpt-oss-120b-default` | `无值开关` | SERVER, CLI | 采用 gpt-oss-120b 模型预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [355](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4700) | `--vision-gemma-4b-default` | `无值开关` | SERVER, CLI | 采用 Gemma 3 4B QAT 视觉模型预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [356](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4711) | `--vision-gemma-12b-default` | `无值开关` | SERVER, CLI | 采用 Gemma 3 12B QAT 视觉模型预设。 | 预设会组合选择模型及运行参数，改变容量、质量和速度；需查看源码或启动日志的展开配置。 |
| [357](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4722) | `--spec-default` | `无值开关` | SERVER, CLI | 启用默认 ngram-mod 推测配置：匹配 24，草稿最小 48、最大 64。 | 改变草稿数量、命中/接受率与验证工作；只有省下的串行目标步骤超过草稿和验证成本才加速。 |

## 完整性与适用性说明

357 行与该提交源码中的 `add_opt(common_arg(` 注册点一一对应，未通过别名去重而丢掉定义。`--parallel` 和 `--output-format` 各有不同工具语义，因此不能只按名称建字典。已移除项仍在源码注册表，但会报错；旧加载参数虽已弃用，仍有兼容处理。部分选项属于工具/模型专用，或仅控制下载，不应将“能被解析”误解为“会对当前任务生效”。

环境变量：JSON 记录 `.set_env()` 明示变量及源码对成对开关生成的 `LLAMA_ARG_NO_` 兼容名；这不是所有后端环境变量的全集。后端如 CUDA 的环境配置、CMake 构建选项、HTTP 请求 JSON 字段和独立工具解析器，不属于本表的命令行注册集合。

