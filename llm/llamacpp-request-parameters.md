# llama.cpp 通用 completion 请求参数完整注册表

版本基线：`f5e85d43a048f3d5adefb4c5e29867d8077fba62`，核对日期 2026-09-22。本表依据 `tools/server/server-schema.cpp` 的 `make_llama_cmpl_schema()`，逐项覆盖其有效注册内容：**66 个顶层字段、1 个嵌套子字段、6 个别名，共 73 个已注册名称/路径**。容器 `stream_options` 也计作一个顶层字段；嵌套子字段全名为 `stream_options.include_usage`。

这是通用 completion 的任务参数 schema，不是整个 HTTP 服务所有 JSON 字段清单。`prompt`、`id_slot` 等由入口/任务分发处理；聊天的 `messages`、`tools`、`tool_choice`、`response_format` 等还经过兼容层。它们不在这 66 项里，不代表端点不支持。embedding、rerank、实时控制、slot save/restore、模型管理等有独立接口。内部聊天解析字段虽在此注册，也不宜当成普通客户端必须手填的稳定公共契约。[注册与执行源码][schema]、[端点文档][server]

## 默认值和优先级如何读

请求解析先从服务器 `params_base` 继承 sampling、speculative、n_keep、n_predict、n_cache_reuse、cache_prompt、stop、SSE ping 和 reasoning format，再应用请求覆盖。因此表内“初值”是未被服务器启动配置改动时的源码初值，不是每台服务器的强制默认。普通字段主名称优先于别名，按别名注册顺序寻找首个非 null 值；通用字段处理将 null 当作未提供，复杂自定义 handler 仍有例外，建议省略不使用的键。`set_limits` 越界会截断，`set_hard_limits` 越界报错。[解析流程][schema]、[字段类型][header]、[任务初值][task]、[采样初值][common]

典型差异：本版 repeat_penalty 初值 **1.0**，不是手写 README 中的 1.1。repeat_last_n 与 dry_penalty_last_n 在请求 schema 的硬下限均为 0，不能把 CLI 的负数特殊值机械套来。temperature 的软范围从 0 开始，负数会被截为 0。samplers 初始链为 penalties、dry、top_n_sigma、top_k、typical_p、top_p、min_p、xtc、temperature；是否真正有效还取决于各采样器的禁用值与采样实现。[源码初值][common]、[schema 限制][schema]

## A. 输出、流式与工作量（16 个顶层 + 1 个嵌套）

| 字段 | 中文作用与边界 | 性能关系 |
|---|---|---|
| `verbose` | 响应附加 `__verbose`；默认由服务器 verbosity 是否 >9 决定 | 增大诊断构造与传输量 |
| `timings_per_token` | 每次响应包含处理/生成速度信息；任务初值 false | 增加观测与返回开销 |
| `stream` | 逐步返回输出；初值 false | 改善首字可见时间，不保证前向更快 |
| `stream_options` | 流式选项对象 | 本身是容器 |
| `stream_options.include_usage` | 是否在流中返回 usage；初值 false | 少量统计与传输开销 |
| `cache_prompt` | 尝试复用已有 token 前缀状态；初值 true，可被启动配置覆盖 | 主要节省重复 prefill，降低 TTFT |
| `return_tokens` | 返回原始生成 token ID；初值 false | 增加响应数据量 |
| `return_progress` | 流模式报告 prompt 处理进度；初值 false | 提供长 prefill 反馈，增加事件量 |
| `sse_ping_interval` | 静默期间 SSE 注释 ping 间隔，秒；初值 30，-1 禁用 | 连接保活，不是模型推理加速 |
| `n_predict` | 最多生成 token；别名 `max_completion_tokens`、`max_tokens`；>=-1；0 只评估 prompt 进缓存 | 直接控制总工作与资源占用时间 |
| `n_indent` | 生成代码的最低缩进空白数；>=0，初值0 | 改变可生成内容及约束行为，非通用加速项 |
| `n_keep` | context shift 时保留原 prompt token 数；-1 保留全部，初值0 | 影响超窗后的保留信息与后续工作 |
| `n_discard` | shift 时 n_keep 后可丢弃 token 数；0 表示默认约半个上下文，>=0 | 影响移位频率、历史长度与重算/注意力成本 |
| `n_cmpl` | 每个输入 prompt 的输出份数，别名 `n`；1 到服务器 parallel；初值1 | 总生成工作倍增，消耗更多 slot |
| `n_cache_reuse` | 尝试 KV shifting 复用的最小块长，>=0；初值0 | 块匹配/移动成本与重复 prefill 节省之间权衡 |
| `t_max_predict_ms` | 生成阶段软时间限制，毫秒，>=-1；初值-1；从首 token 计时，超过后还需满足换行条件 | 限制生成工作；不是含排队/prefill的整请求硬超时 |
| `response_fields` | 返回字段白名单；斜杠路径可提取并展平；缺失字段直接省略 | 可减少响应体；不意味着被隐藏内容从未计算 |

本节直接对应 [schema 开头定义][schema]，默认继承关系见其 `eval_llama_cmpl_schema()`。

