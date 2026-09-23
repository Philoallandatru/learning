# llama.cpp 独立工具参数补编

本补编实际核对本地源码 commit `f5e85d43a048f3d5adefb4c5e29867d8077fba62`，涵盖 `llama-bench`、`llama-quantize`、`llama-gguf-split`、`rpc-server` 的独立解析器全部公开选项、`llama` 分发入口，以及重要构建选项。它与 `common/arg.cpp` 公共参数目录配套使用。版本更新后请以所运行二进制的 `--help` 为准；源码支持不意味着当前构建、后端、模型全部支持。

## 1. 先区分入口，不能跨程序照抄短参数

`llama` 是分发程序：`serve`（别名 `server`）、`cli`（`client`）、`download`（`get`）、`update`、`completion`（`complete`）、`bench`、`batched-bench`、`fit-params`、`quantize`、`perplexity`、`version`、`licenses`（`credits`）、`help`。部分命令默认隐藏，用 `llama help all` 查看；`version/licenses/help` 还接受带 `--` 的形式。`update` 仅安装器构建支持自动更新。子命令传递给对应解析器，`llama bench` 与独立 `llama-bench` 使用同一实现。

特别注意：此版本 `llama-cli` 是交互界面，传统文本补全另有 `llama-completion`；不能把旧版 CLI 文档视为本版所有入口的保证。

