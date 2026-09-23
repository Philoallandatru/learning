# llama.cpp 推理性能参数：源码证据与调参边界

基准版本：本地 `llama.cpp` 的干净 HEAD `f5e85d43a048f3d5adefb4c5e29867d8077fba62`；检查日期 2026-09-22。以下结论绑定这个提交，而不是所有历史版本。此文为参数指南的研究底稿，不包含新跑分。官方 README 中的旧示例跑分也不能当作当前设备的预测值。本文依本地源码阅读形成，固定提交链接用于复核。

## 1. 先分清性能指标

应同时记录启动/模型加载时间、首 token 延迟 TTFT、prefill token/s、decode token/s、单请求 token 间隔、所有并发请求的总吞吐、p50/p95/p99 延迟、峰值显存/RAM。TTFT 还包含排队、模板与 tokenization、缓存恢复、首轮计算；单看生成 token/s 会漏掉长输入和高并发问题。`llama-bench` 明确不包含 tokenization 与 sampling；所以它适合隔离计算核心，不能直接替代 HTTP 端到端测试。[官方 bench 说明](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/README.md)

机制推断：dense 模型单序列 decode 经常受权重/KV 读取带宽约束；prefill 与多序列合批更能摊薄权重搬运，但可能转为计算受限。瓶颈必须实测确认，不能把这个经验套到所有 MoE、混合 recurrent 模型或所有 GPU。

## 2. batch 与 ubatch