## B. 概率分布、惩罚和采样（28 个顶层）

共同原则：这些字段主要改变输出分布、质量、长度及采样成本；不减少 transformer 层数。复杂采样在小模型、大词表或 CPU 紧张时可能成为瓶颈，不能凭降低 temperature 推断 GPU token/s 必然提高。[采样注册][schema]、[采样实现][sampling]

| 字段 | 中文作用、初值或边界 | 性能关系 |
|---|---|---|
| `top_k` | 保留最高概率 K 个候选；初值40，0禁用，软下限0 | 排序/筛选成本与分布变化 |
| `top_p` | 累积概率截断；初值0.95，软范围0..1，1禁用 | 概率与候选筛选 |
| `min_p` | 相对最大概率的最低阈值；初值0.05，0禁用 | 筛选开销、质量变化 |
| `top_n_sigma` | 保留离最高 logit 不超过若干标准差的候选；初值-1，负数禁用 | 统计/筛选成本 |
| `xtc_probability` | 应用 XTC 移除候选的概率；初值0，范围0..1 | 改变分布，额外采样步骤 |
| `xtc_threshold` | XTC 移除阈值；初值0.1，>0.5禁用XTC | 同上 |
| `typical_p` | locally typical 采样阈值；初值1禁用；schema未指定显式范围 | 熵与候选计算 |
| `temperature` | 温度；初值0.8，软下限0；0为greedy | 改变随机性，不是模型主计算开关 |
| `dynatemp_range` | 动态温度幅度；初值0禁用 | 动态温度与熵计算 |
| `dynatemp_exponent` | 熵到温度映射指数；初值1 | 调整上述映射 |
| `repeat_last_n` | 重复惩罚历史窗口；初值64，硬下限0，0禁用 | 窗口影响历史处理成本 |
| `repeat_penalty` | 重复 token 惩罚；初值1.0禁用 | 改变质量与输出长度 |
| `frequency_penalty` | 按出现频次惩罚；初值0禁用 | 历史统计与质量变化 |
| `presence_penalty` | 按是否出现过惩罚；初值0禁用 | 历史统计与质量变化 |
| `dry_multiplier` | DRY 重复片段惩罚倍率；初值0禁用 | 长重复窗口可能增加采样成本 |
| `dry_base` | DRY 指数底数；初值1.75；小于1时回退服务器默认 | 改变惩罚强度，不直接改变主计算 |
| `dry_allowed_length` | 超过此长度的重复片段开始受惩罚；初值2，>=0 | 重复检测与输出质量 |
| `dry_penalty_last_n` | DRY 扫描历史窗口；初值64，硬下限0，0禁用 | 大窗口可能增加 CPU 工作 |
| `mirostat` | 0关闭，1/2两种Mirostat；初值0，软范围0..2 | 改变采样路径，不能假设与所有筛选器同时生效 |
| `mirostat_tau` | Mirostat 目标熵；初值5 | 分布控制 |
| `mirostat_eta` | Mirostat 学习率；初值0.1 | 分布适应速率 |
| `adaptive_target` | 自适应采样目标，初值-1禁用；非负有效值0..1 | 自适应分布和历史状态 |
| `adaptive_decay` | 自适应 EMA 衰减，初值0.9；硬范围0..0.99 | 有效历史约1/(1-decay)，影响分布适应 |
| `seed` | 随机种子；-1随机 | 有助复现实验，但不保证跨backend/batch位级一致 |
| `n_probs` | 返回每token最高N个概率；别名 `logprobs`；初值0 | 额外概率处理、拷贝、序列化，响应体随输出增长 |
| `min_keep` | 候选筛选至少保留N个token；初值0，>=0 | 防止筛选过少，影响后续采样 |
| `backend_sampling` | 尝试后端采样；初值false | 可能减少CPU同步/搬运；支持组合有限 |
| `post_sampling_probs` | 返回采样链之后的top n_probs概率；初值false | 额外返回概率工作，需与n_probs一起理解 |

## C. 适配器、语法和聊天解析（12 个顶层）

| 字段 | 中文作用与边界 | 性能关系 |
|---|---|---|
| `lora` | 请求适配器列表，每项 `id`、`scale`；未列出的适配器scale为0 | 增加适配器计算；配置不同可能限制合批 |
| `dry_sequence_breakers` | DRY 分段字符串数组；必须非空；初值换行、冒号、双引号、星号 | 影响重复扫描范围与质量 |
| `json_schema` | JSON schema 转为约束语法；别名通道 `grammar` 接收GBNF字符串；建议只传一个 | schema构建和逐token约束开销 |
| `grammar_lazy` | 触发后才启用语法约束；初值false | 可避免前段逐token语法工作，需合法触发器 |
| `chat_format` | 内部聊天格式整数枚举 | 影响输出解析；不建议客户端硬编码内部枚举 |
| `reasoning_format` | 思考内容输出/解析格式 | 隐藏或改格式不等于省去思考计算 |
| `generation_prompt` | 已由模板预填充的assistant生成前缀，设置parser及sampling状态 | 影响语法/预算初始状态；不是任意附加文本的通用入口 |
| `parse_tool_calls` | 是否将生成输出解析成工具调用 | 解析开销和输出结构变化 |
| `chat_parser` | 聊天parser配置字符串 | 解析行为与成本；主要为内部兼容流程 |
| `continue_final_message` | 是否继续最后消息；通过continuation解析器解释输入 | 改变对话续写方式和前缀 |
| `echo` | 是否将输入回显到输出 | 增大返回体与解析/传输量 |
| `preserved_tokens` | 将能分词为单一token的指定字符串加入preserved集合 | 用于grammar触发/特殊token保留；多token字符串不会被这个handler强行合并 |