来源：[分发命令及别名](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/app/llama.cpp#L63-L80)、[帮助与分发实现](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/app/llama.cpp#L97-L157)。

## 2. llama-bench：全部独立参数

以下性能关系是对相应设置作用的机制解释，不是实测提速保证。默认测试 `pp512` 和 `tg128`，重复 5 次，逻辑 batch 2048、物理 ubatch 512、K/V 为 f16、GPU 层数 -1、Flash Attention auto。`-p` 和 `-n` 默认产生两类独立测试；需要提示词和生成合在一次测试时使用 `-pg`。

| 参数（同格为别名） | 含义、默认或取值 | 如何影响性能／测量 |
|---|---|---|
| `-h, --help` | 输出帮助 | 无推理影响 |
| `--numa` | `distribute/isolate/numactl`，默认禁用 | 改 CPU NUMA 策略；内存所在节点和线程位置会影响带宽及跨节点延迟 |
| `-r, --repetitions N` | 每组重复次数，默认 5 | 增加可信度及运行耗时，不是提升推理速率 |
| `--prio N` | -1/0/1/2/3，默认 0 | 调整调度优先级，可能减少竞争噪声 |
| `--delay N` | 测试间等待秒数，默认 0 | 可减轻持续热负载，必须记录以保证可比性 |
| `-o, --output` | stdout 格式 `csv/json/jsonl/md/sql`，默认 md | 报表格式 |
| `-oe, --output-err` | stderr 报表格式，默认不输出 | 同上 |
| `--list-devices` | 列设备并退出 | 查看当前构建实际设备名称 |
| `-v, --verbose` | 详细日志 | 排障；避免把日志耗时当模型速度 |
| `--progress` | 显示测试进度 | 观测功能 |
| `--no-warmup` | 跳过暖机 | 首次图构建等开销可能污染稳态测量 |
| `-fitt, --fit-target MiB` | 启用自动内存适配，并留指定显存余量；默认关闭 | 可能改变实际 offload/上下文配置；比较时记录实际日志 |
| `-fitc, --fit-ctx N` | 自动适配最小上下文 | 适配约束影响显存；帮助标 4096，但本版 bench 初始化传入值为 0，最终取值以适配结果为准 |
| `-rpc, --rpc servers` | 注册逗号分隔的远端设备；仅 RPC 构建可用 | 扩展容量，同时引入网络与同步成本 |
| `-m, --model filename` | GGUF 路径，可多模型比较 | 权重规模、架构和量化决定算力、带宽及内存需求 |
| `-hf, -hfr, --hf-repo repo[:quant]` | HF 模型；缺省倾向 Q4_K_M，找不到时回退首文件 | 下载/选型；下载时间不等于稳态推理时间 |
| `-hff, --hf-file filename` | 指定仓库文件，优先于 quant 后缀 | 选型 |
| `-hft, --hf-token token` | 访问令牌；也读 HF_TOKEN | 鉴权，非推理调优 |
| `--offline` | 只用缓存、不联网 | 资源获取 |
| `-p, --n-prompt N` | 提示 token 数，默认 512 | 测 pp；越长工作量越大，不能只比较总秒数 |
| `-n, --n-gen N` | 生成 token 数，默认 128 | 测 tg；生成中上下文逐步增长 |
| `-pg P,G` | 一次联合 pp+tg 测试；可重复指定 | 联合 t/s 混合了不同阶段，不等于纯 decode t/s |
| `-d, --n-depth N` | 预先填入的上下文深度，默认 0 | 填充在计时外；用于观察长上下文下的 pp/tg，尤其 attention 读 KV 成本 |
| `-b, --batch-size N` | 逻辑 batch，默认 2048 | 每次提交 token 上限；主要影响 prefill 调用数量 |
| `-ub, --ubatch-size N` | 物理 microbatch，默认 512 | 决定实际图块大小、临时缓冲及 prefill 并行度 |
| `-ctk, --cache-type-k TYPE` | K 缓存类型，默认 f16 | 更低位宽降低 KV 容量/流量；有精度、解量化及后端支持代价 |
| `-ctv, --cache-type-v TYPE` | V 缓存类型，默认 f16 | 同上；所需后端/FA 支持以运行日志为准 |
| `-t, --threads N` | CPU 线程数，默认自动估算数学线程 | CPU 并行和内存带宽；过多线程可能更慢 |
| `-C, --cpu-mask mask` | CPU 亲和掩码，默认 0x0 | 减少迁移、选择核心/NUMA 节点 |
| `--cpu-strict 0/1` | 严格亲和，默认 0 | 限制可调度核心 |
| `--poll 0..100` | 线程轮询，默认 50 | 响应延迟与 CPU 占用/功耗权衡 |
| `-ngl, --n-gpu-layers N` | GPU 层数，默认 -1 | 更多 offload 通常减少 CPU 计算，但受显存/传输瓶颈约束 |
| `-ncmoe, --n-cpu-moe N` | 前 N 层 MoE 权重在 CPU，默认 0 | 省显存，可能增加 CPU 带宽和设备交互 |
| `-sm, --split-mode MODE` | `none/layer/row/tensor`，默认 layer | 多设备拆分方式；容量、并行和通信开销权衡 |
| `-mg, --main-gpu I` | 主 GPU，默认 0 | 受拆分模式影响；决定部分数据/计算放置 |
| `-nkvo, --no-kv-offload 0/1` | 禁止 KV offload，默认 0 | 省显存，但改变 attention 数据放置和速度 |
| `-fa, --flash-attn on/off/auto` | 默认 auto | 改 attention 内核和中间数据流量；长上下文可能显著受益，依赖后端/形状 |
| `-dev, --device dev0/dev1/...` | 使用的设备，默认自动 | 注意这里单组设备用 `/`，逗号用于测试组合 |
| `-lm, --load-mode MODE` | `auto/none/mmap/mlock/mmap+mlock/dio` | 加载、页缓存、驻留策略；冷启动与内存压力，不直接等于矩阵内核加速 |
| `-mmp, --mmap 0/1` | 已弃用；用 load-mode | 加载策略 |
| `-dio, --direct-io 0/1` | 已弃用；用 load-mode | 加载策略 |
| `-embd, --embeddings 0/1` | embeddings 模式，默认 0 | 测试任务不同，不能与生成无条件对比 |
| `-ts, --tensor-split a/b/...` | 多设备比例；默认自动 | 平衡容量与工作量；逗号用于分隔多组比例 |
| `-ot, --override-tensor pattern=buffer;...` | 精细指定张量缓冲放置 | 可让瓶颈张量走不同设备，也可能引入额外传输 |
| `-nopo, --no-op-offload 0/1` | 禁用算子 offload，默认 0 | 改算子调度位置 |
| `--no-host 0/1` | 不使用额外 host buffer，默认 0 | 改主机缓冲/传输路径，效果依后端 |

逐项来源：[帮助全文与默认值](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L366-L476)、[真实解析器](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L510-L1078)、[上下文参数映射](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L1278-L1295)、[深度填充及计时边界](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L2380-L2452)。

### bench 语法陷阱与正确案例

多数向量测试参数允许逗号列表或重复提供，整数扫描还支持 `first-last`、`first-last+step`、`first-last*mult`。**这不适用于所有标量控制参数**；例如 `-r` 是单个重复次数，`-pg` 一次必须恰好两个整数。多个测试维度会形成组合，先小范围扫描以免测试数量爆炸。

本版 bench 的 KV 类型解析只接受 `f16/bf16/q8_0/q4_0/q4_1/q5_0/q5_1/iq4_nl`；不能直接假设公共解析器支持的每种类型它都接受。

```powershell
# 同一模型，分别测纯 prefill 与纯 decode 的 CPU 线程数影响
llama-bench -m model.gguf -ngl 0 -t 4,8,12 -p 512 -n 128 -r 5 -o json

# 隔离已有上下文长度对 decode 的影响；-p 0 去掉独立 pp 测试
llama-bench -m model.gguf -ngl 99 -p 0 -n 128 -d 0,4096,16384 -fa on -r 5

# 设备名称先用 --list-devices 查询；每组比例内部是 /，外部逗号表示比较两组
llama-bench -m model.gguf -dev CUDA0/CUDA1 -ts 1/1,2/1 -sm layer -p 2048 -n 128

# 保持逻辑 batch 固定，比较实际 microbatch；检查显存峰值
llama-bench -m model.gguf -ngl 99 -p 4096 -n 0 -b 2048 -ub 128,256,512 -r 5
```

以上为可改写的测试方案，未声称在当前设备运行过。bench 使用合成 token 和模型计算测试，不等同 HTTP 请求、真实 tokenizer、采样器、排队及网络在内的端到端 TTFT；服务延迟需另外测 `llama-server`。来源：[测试 token 与 decode 实现](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L2114-L2185)、[参数组合](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/llama-bench/llama-bench.cpp#L1300-L1438)。

## 3. llama-quantize：全部选项与位置参数

语法：`llama-quantize [选项] input.gguf [output.gguf] TYPE [nthreads]`。这是生成新模型文件的离线转换器，不是在线推理设置。`nthreads` 为末尾位置参数，不能照搬 `-t`。选项置于输入路径之前。

| 参数 | 含义 | 性能/质量关系 |
|---|---|---|
| `--help` | 帮助 | 此版本帮助经 usage 退出码为 1，不能只靠退出码认定帮助失败 |
| `--allow-requantize` | 允许对已量化权重再次量化 | 可能叠加误差；不是恢复原始精度 |
| `--leave-output-tensor` | 不量化输出权重 | 可能保留质量但增加容量/带宽 |
| `--pure` | 尽量用指定单一类型，关闭常规混合量化策略 | 与默认混合策略的质量、体积不同 |
| `--imatrix file` | 使用重要性矩阵指导量化 | 优化误差分配；需要合适校准数据 |
| `--include-weights name` | 仅在指定权重使用重要性矩阵，可重复 | **并非只量化这些权重** |
| `--exclude-weights name` | 指定权重不用重要性矩阵，可重复 | 与 include 互斥；并非不量化这些权重 |
| `--output-tensor-type TYPE` | 指定 output.weight 类型 | 定向精度/内存权衡 |
| `--token-embedding-type TYPE` | 指定 embedding 类型 | 定向精度/内存权衡 |
| `--tensor-type name=type` | 对匹配张量指定类型，可重复 | 混合精度调优；如 `attn_q=q8_0` |
| `--tensor-type-file file` | 从空格/换行分隔的上述规则文件读取 | 批量混合精度配置 |
| `--prune-layers L0,L1,...` | 从模型剪除指定层 | 减计算/容量，同时改变模型，须独立验证质量 |
| `--keep-split` | 保留输入的分片结构 | 文件组织，不是多 GPU 计算分片 |
| `--override-kv KEY=TYPE:VALUE` | 修改输出 GGUF 元数据，可重复 | 影响模型解释方式，不是 KV cache 量化 |
| `--dry-run` | 估算量化后大小，不实际量化 | 规划磁盘及容量 |
| `--max-buffer-size MiB` | 单张量量化期间的缓冲上限，默认 8192 MiB | 降低离线 RAM 需求；不改变推理 KV 缓存大小 |
| `input.gguf` | 输入路径 | 建议从高精度权重生成各候选，避免链式重量化 |
| `[output.gguf]` | 可省略输出路径 | 省略时生成 `ggml-model-TYPE.gguf`（保留分片时名称相应变化） |
| `TYPE` | 类型名（大小写不敏感）或相应数值枚举 | 决定模型文件、权重带宽、精度及内核支持 |
| `[nthreads]` | 离线量化线程数 | 影响转换速度，不是生成后模型的推理线程数 |

此 commit 完整类型名：`Q1_0, Q2_0, Q4_0, Q4_1, MXFP4_MOE, Q5_0, Q5_1, IQ2_XXS, IQ2_XS, IQ2_S, IQ2_M, IQ1_S, IQ1_M, TQ1_0, TQ2_0, Q2_K, Q2_K_S, IQ3_XXS, IQ3_S, IQ3_M, Q3_K, IQ3_XS, Q3_K_S, Q3_K_M, Q3_K_L, IQ4_NL, IQ4_XS, Q4_K, Q4_K_S, Q4_K_M, Q5_K, Q5_K_S, Q5_K_M, Q6_K, Q8_0, F16, BF16, F32, COPY`。`Q3_K/Q4_K/Q5_K` 分别是 `_M` 的别名；`COPY` 只复制、不量化。这里是模型量化配方，不能与所有 GGML 底层张量类型等同。

```powershell
llama-quantize --dry-run model-f16.gguf Q4_K_M
llama-quantize --imatrix imatrix.dat model-f16.gguf model-q4km.gguf Q4_K_M 8
```

更低位宽减少权重读取和驻留内存，尤其可能改善带宽瓶颈下的 decode；但低位内核效率、解量化代价、模型质量都可能改变，不能按比特比值线性推算提速。量化帮助中的特定模型容量/PPL 数字不是所有模型和任务的质量保证。

来源：[类型完整表](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/quantize/quantize.cpp#L35-L77)、[帮助说明](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/quantize/quantize.cpp#L122-L181)、[选项解析](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/quantize/quantize.cpp#L395-L494)、[位置参数](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/quantize/quantize.cpp#L559-L618)。

## 4. llama-gguf-split：全部参数

语法 `llama-gguf-split [options] GGUF_IN GGUF_OUT`。这些选项操作磁盘上的 GGUF 文件组织，**不等于** `--split-mode` 多 GPU 计算分配；本身不压缩权重、不减少算术量。

| 参数 | 含义及约束 |
|---|---|
| `--help`（帮助还显示 `-h`） | 输出帮助；此 commit 解析循环只进入 `--` 开头选项，因此建议使用 `--help`，不要依赖 `-h` |
| `--version` | 版本及构建信息 |
| `--split` | 拆分，默认操作 |
| `--merge` | 合并分片；与显式 split 互斥 |
| `--split-max-tensors N` | 每片最多张量数，默认 128 |
| `--split-max-size N(M/G)` | 每片大小限制；与 max-tensors 互斥；M/G 为十进制 10^6/10^9 字节 |
| `--no-tensor-first-split` | 首片仅元数据、不放张量 |
| `--dry-run` | 只打印拆分计划 |
| `--delete-splits` | 合并过程中删除原分片释放磁盘；失败可能无法恢复，源码明确警告，仅明确接受风险时使用 |
| `GGUF_IN GGUF_OUT` | 输入及输出；拆分时输出为命名前缀，合并时输入指定首分片 |

来源：[完整帮助与单位转换](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/gguf-split/gguf-split.cpp#L40-L90)、[实际解析与互斥](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/gguf-split/gguf-split.cpp#L93-L193)。

## 5. rpc-server：全部参数

| 参数 | 含义 | 性能关系 |
|---|---|---|
| `-h, --help` | 帮助 | 无 |
| `-t, --threads N` | 远端 CPU 设备线程数，默认硬件并发数一半且至少 1 | 仅相应 CPU 设备计算；不是远端 GPU 内核线程数 |
| `-d, --device devices` | 设备列表；帮助用逗号，解析也接受 `/` | 控制暴露哪些计算设备；默认先所有非 CPU 设备，无加速设备才退到 CPU |
| `-H, --host HOST` | 监听地址，默认 127.0.0.1 | 决定连接路径；绑定非本机地址时源码明确警告不可暴露公网 |
| `-p, --port PORT` | 端口，默认 50052 | 网络地址，不是 prompt |
| `-c, --cache` | 本地文件缓存，默认关闭 | 加速重复传输/加载的数据复用；不是模型会话的 KV prefix cache |

RPC 是远端后端，不是 OpenAI HTTP 推理服务。多机增加可用容量，也会增加网络传输和同步成本；网络/拆分不适合时单请求可能更慢。来源：[完整默认及解析器](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/rpc/rpc-server.cpp#L171-L247)、[设备选择及缓存初始化](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/rpc/rpc-server.cpp#L250-L325)。

## 6. 构建选项与环境变量：重要项，不是穷尽清单

这些要么在 CMake 配置阶段以 `-D名字=值` 设置，要么在进程环境中设置，**不是**传给 llama-server 的同名命令行开关。

| 项 | 作用和性能解释 |
|---|---|
| `GGML_NATIVE` | 为构建机器 CPU 优化；跨机器分发应检查指令集兼容性 |
| `GGML_CUDA / GGML_METAL / GGML_VULKAN / GGML_SYCL / GGML_RPC` | 编译相应后端；没构建的能力不能靠运行开关凭空启用 |
| `GGML_BLAS` | 启用 BLAS；可能改善大矩阵 prefill，非普遍 decode 加速 |
| `GGML_CUDA_FORCE_MMQ` | 强制使用 MMQ 路径；用于比较量化矩阵内核 |
| `GGML_CUDA_FORCE_CUBLAS` | 强制 cuBLAS 路径；不同 batch/硬件可能各有优势；勿和 MMQ 强制项当成独立可叠加加速 |
| `GGML_CUDA_FA` | 编译 CUDA Flash Attention 内核 |
| `GGML_CUDA_FA_ALL_QUANTS` | 编译更多量化类型的 FA 内核，扩大支持范围并增加构建成本 |
| `GGML_CUDA_GRAPHS` | CUDA Graphs 支持；可降低重复 GPU 工作的 CPU 提交开销 |
| `GGML_CUDA_NCCL` | NVIDIA 集体通信库支持，关联多 GPU 通信 |
| `GGML_CUDA_NO_PEER_COPY` | 禁用 GPU 间 peer copy；改变传输路径，可能降低多 GPU 性能 |
| `GGML_CUDA_NO_VMM` | 禁用 CUDA VMM 尝试；兼容性/显存管理选择 |
| 环境变量 `GGML_CUDA_DISABLE_GRAPHS` | 存在即禁用 CUDA Graphs；即使值是 `0`，源码也按“存在”处理 |

来源：[GGML CMake 选项](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/ggml/CMakeLists.txt#L123-L255)、[CUDA Graphs 环境变量存在性判断](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/ggml/src/ggml-cuda/common.cuh#L1258)。

## 7. 其他工具及完整性边界

`llama-batched-bench` 使用公共参数解析器 `LLAMA_EXAMPLE_BENCH`，不同于上面的独立 `llama-bench`；`llama-tokenize`、`llama-export-lora` 也走公共参数目录。`llama-imatrix`、`llama-perplexity`、`llama-cvector-generator`、`llama-tts`、`llama-mtmd-cli` 等应按自身 `--help` 的目标过滤查看，不能把公共目录内所有参数都传给任意工具。构建是否生成它们受 CMake 条件限制。

另有 Metal 开发者离线调优工具 `llama-tuning fa-vec`：`-h/--help`、`-b name`、`--dtype list`、`--dk list`、`--reps N`、`--seed N`、`--no-cooldown`、`--cool-drift F`、`--cool-eps F`、`--cool-max-wait S`、`--cool-max-retry N`。它扫描内核配置并输出表格行，不是生产推理自动加速开关。各字段分别限制设备、KV 类型、head 大小、重复数、随机种子及温漂冷却判据；开发用途请读完整帮助。

项目还包含模型转换 Python 脚本、`scripts/server-bench.py`、`scripts/compare-llama-bench.py`、`scripts/tool_bench.py`、`gguf-py` 工具、测试与 examples；它们各有解析器及参数空间。本补编明确不把所有开发脚本、C API、HTTP JSON 字段、第三方封装参数混称为“全部 CLI 参数”。针对特定脚本先执行其 `--help` 或检查 `argparse` 定义。核心推理、公共参数及本补编四个常用独立工具的目录已分别建立。

权威入口：[工具构建清单](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/CMakeLists.txt#L16-L44)、[batched-bench 解析入口](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/batched-bench/batched-bench.cpp#L28)、[tokenize 解析入口](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/tokenize/tokenize.cpp#L102)、[export-lora 入口](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/export-lora/export-lora.cpp#L423)、[Metal tuning 帮助与解析](https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/tuning/main.cpp#L18-L79)。