`--batch-size / -b` 是一次逻辑批次最多 token 数；`--ubatch-size / -ub` 是一次物理计算微批次最多 token 数。不是“并发请求个数”；并发 slot 用 `--parallel / -np`。本版上下文初始化先将 causal 模型 n_batch 截至 n_ctx，再将 n_ubatch 截至 n_batch。[参数定义](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1666-L1680)、[截断实现](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-context.cpp#L242-L247)

例如一次逻辑批次 2048 token、ubatch 512，可以理解为最多四段物理工作；实际拆分还受多序列布局等条件约束。提高 ubatch 常有利于 prefill 的 GPU 利用率，但会增加计算缓冲/激活占用；调高 b 而 ub 固定不保证线性提速。单请求逐 token decode 本来每轮只有很少 token，b 从 2048 加到 4096 不会自动把未来 token 并行出来。这是根据自回归依赖和上述拆批机制的推断，而非固定跑分。

特殊情况：本版 server embedding 模式发现 n_batch > n_ubatch 时会把 n_batch 降为 n_ubatch，因为 embedding 需要整段 token 在一个 ubatch 内处理。不要照搬文本生成的拆批配方。[server 初始化](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server.cpp#L142-L150)

## 3. ctx、parallel 与 unified KV 的真实关系

非 unified：n_ctx_seq 大体为 n_ctx / n_seq_max，但本版会按 256 对齐，最终总容量可能重算。unified：n_ctx_seq = n_ctx，表示序列能够使用共享池，不表示每个 slot 都独占一整份 n_ctx。server 的单 slot 上限还取模型训练上下文、可选 `--kv-unified-per-slot` 与 n_ctx_seq 的最小值。[上下文计算](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-context.cpp#L287-L303)、[slot 上限](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server-context.cpp#L4000-L4008)

本版 server 的 auto parallel 会解析成 4 个 slot 并打开 unified KV。显式 `--parallel 4` 不应被当作和 auto 完全等价，建议比较实验显式给 `--kv-unified` 或 `--no-kv-unified`。`--kv-unified-per-slot 8192` 在没有固定 -c 时尝试把总池设为 parallel × 8192；显式 `-c 0` 则被识别为请求模型上下文，不走这个自动乘法。[server 参数解析](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server.cpp#L152-L170)

例如 `-c 32768 -np 4 --no-kv-unified` 可规划四个约 8192 token slot（仍受训练上下文上限约束）；`-c 32768 -np 4 --kv-unified` 是共享 32768 token 池，不能承诺四路各用 32768。提高 parallel 可能提升总吞吐，但每位用户生成速度和排队后的尾延迟不一定改善。提高 c 增加容量预算；真正已用上下文增长还会增加后续注意力读取工作，不能将“容量设大”与“已经填满”混为一谈。

普通 dense/GQA attention 的粗略 KV 公式为：`token 容量 × 层数 × KV heads × head dimension × (K 每元素字节 + V 每元素字节)`。混合层、SWA、MLA、recurrent 状态、padding、量化块元数据会使该公式不适用或需要修正。例：32 层、8 KV heads、head_dim=128、f16 K/V、8192 token，纯 KV 约 1 GiB；32768 token 约 4 GiB。这是明确假设下的算术示例，不是某个实际模型的显存读数。

## 4. Flash Attention 与 KV 量化

`-fa on/off/auto` 控制 Flash Attention。源码在满足条件时构造融合 attention 运算，减少中间结果的物化；因此长上下文/大 prefill 时有望降低工作内存和访存开销，实际收益依 backend、head 形状和 kernel 支持。[attention 图构建](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-graph.cpp#L2564-L2586)

`--cache-type-k` 与 `--cache-type-v` 改的是 KV，不是 GGUF 权重量化。q8_0/q4_0 等可以减小 KV 占用及带宽压力，但量化解量化、kernel 适配和精度风险意味着“更省显存”不等于“更快”。本版量化 V 要求 FA：auto 会启用，off 会报错。量化类型还受 head dimension 与块大小约束；MLA / DeepSeek4 不允许 K/V 选择不同类型。[兼容性检查](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-context.cpp#L3659-L3708)

建议实验分别测 f16/f16、q8_0/q8_0，再试更低精度，同时测长上下文质量；不要仅拿短文本 tokens/s 判断量化可靠性。

## 5. GPU offload、多 GPU 与 MoE

`-ngl auto/all/N` 控制放入显存的层；本版 -1 表示 auto、-2 表示 all。旧教程“用 -1 就是全上 GPU”的口径不能照抄。`--device` 选择设备，`--override-tensor` 能进一步改变张量放置，`--cpu-moe` 把所有专家权重放 CPU，`--n-cpu-moe N` 只对前 N 层专家这么做；`--n-cpu-ffn N` 针对 dense FFN。它们首先解决容量问题，CPU 内存带宽和跨设备传输可能成为新瓶颈。[参数源码](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2775-L2836)

| split-mode | 实际分法 | 性能判断 |
|---|---|---|
| none | 单 GPU | 避免卡间通信，要求容量足够 |
| layer | 各卡分层和相应 KV | 扩大容量；可流水，但单请求不等于线性多卡加速 |
| row | 按行分权重并行 | 更多层内协作，互联慢时通信抵消计算收益 |
| tensor | 分权重与 KV 并行，实验性 | 本版要求 FA；backend sampling 回退 CPU |

`--tensor-split 3,1` 是分配比例，不是字节上限；`--main-gpu` 对 none 指模型所在卡，对 row 指中间结果和 KV 所在卡，不是所有模式的通用“主计算卡”。[模式定义](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2829-L2889)、[tensor sampling 限制](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-context.cpp#L1224-L1228)

容易漏掉的限制：layer 模式自动流水要求多设备、完整 GPU 层卸载、KV offload、没有 tensor overrides，而且所有参与设备支持 async/events。CPU MoE 的 tensor override 可破坏这一条件；不要一边 offload 专家到 CPU 一边假定 layer pipeline 仍有效。[流水启用条件](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/src/llama-context.cpp#L427-L458)

`--fit on` 自动调整未固定参数来适应显存；`--fit-target` 是每卡保留的余量，MiB，不是目标模型大小。自动适配可能让两个“相同命令”的实验实际上下文/卸载量不同，必须保留启动日志。`--no-kv-offload` 把 KV 相关工作移出 GPU，能缓解容量却可能拖慢计算；`--no-op-offload` 是另一个开关，不能和前者混淆。[fit 定义](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2891-L2955)

## 6. CPU、加载与缓存

`--threads` 和 `--threads-batch` 分别面向生成与批量/prompt 工作；更多线程会增加并行度，也可能超出内存带宽、造成同步开销或跨 NUMA 访问。分别扫描两个阶段，不以逻辑核心数作为固定最优值。CPU affinity、priority、poll、NUMA 需要按平台理解；poll 提升等待响应可能提高功耗/CPU 占用。[CPU 参数](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1440-L1590)

本版推荐 `--load-mode`，旧 mmap/mlock/direct-io 开关已弃用；混用时只有命令行最后一个相关设置生效。mmap/DirectIO 首先改变加载、page cache 和缺页行为；模型已常驻时不能承诺提高 steady-state token/s。mlock 防换出会占住 RAM，不能增加物理内存。repack 是权重布局优化，可能改变加载成本与运行 kernel 效率。[加载参数](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L2685-L2736)、[混用规则](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L876-L886)

`cache_prompt` 复用相同 token 前缀，使已有部分不再 prefill；最直接改善 TTFT，不是必然提高后续 decode 每秒速度。`--cache-ram` 是 server/CLI prompt state 缓存预算，MiB，0 禁用，-1 不限；不是 GPU KV 大小。`--cache-idle-slots` 可在新任务到来时保存闲置 slot，并在 unified 模式清理其活动状态。缓存存在保存、恢复和驱逐成本，必须比较复用收益与这些成本。[缓存开关](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1713-L1737)

`--cache-reuse N` 是通过 KV shifting 尝试复用的最小 token 块长，不是缓存容量；需 prompt caching，且本版多模态/某些上下文不支持。SWA/recurrent 的恢复还涉及 `--ctx-checkpoints`、`--checkpoint-min-step`，以保存开销/RAM 换减少重算。[cache reuse 定义](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L3591-L3600)、[禁用条件](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server-context.cpp#L1179-L1193)

## 7. 投机解码与采样

投机解码先生成若干候选，再由目标模型验证；收益取决于平均接受 token 数与草稿/验证/回滚总耗时。机制上可用 `每轮接受 token 数 / 每轮总耗时` 理解收益。高接受率、廉价草稿和目标模型昂贵时更可能受益；低接受率或草稿争夺同一 GPU、显存挤压目标模型时可能变慢。ngram 不一定需要另一个模型，但依赖文本重复性；MTP/Eagle3/DFlash/DSpark 需要匹配模型能力，不能拿任意小模型代替专用 sidecar。[投机实现](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/speculative.cpp)

本版 `--spec-draft-n-max`、`--spec-draft-n-min` 是草稿长度参数；旧 `--draft`、`--draft-max`、`--draft-min` 已被明确移除，会报错。`--spec-draft-p-min` 是 greedy 草稿概率阈值，不能当作保证接受率。草稿模型有独立 threads、device、ngl、KV 类型设置。[参数与移除声明](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L4140-L4400)

temperature、top-k、top-p、min-p、typical、penalty、grammar/JSON schema 等主要影响分布、质量和输出长度，也带来采样工作。降低温度不会减少 transformer 的层数或运算量；输出更短可能降低整次耗时，但要分清其与 token/s。复杂 grammar 或大词表概率输出可能让 CPU 采样/序列化显著。`--backend-sampling` 是实验功能，不是无条件全套 GPU 采样，tensor split 会回退 CPU。[采样开关](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/arg.cpp#L1982-L2310)

## 8. 可复现案例设计：只给实验，不编造结果

1. **长输入 TTFT**：固定模型、设备、c、并发 1 和输出长度，扫描 ub=128/256/512/1024，b 固定 2048，再独立比较 FA；记录 prefill、TTFT、峰值显存。不要一次同时更换量化、KV、线程。
2. **显存紧张**：先保持权重与上下文不变比较 f16/f16 和 q8_0/q8_0 KV；再单独缩小 c；最后才比较减少 ngl/CPU MoE。验证哪些组合仍完全驻留 GPU，以及长文本质量。
3. **多人服务**：固定每请求输入/输出分布，分别测试 1/2/4/8 并发，报告总体输出 token/s 和每请求 p95 TTFT/TPOT；明确 unified 或非 unified，保证上下文容量足以完成请求。
4. **缓存 ROI**：相同前缀重复访问与完全新前缀分别测量，区分活跃 slot 命中、RAM 恢复、冷 miss；保持模板和 token 前缀相同，记录复用 token 数及恢复耗时。
5. **投机收益**：无投机对照，加草稿后扫描 n_max，记录接受率、accepted tokens/round、显存、目标/草稿耗时；不只报告最有利文本。

核心计算基准可用本版 bench 的多值扫描，例如 `llama-bench -m model.gguf -p 2048 -n 0 -b 2048 -ub 128,256,512,1024 -fa on -r 5 -o json`。长上下文 decode 要加 `-d 8192` 等 context depth，不能用空 KV 的 tg128 代表 32K 会话。bench 的多值参数会做组合测试；注意其选项语法不一定等同 server，例如某些 bool 在 bench 要写 0/1。[bench 选项与 context depth](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/README.md)

每次记录 commit、编译 backend/驱动、模型文件与量化、真实生效的 ngl/c/b/ub/KV、提示 token 数、输出 token 数、并发、缓存冷热、warmup、采样与 seed、重复次数、温度/功耗环境。保留均值及离散度；服务器端补端到端百分位。不要把加载时间和热运行混算；不要把前缀缓存命中的 prefill “等效速度”当作实际全量计算速度。[bench 输出字段](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/README.md#output-formats)