**本版代码与描述有一处冲突：** json_schema 的描述声称它优先，但 handler 只有在存在 json_schema 且没有 grammar 时才转换 schema；同时存在时走 grammar 分支。因此不要同时传两个字段，更不要依赖这段描述中的优先级。[实际handler][schema]

`generation_prompt` 的普通字段说明容易读成服务器会在此追加 prompt；但 handler 本身只是赋给 chat_parser_params 和 sampling。公共结构体说明它是“already prefilled”的assistant前缀，用于推进语法状态、初始化预算状态，故表中按实际状态用途说明。[sampling generation_prompt注释][common]

## D. 语法触发、思考预算与终止（10 个顶层）

| 字段 | 中文作用与边界 | 性能关系 |
|---|---|---|
| `grammar_triggers` | 字符串/模式触发器列表；单token word须在preserved_tokens中；lazy模式触发器不能为空 | 推迟约束执行，也增加匹配成本 |
| `reasoning_control` | 按需建立预算采样器，支持运行时结束思考；初值false | 控制思考长度，不自动提高每token速度 |
| `reasoning_budget_tokens` | 思考token预算；>=-1，初值-1禁用 | 限制思考总工作与正文出现延迟，可能影响质量 |
| `reasoning_budget_start_tag` | 思考预算段开始标记，字符串会分词 | 正确识别预算区间的前提 |
| `reasoning_budget_end_tags` | 结束标记数组；第一项用于预算耗尽时强制结束；别名 `reasoning_budget_end_tag` 为单字符串 | 结束预算区间，影响实际token数量 |
| `reasoning_budget_message` | 强制结束标记前插入的信息；需有end tag | 额外注入token并改变模型后续行为 |
| `logit_bias` | token偏置；数组[token,bias]或对象映射；false可禁用token，字符串可能分成多个token | 额外logit处理，影响质量/长度 |
| `ignore_eos` | 忽略结束token；初值false，实际向EOG token添加偏置 | 可能显著延长生成，应保留输出上限 |
| `stop` | 停止字符串或数组；命中字符串不包含在最终输出；空有效列表回退CLI stop默认 | 直接控制输出长度；不是取消所有默认停止词的通用办法 |
| `samplers` | 采样器名称数组或单个采样器缩写字符串 | 执行顺序改变分布与CPU开销 |

本节字段类型及handler全部见 [schema后半部分][schema]。表的“性能关系”是根据代码路径的机制推断，不代表已测量加速比例。

## 别名总表（6 个）

| 规范字段 | 已注册别名 | 注意 |
|---|---|---|
| `n_predict` | `max_completion_tokens`、`max_tokens` | 同时给时依主字段及别名顺序选择 |
| `n_cmpl` | `n` | 受服务器parallel硬上限约束 |
| `n_probs` | `logprobs` | 这是通用schema数值别名；不要推断所有OpenAI端点都使用同一类型 |
| `json_schema` | `grammar` | 两者结构不同，handler自行处理；只传一个 |
| `reasoning_budget_end_tags` | `reasoning_budget_end_tag` | 复数数组与单数字符串不是相同JSON类型 |

## 源码出现但本版没有注册生效的字段

`speculative.n_max`、`speculative.n_min`、`speculative.p_min`、`speculative.type`、`speculative.ngram_size_n`、`speculative.ngram_size_m`、`speculative.ngram_min_hits` 共 **7项** 位于 `#if 0` 中；不能宣称可以按请求调整。投机参数从服务器配置继承，调整应使用本版本支持的启动参数。`t_max_prompt_ms` 是注释掉的 TODO，亦不计入可用字段。

`grammar_type` 在自定义handler内部读取，但没有独立field注册，作为模板语法内部元信息不计入66项。`lora` 的 `id/scale`、grammar trigger对象成员等属于复合值结构，也不重复计作独立注册字段。以上计数通过排除 `#if 0` 区域及单行注释后，核对所有 `new field_*` 和 `add_alias()` 得到；仅注册总数不构成对所有 HTTP 入口的覆盖承诺。[唯一统计源][schema]

[schema]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server-schema.cpp
[header]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server-schema.h
[task]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/server-task.h
[common]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/common.h
[sampling]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/common/sampling.cpp
[server]: https://github.com/ggml-org/llama.cpp/blob/f5e85d43a048f3d5adefb4c5e29867d8077fba62/tools/server/README.md
