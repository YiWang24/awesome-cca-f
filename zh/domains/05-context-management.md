# Domain 5: Context Management & Reliability（上下文管理与可靠性）

> **权重：15%**  
> 官方文档：[Context Windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) | [Long Context](https://docs.anthropic.com/en/docs/build-with-claude/long-context-tips)

---

## Task Statement 覆盖范围

| Task | 主题                                 |
| ---- | ------------------------------------ |
| 5.1  | 在长交互中管理上下文以保留关键信息   |
| 5.2  | 设计有效的升级与歧义消解模式         |
| 5.3  | 在多智能体系统中实现错误传播策略     |
| 5.4  | 在大型代码库探索中有效管理上下文     |
| 5.5  | 设计人工审核工作流与置信度校准       |
| 5.6  | 在多源综合中保留来源链并处理不确定性 |

---

> 核对范围：Task 5.1-5.6，共 24 条 `Knowledge of:` 与 29 条 `Skills in:`，合计 53 条官方要求。

### Task Statement 5.1: Manage conversation context to preserve critical information across long interactions

#### Knowledge of:

- Progressive summarization risks: condensing numerical values, percentages, dates, and customer-stated expectations into vague summaries
  - Progressive summarization 压缩的是文字，但关键事实（订单号、金额、截止日期、用户明确期望）不应该被"大约""前几天"这类模糊词替代。一旦自然语言摘要覆盖这些细节，后续轮次的模型无法恢复原始数值，可能做出错误判断。这类精确信息应进入结构化 case facts block，在摘要之外独立保存。
- The "lost in the middle" effect: models reliably process information at the beginning and end of long inputs but may omit findings from middle sections
  - "Lost in the Middle"是长上下文的已知特性：模型更可靠地处理输入的开头和结尾，中间段落的内容被遗漏的概率更高。多源综合时，关键 claim 摘要和证据来源应放在合并输入的开头，详细文件内容按节分组排在后面。对多轮对话，重要约束可在 system prompt 中持续注入，而不是只出现一次后进入历史消息的中段。
- How tool results accumulate in context and consume tokens disproportionately to their relevance (e.g., 40+ fields per order lookup when only 5 are relevant)
  - 工具调用是上下文增长的主要来源之一。一次客户信息查询可能返回 40+ 个字段，但退款判断只需要订单号、金额、日期、退货资格和当前状态。将全部字段保留在上下文中，会在后续轮次稀释相关信息，增加模型出错的概率。工具结果在进入 messages 列表前应通过代码过滤，只保留与当前 task 相关的字段。
- The importance of passing complete conversation history in subsequent API requests to maintain conversational coherence
  - Anthropic API 是无状态的：每次 `messages.create` 调用都是独立请求，服务器不存储任何会话状态。多轮对话时，所有相关的历史消息必须由客户端在每次调用时显式传入 messages 数组。如果省略历史，模型无法知道用户已经提供的信息或之前的决策，会像全新对话一样重新开始。

#### Skills in:

- Extracting transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block included in each prompt, outside summarized history
  - Case facts block 是一种结构化注入机制：在每次 API 调用的 system prompt 或对话开头，以 JSON 或 XML 格式传入关键事实，无论会话多长都不会丢失或被摘要覆盖。自然语言摘要适合描述对话过程，但不适合存储精确数值和状态。把两者分开管理，能保证模型在第 30 轮对话时仍然基于与第 1 轮相同的精确数值决策。
- Extracting and persisting structured issue data (order IDs, amounts, statuses) into a separate context layer for multi-issue sessions
  - 用户在同一会话中可能处理多个独立问题（退款问题 + 账单争议），每个问题有自己的订单号、金额、时间线和处理状态。把所有问题的事实混在一段自然语言历史中，模型可能把一个问题的金额应用到另一个问题的判断。为每个 issue 维护独立的结构化状态对象，并在提示中分别引用，是解决多问题会话混乱的标准模式。
- Trimming verbose tool outputs to only relevant fields before they accumulate in context (e.g., keeping only return-relevant fields from order lookups)
  - 字段过滤应该在代码层完成，而不是交给模型"自己忽略不相关字段"。过滤的标准由当前任务决定：退款任务保留订单 ID、金额、日期和退款资格；账单争议任务保留账单号、争议项目和支付状态。这种主动裁剪能控制上下文增长速率，在多轮交互中保持上下文的信噪比。
- Placing key findings summaries at the beginning of aggregated inputs and organizing detailed results with explicit section headers to mitigate position effects
  - 当协调者将多个子智能体的输出合并成一个提示时，结构至关重要。以 executive summary 或关键 claim 列表开头，让模型在进入细节前就建立全局认知；然后按来源分节组织详细证据，配合明确的段落标题；最重要的约束或判断标准可在末尾再次重申。这种结构比把所有内容线性拼接更能保留关键发现。
- Requiring subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis
  - 如果子智能体只返回"该市场 35% 的用户受影响"而不提供数据来源、采集时间、地理范围和测量方法，协调者在合成时无法判断这个数字的适用范围，也无法识别与其他来源的潜在冲突。结构化输出应包含 claim、evidence、source、publication_date、methodology 等字段，让下游代理有足够上下文正确使用每条数据。
- Modifying upstream agents to return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when downstream agents have limited context budgets
  - 在多智能体流水线中，上游代理的冗长推理过程会快速消耗下游代理有限的上下文预算。上游应返回精炼的结构化输出：关键结论、证据引用、字段化元数据、置信度和缺口说明；推理过程可以记入日志，但不应作为主输出传给下游。这是多智能体上下文预算管理的关键设计原则。
- 对长文档分块时使用重叠分块（Overlapping Chunks），防止信息在分块边界处丢失
  - 当文档过长无法放入上下文时，按顺序切成无重叠的块会产生边界问题：跨块的句子和段落会失去周围语境，可能被误解或遗漏。重叠分块的做法是在每个块的末尾和下一个块的开头保留一段共同内容（通常为块大小的 10–20%），确保模型能正确理解边界附近的内容。考试中如果描述"分块边界附近的信息被遗漏或误解"，重叠分块是正确的解决方向，而不是单纯增大分块或减少重叠。
- 对大型文档集合使用分层摘要（Hierarchical Summarization）实现高压缩比同时保留结构
  - 对大量文档做单次直接摘要会丢失太多细节。分层摘要分两个阶段：先对每个章节或文档独立摘要（保留关键 claim、证据和来源引用），再对章节摘要合并成最终高层综述。这种方式比单次压缩保留更多结构，同时让每个结论仍可追溯到具体来源。考试中问到如何高效摘要大型文档集合时，分层摘要优于单次压缩。

### Task Statement 5.2: Design effective escalation and ambiguity resolution patterns

#### Knowledge of:

- Appropriate escalation triggers: customer requests for a human, policy exceptions/gaps (not just complex cases), and inability to make meaningful progress
  - 以"问题复杂度"作为升级触发条件存在两个问题：复杂度没有客观定义，而且用户体验上很可能在复杂问题上等待更长时间之后才升级。实际升级条件应为可程序化判断的规则：用户发出明确的人工请求、当前请求超出政策覆盖范围、系统经过多步尝试仍无法推进。这些条件明确、可测试，也可以在 few-shot 示例中展示。
- The distinction between escalating immediately when a customer explicitly demands it versus offering to resolve when the issue is straightforward
  - 这两种情况的区别在于用户的明确表达：使用"我要跟真人说话""请转人工客服"等语句是显式请求，必须立即升级，无论问题多简单都不能继续尝试解决。"我很失望""这太麻烦了"是情绪表达，不等于人工请求，此时可先承认情绪并提供解决方案；如果用户在此后再次明确要求人工，则立即升级。
- Why sentiment-based escalation and self-reported confidence scores are unreliable proxies for actual case complexity
  - 情绪分析有误判——相同的表达在不同文化和语境中情绪解读不同，而且负面情绪不等于需要人工干预。模型自评置信度（"我对这个答案有 80% 把握"）未经校准，不能作为安全阈值依据。可靠的升级机制应该是规则驱动：检测特定触发词（"人工""真人"）、检测政策缺口、检测流程停滞（同一问题尝试 N 次无进展）。
- How multiple customer matches require clarification (requesting additional identifiers) rather than heuristic selection
  - 当工具返回多个可能匹配的用户或订单时，选择其中一个是错误的，无论选择依据是哪个更近、哪个金额更接近还是哪个名字更常见。错误选择会导致向错误用户暴露信息（隐私违规）或操作错误账户。唯一安全的处理方式是向用户明确请求额外标识符（如邮箱后缀、订单号、账单地址邮编），从而确认身份后再继续。

#### Skills in:

- Adding explicit escalation criteria with few-shot examples to the system prompt demonstrating when to escalate versus resolve autonomously
  - 即使系统 prompt 中有文字规则，模型在处理边界案例时仍可能做出不一致的判断。在 system prompt 中加入 few-shot 示例，展示"用户要求人工 → 立即升级，不再说服"和"用户表达不满但未要求人工 → 承认情绪并提供方案"的对比，能锚定模型在真实对话中的行为边界。Few-shot 在升级规则上的作用与在代码审查分类中的作用相同：补充文字规则无法表达的判断边界。
- Honoring explicit customer requests for human agents immediately without first attempting investigation
  - 在收到明确人工请求后继续调用工具或尝试解决问题，是对用户意图的忽视，会进一步激怒用户并降低对系统的信任。正确行为是立即停止工具调用序列，生成升级摘要（包含已收集的 case facts 和 issue 状态），并交接给人工渠道。系统 prompt 应明确这个停止条件，并通过 few-shot 展示触发时的正确响应模式。
- Acknowledging frustration while offering resolution when the issue is within the agent's capability, escalating only if the customer reiterates their preference
  - 只基于情绪分数升级会损失不必要的自动解决机会。当用户表达沮丧但问题在系统能力范围内时，正确做法是先承认情绪，然后提供解决方案，避免重复要求用户解释已说明的情况。只有当用户在此后再次明确提出人工请求时，才升级。这个两步模式把用户满意度和自动化效率都考虑在内。
- Escalating when policy is ambiguous or silent on the customer's specific request (e.g., competitor price matching when policy only addresses own-site adjustments)
  - 当客户的请求落在系统已有政策的字面范围之外时（例如政策只说"本站价格调整"但客户要求竞争对手价格匹配），不应该让模型推断这个扩展是否"合理"然后自行处理。让模型自行解读政策边界会造成不一致的客户体验，也可能违反业务规则。正确做法是检测政策缺口并升级，由有权限的人员做出例外判断。
- Instructing the agent to ask for additional identifiers when tool results return multiple matches, rather than selecting based on heuristics
  - 在客户支持场景中，以错误的身份执行操作（如退款到错误账户、查看他人订单详情）可能违反隐私法规，也会导致严重的客户纠纷。歧义消解的安全原则是"宁可多问一次，不可猜一次"。请求的额外标识符应该是用户能立即提供的信息（如邮箱、账单邮编、订单号后四位），而不是要求用户输入整个账户密码或其他高敏感信息。

### Task Statement 5.3: Implement error propagation strategies across multi-agent systems

#### Knowledge of:

- Structured error context (failure type, attempted query, partial results, alternative approaches) as enabling intelligent coordinator recovery decisions
  - 结构化错误信息是多智能体系统的决策输入，而不只是日志。`failure_type` 区分超时、权限拒绝、解析错误和真正空结果；`attempted` 记录已执行的查询，避免协调者重复同样失败的操作；`partial_results` 让协调者判断是否可以带不完整数据继续；`alternatives` 给出替代查询词或来源，让协调者有具体方案可路由。这四个字段是最小可行的结构化错误格式。
- The distinction between access failures (timeouts needing retry decisions) and valid empty results (successful queries with no matches)
  - 访问失败（如超时、权限错误、服务不可达）表示查询没有成功执行，结果未知；空结果表示查询成功执行，但数据库中没有匹配项。把空结果表示为失败，协调者会触发不必要的重试，浪费时间和资源。把访问失败表示为空结果，协调者会误以为主题无数据，在合成报告中产生错误的缺口说明。两者应在状态字段中明确区分（`success: true, count: 0` vs `status: "timeout"`）。
- Why generic error statuses ("search unavailable") hide valuable context from the coordinator
  - 返回 `"operation failed"` 或 `"search unavailable"` 这样的通用错误，等于把所有失败类型折叠成一个无法区分的信号。协调者不知道这是超时（可以重试）、权限问题（重试无效，需要换工具），还是解析失败（需要改 prompt）。没有具体失败类型，协调者只能选择放弃，无法做出智能恢复决策。结构化错误的成本很低，但对系统可靠性的提升很大。
- Why silently suppressing errors (returning empty results as success) or terminating entire workflows on single failures are both anti-patterns
  - "吞错"（silently returning empty results as success）会让协调者以为数据不存在，导致合成报告中出现错误的缺口或漏报。"单点失败全局终止"会丢弃其他子智能体已经完成的工作，强迫重头执行。结构化传播的正确模式是：每个子智能体报告自己的成功、部分成功或失败状态，协调者综合这些状态决定如何推进——重试哪个、继续哪个、标记哪个缺口。

#### Skills in:

- Returning structured error context including failure type, what was attempted, partial results, and potential alternatives to enable coordinator recovery
  - 诊断单类比很具体：医生的诊断单会记录症状、已检查项目、检查结果和建议的下一步，而不是只写"病了"。子智能体的失败报告同样应包含：做了什么（attempted_queries）、拿到了什么（partial_results）、为什么失败（failure_type: timeout/access_denied/parse_error）、可不可以重试（recoverability）、还能怎么做（alternatives）。这些字段让协调者能做出具体决策，而不是只能放弃。
- Distinguishing access failures from valid empty results in error reporting so the coordinator can make appropriate decisions
  - 建议的返回结构是两类：成功时返回 `{"status": "success", "count": 0, "results": []}` 表示查询成功但无匹配；失败时返回 `{"status": "failed", "failure_type": "timeout", "recoverability": "retryable"}` 表示执行本身出了问题。这两种格式让协调者在代码层面就能区分，不需要解析文字描述。`recoverability` 字段直接告诉协调者这次失败是否值得重试。
- Having subagents implement local recovery for transient failures and only propagate errors they cannot resolve, including what was attempted and partial results
  - 子智能体应该有自己的本地错误处理层：网络超时可以在本地尝试 1-2 次重试，解析失败可以尝试不同解析策略，再上报给协调者之前先尽力自我恢复。只有在本地无法恢复时，才把结构化失败报告传给协调者。这种分层减少了协调者需要处理的失败信号数量，也降低了因瞬时网络抖动触发不必要的全局重试。
- Structuring synthesis output with coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources
  - 合成报告不应该假装所有来源都可用且质量相同。当部分子智能体失败或返回覆盖不足时，对应的结论应该标注 `coverage: "partial"` 或附注"来源 X 不可用，此结论仅基于来源 Y 和 Z"。这让读者能够判断哪些结论有充分支撑，哪些需要进一步验证。把所有结论以相同置信度呈现，是不诚实的多源综合。

### Task Statement 5.4: Manage context effectively in large codebase exploration

#### Knowledge of:

- Context degradation in extended sessions: models start giving inconsistent answers and referencing "typical patterns" rather than specific classes discovered earlier
  - 上下文退化不是突然出现的，而是逐渐显现：模型开始用"这类项目通常会..."替代之前发现的具体文件路径；开始说"一般的 auth 模块会..."而不是引用 `src/auth/jwt_validator.py` 第 45 行。这种转变说明早期发现已被当前上下文压缩或丢失。检测到这个信号时，应注入 scratchpad 内容或阶段性摘要，补回丢失的具体发现。
- The role of scratchpad files for persisting key findings across context boundaries
  - Scratchpad 文件（如 `.claude/scratchpad/findings.json`）是持久化到文件系统的外部记忆，不受单次会话上下文窗口限制。模型无法"记住"第 N 轮会话前发现的文件，但可以从 scratchpad 读取。关键发现、入口点、依赖关系、待办事项和风险点应在发现时立即写入 scratchpad，后续任务优先读取它，而不是依赖模型的上下文记忆。
- Subagent delegation for isolating verbose exploration output while the main agent coordinates high-level understanding
  - 主智能体如果直接承担大量文件的逐一阅读，上下文会快速充满细节，主智能体自身的判断能力会因注意力稀释而下降。正确分工是：主智能体维护架构地图（入口点、核心模块、依赖关系）和任务清单，子智能体负责回答具体问题（"找出所有调用支付接口的文件"）并返回结构化摘要，主智能体基于摘要合并全局理解。这样主智能体始终保持在高层，不被细节淹没。
- Structured state persistence for crash recovery: each agent exports state to a known location, and the coordinator loads a manifest on resume
  - Manifest 是长期任务的可恢复性基础。它记录已完成的模块（`completed_modules`）、待处理的模块（`pending_modules`）、关键发现文件路径（`findings_file`）和当前假设。任务崩溃或中断后，协调者加载 manifest，注入状态到新 agent prompt，从待处理模块继续，而不是从头重新扫描整个代码库。没有 manifest 的长任务在崩溃后只能重做，浪费大量时间。

#### Skills in:

- Spawning subagents to investigate specific questions (e.g., "find all test files," "trace refund flow dependencies") while the main agent preserves high-level coordination
  - 子智能体任务描述越模糊，它探索的范围越广，返回的内容越冗长，对主智能体的价值越低。"分析代码库"这样的指令会让子智能体读大量文件后返回笼统总结；"找出所有调用 `process_refund()` 的文件和调用方式"才是可执行的具体任务。明确边界还能防止子智能体在探索深度上过度投入，确保每个子任务在合理 token 预算内完成。
- Having agents maintain scratchpad files recording key findings, referencing them for subsequent questions to counteract context degradation
  - 这条原则的实际操作是：每当子智能体发现重要事实（如发现 auth 模块在第 45 行缺少 JWT 过期检查），立即以结构化格式 append 到 scratchpad 文件，而不是只在当前会话的 text 输出中提到它。后续的另一个子智能体或恢复后的主智能体，通过读取 scratchpad 文件获取已知事实，而不是搜索聊天历史。外部持久化的发现不受会话上下文窗口限制。
- Summarizing key findings from one exploration phase before spawning sub-agents for the next phase, injecting summaries into initial context
  - 阶段间上下文传递的原则是"压缩但保真"：不传完整的探索输出（太长），也不传模糊的自然语言总结（丢失关键细节），而是传递结构化摘要：已完成模块、关键发现（文件、函数、行号）、发现的问题、未解决问题、下一阶段建议。这个摘要作为下一阶段子智能体的初始上下文，让它能在已知基础上继续，而不是从头探索。
- Designing crash recovery using structured agent state exports (manifests) that the coordinator loads on resume and injects into agent prompts
  - 在代码中实现恢复逻辑时，协调者应首先检查是否存在 manifest 文件，如果存在则加载 `completed_modules`、`pending_modules` 和 `findings_file` 路径，注入状态后只对 `pending_modules` 调用子智能体，并把 `findings_file` 的内容作为已知上下文传入。如果没有检查 manifest 就全量重新探索，会重复已完成的工作，并在有新发现时可能得出与之前矛盾的结论。
- Using /compact to reduce context usage during extended exploration sessions when context fills with verbose discovery output
  - `/compact` 是 Claude Code 中的上下文压缩命令，用于在上下文接近满载时压缩已有内容，保留关键结论和状态，丢弃冗余的探索过程和中间步骤输出。在大型代码库探索中，子智能体的详细输出往往包含大量中间步骤，这些信息在确认结论后就不再需要。主动使用 `/compact` 能防止上下文窗口过早耗尽，使长时间探索会话得以继续。

### Task Statement 5.5: Design human review workflows and confidence calibration

#### Knowledge of:

- The risk that aggregate accuracy metrics (e.g., 97% overall) may mask poor performance on specific document types or fields
  - 提取系统在标准文档上 99% 准确、在手写收据上 68% 准确，整体准确率如果是 97% 仍然看似优秀，但手写收据的错误率足以造成实际业务问题。分层评估要求按文档类型（标准发票、手写收据、扫描件、多语言）和字段（总金额、日期、税号、行项目）分别计算准确率，找出表现不达标的细分场景，而不是用总分掩盖它们。
- Stratified random sampling for measuring error rates in high-confidence extractions and detecting novel error patterns
  - 持续抽样的目的不只是统计已知错误的比例，也是为了及时发现系统上线后出现的新错误模式。文档格式变化、新供应商模板、OCR 质量变化或 prompt drift 都可能导致原本高置信的字段开始出错。分层随机抽样（按文档类型、字段、置信档位分别抽取样本）能确保不会因为样本偏差而错过特定细分场景的退化。
- Field-level confidence scores calibrated using labeled validation sets for routing review attention
  - 校准的具体操作是：在带有人工标注正确答案的验证集上，统计模型输出 high/medium/low 置信的字段中，真实准确率分别是多少。如果 high confidence 对应的真实准确率是 92%，那么用这个档位自动通过时，期望错误率是 8%，是否可接受取决于业务风险。未经校准的置信度无法设定合理阈值，因为不同模型、不同任务、不同字段上的置信输出范围各不相同。
- The importance of validating accuracy by document type and field segment before automating high-confidence extractions
  - 总体高置信准确率达到阈值，不能作为全面开启自动化的依据。需要逐一确认每个文档类型和关键字段的置信-准确率校准结果都满足设定阈值。只有在每个细分维度都验证通过后，才安全地对那个维度的高置信结果开启自动化。未充分验证的细分场景应继续保留人工审核，避免盲目扩大自动化范围。

#### Skills in:

- Implementing stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection
  - 系统上线后，文档分布会随时间变化，新的格式、新的供应商、新的数据质量问题都会出现。如果在自动化上线后停止抽样，漂移可能悄悄积累，直到错误率明显升高时才被发现。持续抽查的频率可以低于初始校准阶段，但不能为零。抽样中发现的新错误模式应作为新的标注样本回流到校准集，保持校准数据与当前分布一致。
- Analyzing accuracy by document type and field to verify consistent performance across all segments before reducing human review
  - 矩阵分析的维度是文档类型 × 字段，例如 [发票, 税号]、[收据, 总金额]、[合同, 有效期] 各自的准确率。这个矩阵能清楚显示哪些维度已经达到自动化要求、哪些仍然需要人工。只看总体分数可能导致在一些关键字段（如高风险的金额、日期）还未达标时就整体减少审核。每次调整人工审核比例，都应以这个矩阵的最新数据为依据。
- Having models output field-level confidence scores, then calibrating review thresholds using labeled validation sets
  - 整体置信度（"这份文档的提取置信度为 0.88"）无法告诉你具体哪个字段可靠。字段级置信度（`invoice_number: high, total_amount: medium, tax_id: low`）能精确路由到合适的审核策略：总金额 medium 置信且金额大的进人工审核，发票号 high 置信且该类型已校准通过的自动通过。阈值的设定需要用真实标注数据估计各档位的实际准确率，而不是直接使用模型输出的置信分值。
- Routing extractions with low model confidence or ambiguous/contradictory source documents to human review, prioritizing limited reviewer capacity
  - 人工审核队列应该按优先级排序，而不是 FIFO。优先级最高的是：模型置信度低的字段、多个来源之间有数值冲突的情况、高风险操作相关字段（如大额金额、合同有效期、权限控制）、以及系统识别出有歧义或格式异常的记录。将有限的审核员时间集中在最高风险和最不确定的记录上，能在相同人工成本下最大化错误发现率。

### Task Statement 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis

#### Knowledge of:

- How source attribution is lost during summarization steps when findings are compressed without preserving claim-source mappings
  - 摘要过程中来源丢失是多源系统的典型失效模式：子智能体将多篇文档总结为一段话，但总结中不包含"这个数字来自哪篇文档的哪段"。合成代理接收这个摘要后无法知道来源，最终报告中的结论无法核查。结构化摘要应始终保留 claim-to-source 映射，即使需要压缩文字，也要保留来源 URL、文档名、相关摘录和发布时间。
- The importance of structured claim-source mappings that the synthesis agent must preserve and merge when combining findings
  - 合成代理的正确操作不是用自己的语言重新表述已有结论，而是把来自多个子智能体的 claim-source 映射合并和去重。如果两个子智能体报告了同一个 claim 但来自不同来源，合成结果应该包含两个来源，而不是只写结论一次。重新表述无出处结论会产生"流畅但无法验证"的报告，在学术、法律或商业场景中可靠性极低。
- How to handle conflicting statistics from credible sources: annotating conflicts with source attribution rather than arbitrarily selecting one value
  - 可信来源之间的数值差异可能有多种解释：测量时间不同、样本范围不同、测量方法不同、变量定义不同。在没有充分信息判断哪个更准确时，报告应该并列两个数值、各自的来源、时间和方法，并说明可能的解释。让模型自主选择"看起来更权威"的数值，会把方法论判断的责任错误地留给模型。
- Temporal data: requiring publication/collection dates in structured outputs to prevent temporal differences from being misinterpreted as contradictions
  - 同一指标在不同时间点测量可能得到不同结果，这是正常的时序变化，不是来源冲突。但如果两个来源都没有时间戳，协调者无法判断差异是时序造成的还是方法不同造成的。在结构化输出中强制要求 `publication_date` 和 `data_collection_date` 字段，能让合成代理正确标注数据的时效性，也能在比较多个来源时优先使用更近期的数据或明确标注时序差异。

#### Skills in:

- Requiring subagents to output structured claim-source mappings (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis
  - 这是系统设计约束，不只是建议：在子智能体的输出 schema 中，每个 claim 对象必须包含 `claim`、`source_url`、`document_name`、`excerpt`、`publication_date` 字段，并且协调者和合成代理的 prompt 中必须明确"不得省略来源字段"。这个约束要在 JSON Schema 层面（required 字段）和 prompt 层面（few-shot 展示完整映射结构）双重强化。
- Structuring reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context
  - "AI 提高效率"可能是多项研究的共同发现（已确立），也可能只有一项内部研究有数据（需标注局限性），或者不同研究得到相反结论（有争议）。报告应明确区分这三类情况，而不是用同样的语气呈现所有结论。"有争议"还要附上争议的来源和方法差异，让读者理解争议的性质，而不是只知道"说法不一"。
- Completing document analysis with conflicting values included and explicitly annotated, letting the coordinator decide how to reconcile before passing to synthesis
  - 分析代理处理单个文档或数据源，没有全局视角，不能判断哪个来源更可信或哪个方法更权威。当它发现文档内部有冲突数值（如两处税额不一致）或与已知数据有差异时，应把两个值都报告给协调者，附上各自的位置、上下文和可能的解释。由协调者或最终合成代理在掌握全局信息的情况下做出裁决，或把冲突透明呈现给读者。
- Requiring subagents to include publication or data collection dates in structured outputs to enable correct temporal interpretation
  - 时间戳包括两种：发布日期（文档何时发布）和数据采集日期（研究数据何时收集）。两者可能相差数月甚至数年，都是重要的元数据。在子智能体的输出 schema 中，这两个字段应设为 required（或明确允许 null 并标注原因），不能只靠文档文字描述来传递时效信息。下游代理在比较多个来源时，第一步往往是对齐时间维度，没有时间戳就无法做这一步。
- Rendering different content types appropriately in synthesis outputs—financial data as tables, news as prose, technical findings as structured lists—rather than converting everything to a uniform format
  - 财务数据用表格展示（方便对齐和比较），新闻事件或叙述性内容用散文（方便理解时间顺序和因果关系），技术发现用结构化列表（方便查阅和引用），有争议的结论用并列区块（明确标注支持方、反对方和不确定方）。把所有内容强制转成同一种格式（如全文散文或全部列表），会降低特定内容类型的可读性，有时也会掩盖数据结构中的重要关系。

---

## Task 5.1：长交互中的上下文管理

### 先看问题在哪里

1. **"Lost in the Middle"效应**：模型对长输入的开头和结尾更稳，中间部分容易遗漏
2. **工具结果累积**：每次工具调用的结果不断积累，消耗大量 token
3. **渐进摘要风险**：摘要会有丢失细节的风险，尤其是数值、日期、用户明确要求

### Case Facts（案件事实）模式

将关键事实抽取为持久化结构块，防止在长对话中被遗忘：

```python
case_facts = {
    "customer_id": "CUST-12345",
    "account_status": "active",
    "order_id": "ORD-67890",
    "order_amount": 299.99,
    "order_date": "2024-01-15",
    "issue_type": "missing_item",
    "previous_contacts": 2
}
# 每轮对话开始时注入，确保关键信息不丢失
```

### 工具结果精简

```python
# 保留完整工具结果（40+ 字段全部进入上下文）
full_result = get_customer(customer_id)

# 只保留相关字段
relevant_fields = {
    "customer_id": full_result["id"],
    "name": full_result["name"],
    "status": full_result["account_status"],
    "refund_eligible": full_result["refund_eligible"]
    # 忽略：地址、历史交易、营销偏好等无关字段
}
```

### 上下文位置策略

- **重要信息前置**：放在 system prompt 或消息开头，而非中间
- **结构化分节**：用明确的标题分隔内容，减轻位置效应
- **独立上下文层**：对多问题会话，将结构化 issue 数据单独保存

### 这题怎么考

这一题考的不是“如何把对话变短”，而是如何在压缩上下文时不丢掉会影响决策的事实。考试场景通常会出现长客服会话、多个订单、多个工具结果或多个子智能体输出，正确答案会偏向结构化事实层、字段裁剪、关键发现前置和完整历史传递，而不是简单让模型“总结一下”。

实战中要把上下文分成三层：

1. **持久事实层**：订单号、金额、日期、状态、用户明确期望、政策判断结果。
2. **工作摘要层**：当前问题进展、已尝试步骤、下一步计划。
3. **原始证据层**：必要时可回查的工具返回、引用、子智能体元数据。

这样设计的核心价值是让模型在长交互后仍然基于同一组关键事实行动，而不是基于被摘要稀释后的模糊印象行动。

---

## Task 5.2：升级与歧义消解

### 升级触发条件

| 触发条件             | 示例                         |
| -------------------- | ---------------------------- |
| 用户明确要求人工     | "我要跟真人说话"             |
| 政策存在空白或例外   | 不在标准策略范围内的情况     |
| 系统无法继续取得进展 | 多次尝试均无效               |
| 多个模糊匹配无法消解 | 找到多个可能的客户，无法确认 |

### 不可靠的升级触发

| 方法                        | 问题               |
| --------------------------- | ------------------ |
| 情绪分析（负面情绪 > 阈值） | 情绪不代表复杂度   |
| 模型自报置信分（1-10分）    | 模型自我评估不可靠 |
| 任务"感觉复杂"              | 主观判断，无法量化 |

### 这条规则别忘

> **用户明确要求人工时，应立即升级，不再尝试继续解决。**

```
用户首次表达不满 → 承认情绪，尝试解决
用户再次要求人工 → 立即升级（不再争论）
```

### 歧义消解

当工具返回多个匹配结果时，**请求额外标识符**，而不是启发式猜测：

```python
# 错误：猜测最可能的匹配
if len(matches) > 1:
    return matches[0]  # 猜测第一个

# 正确：请求额外信息消解歧义
if len(matches) > 1:
    return {
        "status": "ambiguous",
        "matches": matches,
        "request": "请提供更多标识信息（如邮箱或订单号）以确认身份"
    }
```

### 这题怎么考

这一题常见干扰项是“先继续调查”“根据情绪分数升级”或“选择最可能匹配的客户”。官方要求更强调可审计的升级规则：用户明确要求人工时立即升级；政策没有覆盖时升级；无法取得有意义进展时升级；工具返回多个候选时先消歧。

实战中升级逻辑应该写成明确的决策树，而不是交给模型自由判断：

1. **显式人工请求**：直接转人工，不再说服用户继续与机器人沟通。
2. **能力范围内的问题**：先承认情绪并尝试解决，除非用户再次坚持人工。
3. **政策空白或例外**：不要扩展政策含义，交给人工或更高权限流程。
4. **身份或对象歧义**：请求额外标识符，不能用排序、相似度或“最像”来猜。

可靠系统的目标不是减少升级次数，而是把升级发生在正确边界上。

---

## Task 5.3：多智能体系统中的错误传播

### 结构化错误上下文

子智能体失败时，应返回**足够信息让协调者做出恢复决策**：

```json
{
  "status": "failed",
  "failure_type": "search_timeout",
  "attempted_queries": ["AI in music industry 2024", "music technology trends"],
  "partial_results": [
    { "title": "AI Music Generation Report", "url": "...", "snippet": "..." }
  ],
  "alternatives": [
    "可尝试搜索 'music technology AI impact'",
    "可查询学术数据库获取更全面数据"
  ],
  "coverage_gaps": ["音乐流媒体行业数据缺失"]
}
```

### 协调者的恢复决策

基于结构化错误信息，协调者可以：

- **重试**（瞬时错误）
- **修改查询**（语义错误，有替代方案）
- **带部分结果继续**（覆盖不完整但可接受）
- **升级到人工**（无法恢复）

### 错误传播反模式

| 反模式                               | 问题                       |
| ------------------------------------ | -------------------------- |
| 返回通用错误（`"operation failed"`） | 协调者无法做出恢复决策     |
| 悄悄吞掉错误，返回空结果             | 协调者误以为成功           |
| 单点失败终止整个流程                 | 损失所有已完成工作         |
| 将空结果伪装成错误                   | 协调者误判，触发不必要重试 |

### 这题怎么考

这一题考多智能体系统中“失败信息是否足够恢复”。子智能体不能只返回 `failed` 或 `search unavailable`，也不能把失败吞掉后返回空数组。协调者需要知道失败类型、尝试过什么、拿到了哪些部分结果、还有哪些替代路径，才能决定重试、换查询、换数据源、继续合成或标注覆盖缺口。

实战中建议把子智能体结果统一成可合成的 envelope：

```json
{
  "status": "success | partial | failed",
  "failure_type": "timeout | access_denied | no_results | parse_error | unknown",
  "attempted": ["query or operation"],
  "partial_results": [],
  "recoverability": "retryable | needs_alternative_source | unrecoverable",
  "coverage_gaps": [],
  "next_options": []
}
```

最容易考的区别是 **valid empty results** 与 **access failure**：前者表示查询成功但没有匹配，后者表示没有获得可靠结果。两者会导向完全不同的协调决策。

---

## Task 5.4：大型代码库探索中的上下文管理

### 上下文退化的症状

> 模型开始说"通常模式是..."，却忘了前面发现的具体类和结构

### Scratchpad 模式

跨上下文边界持久化关键发现：

```python
# 智能体维护结构化 scratchpad
scratchpad = {
    "architecture": {
        "entry_points": ["src/main.py", "src/api/router.py"],
        "key_modules": ["auth", "payment", "notification"]
    },
    "findings": [
        {"file": "auth.py", "issue": "JWT 验证缺少过期检查", "line": 45},
    ],
    "dependencies": {
        "auth -> user_service": "直接调用",
        "payment -> notification": "事件驱动"
    }
}
```

### 阶段总结策略

```
探索阶段：
→ 子智能体探索具体模块
→ 返回结构化发现摘要（而非完整探索输出）
          ↓
阶段切换时：
→ 总结本阶段关键发现
→ 将摘要注入下一阶段子智能体的上下文
          ↓
下一阶段：
→ 基于结构化摘要继续工作
→ 不需要重新探索已知内容
```

### `/compact` 命令

在长时间探索任务中，主动使用 `/compact` 压缩上下文，防止上下文窗口耗尽。

### 崩溃恢复设计

```python
# 每个子智能体输出状态到固定位置
agent_state = {
    "manifest_version": "1.0",
    "completed_modules": ["auth", "payment"],
    "pending_modules": ["notification", "reporting"],
    "findings_file": ".claude/scratchpad/findings.json"
}
# 恢复时，协调者加载 manifest 继续工作
```

### 这题怎么考

这一题把上下文管理放到大型代码库探索场景中考。关键不是让一个模型读完整个仓库，而是把探索任务拆成可验证的小问题，并把发现写到上下文之外。考试选项中如果出现“让主 agent 一直保留所有探索输出”，通常不是最佳答案；更好的答案会涉及 subagent、scratchpad、阶段总结、manifest 和 `/compact`。

实战中大型代码库探索应遵循这个顺序：

1. **先建立地图**：入口文件、核心模块、测试位置、关键依赖。
2. **再派发问题**：每个 subagent 只查一个具体问题，例如调用链、测试覆盖或配置来源。
3. **沉淀发现**：把文件路径、类名、函数名、行号、结论写入 scratchpad。
4. **阶段性压缩**：将冗余探索日志压缩成结论和证据，必要时使用 `/compact`。
5. **可恢复执行**：manifest 记录已完成、待完成、证据文件和下一步，避免崩溃后重做。

判断上下文退化的信号是模型开始引用“常见架构模式”，却无法指出本仓库中已经发现的具体文件和符号。

---

## Task 5.5：人工审核工作流与置信度校准

### 总体准确率的陷阱

> `97% overall accuracy` 可能隐藏某类关键字段只有 70% 的准确率

**正确做法：按文档类型和字段维度分层分析准确率**

| 文档类型 | 字段   | 准确率             |
| -------- | ------ | ------------------ |
| 标准发票 | 总金额 | 99%                |
| 手写收据 | 总金额 | 68% ← 需要人工审核 |
| 所有类型 | 日期   | 95%                |
| 所有类型 | 税额   | 71% ← 需要人工审核 |

### 置信度校准流程

```
1. 模型输出字段级置信度
        ↓
2. 用带标签的验证集校准阈值
（不要直接使用未经校准的置信分）
        ↓
3. 设定分流阈值：
   - 高置信（>0.95）→ 自动通过
   - 中置信（0.7-0.95）→ 抽样审核
   - 低置信（<0.7）→ 全量人工审核
        ↓
4. 持续监控：分层随机抽样验证高置信样本的真实错误率
```

### 减少人工审核前的验证步骤

**不能只看整体准确率**，需要：

1. 按文档类型分析准确率
2. 按字段维度分析准确率
3. 在高置信样本中持续抽样验证

### 这题怎么考

这一题考的是“置信度如何变成可靠的人审分流机制”。模型输出的高置信度本身不能直接作为自动通过依据，必须用带标签验证集校准，并且要按文档类型和字段分层观察真实错误率。总体准确率很高并不代表所有字段都安全。

实战中可以把人审流程设计成四类队列：

1. **强制人审**：低置信、来源冲突、字段缺失、高金额或高风险操作。
2. **抽样人审**：高置信结果也要分层随机抽样，用来监控漂移和新错误模式。
3. **自动通过**：只有在该文档类型和该字段都经过验证后，才进入自动化路径。
4. **规则复核**：对特定字段设置硬规则，例如金额范围、日期格式、税额关系。

考试里遇到“97% overall accuracy”这类说法，要立刻想到分层验证：文档类型、字段、置信区间和持续抽样，而不是直接减少人工审核。

---

## Task 5.6：多源综合中的来源链保留

### 来源链保留

综合子智能体必须保留 **claim-source mapping**：

```json
{
  "claim": "AI使音乐创作效率提升35%",
  "confidence": "high",
  "sources": [
    {
      "url": "https://music-tech-report.com/2024",
      "document_name": "Music_AI_Report_2024.pdf",
      "excerpt": "平均创作时间从8小时减少到5.2小时...",
      "publication_date": "2024-03-15"
    }
  ]
}
```

### 处理冲突数据

当可信来源给出不同数据时：

```
错误做法：随机选一个，或选看起来更大的
正确做法：同时保留两个值，注明来源和日期

示例：
{
  "statistic": "AI辅助创作效率提升",
  "values": [
    {"value": "35%", "source": "MIT研究2024", "date": "2024-01"},
    {"value": "28%", "source": "Stanford调查2023", "date": "2023-11"}
  ],
  "note": "数值差异可能反映研究方法和时间差异，非矛盾"
}
```

### 时间戳的重要性

不同时间的数据差异可能是**时序变化**而非矛盾——**发布时间/采集时间**是判断的关键证据。

### 综合报告的展示原则

- **已充分证实的结论** vs **存在争议的结论** → 明确区分
- 财务数据 → 表格
- 事件叙述 → 散文
- 技术发现 → 结构化列表

### 这题怎么考

这一题考多源综合时如何保持可追溯性和不确定性。正确答案不会把多个来源压成一个没有出处的流畅总结，而是保留 claim-source mapping：每个结论对应来源、摘录、文档名、URL、发布时间或采集时间。来源之间冲突时，系统应该显式呈现冲突，而不是让模型任意选择一个数值。

实战中合成报告至少应包含：

1. **结论层**：这个 claim 是什么，属于已证实、部分支持还是存在争议。
2. **证据层**：支持 claim 的来源、摘录、时间和方法背景。
3. **冲突层**：不同可信来源给出的不同数值或解释，以及可能原因。
4. **格式层**：财务数据用表格，新闻事件用叙述，技术发现用结构化列表。

这类题的判断标准是“读者能否追溯每个结论从哪里来”。如果摘要读起来顺畅但无法定位来源，就是不可靠的多源综合。

---

## 讲透这一域：上下文管理、可靠性与人机协作边界

Domain 5 是很多考生低估的部分。它权重只有 15%，但它连接了前面所有 domain：智能体循环会产生工具结果，上下文会膨胀；多智能体会传递摘要，来源可能丢失；结构化提取会给出置信度，需要人工审核；客户支持会遇到歧义和升级；大型代码库探索会让模型遗忘早期发现。Domain 5 的核心不是“上下文窗口多大”，而是如何在有限上下文中保留关键事实、传播错误、处理不确定性，并设计可靠的人类介入机制。

### 一、上下文不是越多越好，而是越相关越好

长对话中最危险的不是 token 不够，而是关键信息被低价值内容淹没。工具结果往往很长，一个订单查询可能返回 40 多个字段，但退款判断只需要订单号、金额、日期、状态、退货资格。把完整结果全部塞进上下文，会让模型在后续轮次更难找到真正重要的信息。正确做法是在工具结果进入对话前裁剪，只保留与当前任务有关的字段。

progressive summarization 也有风险。摘要可能把“客户要求 2024-03-15 前退款 129.99 美元”压缩成“客户要求退款”，导致日期、金额、承诺丢失。因此数值、日期、订单号、客户明确期望不能只放自然语言摘要，而应进入持久的 case facts block。每轮都注入结构化事实，能避免长对话后模型忘记关键约束。

“Lost in the Middle”说明长输入的中间部分更容易被遗漏。解决方法不是简单缩短，而是重新组织：关键发现放开头，详细证据分节，重要限制在结尾再次提醒。多源综合时，先给 executive summary 和 claim-source map，再给详细材料。这样模型在注意力上更容易抓住任务主线。

### 二、升级策略要基于规则，不基于情绪猜测

客户支持场景里，升级不是“用户生气就升级”。情绪分析不是复杂度的可靠代理，模型自报置信也不可靠。真正的升级条件包括：用户明确要求人工、政策有空白或例外、系统无法取得有意义进展、多个身份或订单匹配无法消歧、高风险操作需要人工审批。规则越明确，系统越稳定。

用户明确要求人工时，应立即升级，不要继续劝说或调查。用户只是表达不满但没有要求人工，且问题在自动能力范围内，可以先承认情绪并尝试解决；如果用户再次要求人工，再升级。这个边界很重要，因为过早升级会降低自动解决率，过晚升级会激怒用户。考试题常把“用户不满”和“用户要求人工”混在一起，正确答案要区分。

歧义消解也不能靠启发式猜测。工具返回多个 customer matches 或 order matches 时，不应选第一个、最近一个或金额最大的一个，而应请求额外标识符，例如邮箱、订单号、邮编、手机号后四位。错误选择可能导致隐私泄漏或错误操作。多匹配是请求澄清的信号，不是模型发挥常识的机会。

### 三、多智能体错误传播要保留恢复线索

多智能体系统中，子智能体失败很正常。搜索服务超时、文档加载失败、权限不足、来源不可用、查询无结果，都可能发生。关键是失败如何上报。通用 `"search unavailable"` 隐藏了太多信息；协调者不知道查询了什么、重试了几次、有没有部分结果、是否有替代关键词。结构化错误应包含 failure_type、attempted_query、attempts、partial_results、alternatives、coverage_gaps。

有效空结果也必须和访问失败分开。搜索成功但没有匹配，说明可能需要换查询词或记录覆盖缺口；搜索失败，说明需要重试或换工具。把失败伪装为空结果，会让合成代理以为某个主题没有资料；把空结果伪装成失败，会浪费重试资源。Domain 5 与 Domain 2 在这里重叠，但 Domain 5 更关注错误如何跨 agent 传播到协调者和最终综合。

协调者不应该因为一个子智能体失败就终止整个流程。它可以带部分结果继续，重新委托缺口，标注覆盖不足，或者升级人工。最终报告应说明哪些发现证据充分，哪些区域因为来源不可用存在缺口。这样用户能理解不确定性，而不是收到一个看似完整但实际缺证据的报告。

### 四、大型代码库探索需要外部记忆

长时间探索代码库时，模型会逐渐上下文退化。典型症状是它开始说“通常这种架构会……”而不是引用前面发现的具体文件、类和函数。解决方法是 scratchpad、阶段总结、子智能体隔离和 `/compact`。scratchpad 保存关键发现、入口点、依赖关系、待办、风险；阶段总结把每轮探索结果压缩成结构化状态；子智能体负责局部问题，主智能体只保留高层地图。

探索任务应按问题拆分，而不是让一个 agent “读完整个仓库”。例如让子智能体分别找所有测试文件、追踪退款流程依赖、分析认证中间件、列出数据库迁移。每个子智能体返回结构化摘要，主智能体合并成架构地图。这样既减少上下文噪音，也方便恢复。

崩溃恢复要设计 manifest。每个长任务应记录已完成模块、待处理模块、发现文件、当前假设、未解决问题。恢复时协调者加载 manifest，并把状态注入新 agent。没有 manifest 的恢复只是“希望模型记得”，不可靠。`/compact` 适合上下文被大量探索输出填满时使用，它压缩过程细节，保留关键结论。

### 五、人工审核与置信度校准

总体准确率很容易误导。一个提取系统 97% overall accuracy，可能在标准发票上 99%，在手写收据金额字段上只有 68%。如果只看总分就减少人工审核，会把低表现细分场景放进自动化。正确做法是按文档类型、字段、来源质量、版式复杂度分层评估。

模型可以输出字段级置信度，但这个置信度必须校准。校准需要带标签验证集：看模型说 high 的字段真实准确率是多少，medium 和 low 分别是多少。然后设置分流策略：高置信自动通过但持续抽样，中置信抽样审核，低置信或冲突来源全量人工审核。未经校准的“confidence: high”不能直接当自动通过依据。

高置信样本也需要持续抽样。系统上线后文档分布会变化，新的表格格式、新供应商模板、新语言、新扫描质量都会带来错误。分层随机抽样能发现新错误模式，防止自动化静悄悄退化。人工审核不是一次性门槛，而是持续监控机制。

### 六、来源链是多源综合的生命线

多源综合最常见失败是摘要过程中丢失来源。子智能体把多个文档压缩成“AI 提高效率 35%”，但没有保留来源 URL、文档名、页码、发布时间、摘录。后续合成再引用这个结论，就无法追溯。正确做法是所有子智能体输出 claim-source mapping，每个 claim 绑定来源和证据，合成代理必须保留并合并这些映射。

可信来源之间冲突时，不要让模型随便选一个。两个统计数可能因为时间、方法、样本、定义不同而差异。报告应并列呈现数值、来源、时间和方法背景，并说明冲突是否真实矛盾。比如 2023 调查说 28%，2024 实验说 35%，这可能是时间变化或方法差异，不一定冲突。没有时间戳的数据很难解释，因此 publication date 和 data collection date 是结构化输出的重要字段。

不同内容类型也应选择不同展示方式。财务数据适合表格，新闻事件适合叙述，技术发现适合结构化列表，争议点适合“支持/反对/不确定”分区。把所有内容强行转成同一种格式，会降低可读性和准确性。Domain 5 的多源综合不仅要求保留来源，还要求把不确定性以合适格式呈现给人类。

### 七、Domain 5 的答题框架

遇到上下文和可靠性题，可以按问题类型判断。长对话遗忘事实，使用 case facts 和结构化 issue layer；工具结果太长，裁剪相关字段；长文档遗漏中间信息，关键摘要前置并分节；用户要求人工，立即升级；政策空白，升级；多匹配，要求额外标识符；子智能体失败，结构化错误传播；大型代码库探索退化，scratchpad、子智能体、manifest、`/compact`；人工审核减少，先做分层准确率和置信度校准；多源综合冲突，保留 claim-source mapping、时间戳和冲突注释。

错误答案通常有这些特征：用情绪分数决定升级；让模型自报置信决定是否人工；把所有工具结果原样塞上下文；把失败返回空列表；单个子任务失败就终止全局；摘要时丢掉来源；冲突统计只选一个；总体准确率高就取消人工审核。正确答案会保留事实、来源、错误和不确定性，让系统在长流程中仍可追溯、可恢复、可审计。

### 八、把上下文拆成多层，而不是一条聊天记录

生产系统不应该只维护一条不断增长的 messages 列表。更可靠的做法是把上下文分层：原始对话层保存完整交互，case facts 层保存关键事实，issue state 层保存每个问题的状态，tool summary 层保存裁剪后的工具结果，source map 层保存来源关系，handoff layer 保存给人工的摘要。每一层服务不同目的，而不是把所有内容混成一段自然语言。

这种分层能解决很多长对话问题。用户问订单退款又问账户登录，两个 issue 的事实不应混在一起；工具返回大量字段，只把当前 issue 需要的字段放进 issue state；人工升级时，不需要完整工具输出，只需要 handoff summary；多源报告时，不需要每次都重传全文，但必须保留 claim-source map。Domain 5 的“上下文管理”本质就是把信息放在正确层级。

### 九、可靠性不是模型单独负责

很多错误选项会把可靠性归因于模型：“让 Claude 更注意”“让 Claude 自评置信”“让 Claude 判断是否升级”。官方思路更工程化：可靠性来自系统设计。上下文裁剪由代码做，关键事实持久化由状态层做，错误分类由工具做，升级规则由 policy 做，人工审核阈值由验证集校准，来源保留由结构化 schema 做。模型参与判断，但不应独自承担所有保证。

例如客户支持 agent 可以理解用户意图，但身份匹配不能靠猜；文档提取模型可以给字段置信度，但是否自动通过要靠校准；研究 agent 可以合成多源结论，但来源冲突要保留给协调者或读者判断。Domain 5 的最佳答案通常会把模型输出与外部机制结合，而不是完全相信模型。

### 十、长期会话中的“新鲜度”问题

上下文不仅会变长，还会变旧。代码库探索中，文件可能已经被修改；客户支持中，订单状态可能更新；研究系统中，新的来源可能出现；CI 中，PR 可能新增 commit。旧上下文如果不标记时间和版本，会让模型基于过期事实推理。因此关键工具结果最好带 timestamp、source version、file commit hash 或 retrieved_at。恢复会话时，要告诉模型哪些事实需要重新验证。

这个点和 Domain 1 的 session resume 呼应。恢复旧会话不是自动正确，旧上下文必须经过新鲜度判断。上下文管理不仅是压缩，还包括失效策略：哪些事实可以长期保留，哪些事实需要每次刷新，哪些事实只能作为历史参考。考试题如果出现“恢复后基于旧文件分析出错”，正确方向是注入结构化摘要并重新分析变更文件，而不是盲目 resume。

### 十一、人工审核工作流要考虑人类负载

Human-in-the-loop 不是把所有不确定结果都扔给人工。人工审核能力有限，系统要把最值得看的内容排在前面。低置信字段、冲突来源、高金额、高风险客户操作、政策空白、陌生文档类型，优先级应高；高置信且经过校准的常见字段可以自动通过或抽样。审核界面也应展示原文摘录、字段位置、模型值、冲突说明，而不是只给最终 JSON。

人工反馈还要回流。审核员纠正字段后，系统应记录错误类型，是 OCR 问题、字段定义问题、prompt 问题、schema 问题还是来源缺失。只有这样才能改进模型流程。否则 human review 只是成本中心，不会提高系统质量。Domain 5 与 Domain 4 的反馈闭环在这里交汇：人工审核结果可以成为后续校准集和 few-shot 样本。

### 十二、答题时关注“可追溯”

Domain 5 的核心标准可以简化成三个字：可追溯。事实能否追溯到用户原话或工具结果？错误能否追溯到失败类型和已尝试动作？结论能否追溯到来源和时间？人工交接能否追溯到已验证步骤？置信度能否追溯到校准数据？如果答案让信息在摘要、合成或升级中丢失，就不可靠。

可追溯并不意味着保留所有原文，而是保留决策所需的证据链。保留全部原文会浪费上下文；只保留结论会丢证据。正确做法是结构化压缩：事实、来源、时间、方法、冲突、置信、状态。考试中看到“summary”这个词要警惕：好的 summary 会保留 metadata，坏的 summary 只留下流畅文字。

### 十三、与其他 Domain 的连接

Domain 5 是收尾能力。Domain 1 的多智能体需要上下文传递和错误传播；Domain 2 的工具需要裁剪结果和结构化错误；Domain 3 的大型代码库探索需要 scratchpad、manifest、compact；Domain 4 的结构化提取需要置信度校准和人工审核。复习时不要把 Domain 5 当独立章节，而要把它理解为所有智能体系统长期运行后的可靠性问题。

如果一个题目看起来同时涉及多源报告、子智能体失败和人工审核，优先从信息是否丢失入手。谁知道什么？这些信息传给了谁？来源还在吗？错误还在吗？人类接手时够不够？这样比背单个术语更稳。

### 十四、用三个场景串起 Domain 5

第一个场景是客户支持。用户先问退款，又补充账单争议，工具返回多个订单，用户后来要求人工。正确系统会维护 case facts，按 issue 分开状态，请求额外标识符消歧，用户明确要求人工时立即升级，并交接结构化摘要。错误系统会把所有对话揉成摘要、猜测订单、基于情绪分数升级或不升级。

第二个场景是多智能体研究。搜索代理超时但拿到部分来源，文档代理发现两个来源统计冲突，合成代理需要写报告。正确系统会让搜索代理上报 partial results 和 coverage gaps，让文档代理保留冲突值、来源、时间和方法，让合成代理区分已证实和有争议结论。错误系统会吞掉错误、只写一个流畅总结、丢掉来源映射。

第三个场景是大型代码库探索。主 agent 想理解退款流程，子 agent 分别追踪 API、数据库、测试和事件流。正确系统会让每个子 agent 输出结构化发现，写入 scratchpad，阶段性 compact，manifest 记录已完成和待办。错误系统会让一个 agent 连续读大量文件，最后开始说“通常模式”，忘掉具体类和路径。

这三个场景覆盖了 Domain 5 的大部分考点：关键事实持久化、歧义消解、升级、错误传播、来源保留、上下文退化、人工审核。备考时不要把它们当零散知识点，而要理解它们都在解决同一个问题：长流程中信息会丢失、变旧、变模糊，系统必须主动保真。

### 十五、最小可行可靠性清单

设计任何 Claude 生产系统时，至少要问十个问题。哪些事实必须永久保留？哪些工具结果可以裁剪？哪些错误可重试，哪些必须升级？用户要求人工时是否立即执行？多匹配时是否请求澄清？子智能体是否保留来源和时间？长任务是否有 scratchpad？恢复任务是否有 manifest？高置信自动化是否经过校准？最终报告能否追溯每个 claim？

如果这些问题没有答案，系统可能在 demo 中表现很好，但在真实长对话、多工具、多来源、多轮修改中失效。Domain 5 的价值就在于把 demo 变成可靠系统。它要求你承认模型会遗忘、会受上下文位置影响、会低估不确定性，也要求你用结构化状态、外部记忆、人工审核和来源链来补足这些限制。

最后复习时，把 Domain 5 视为“信息保真工程”。保真不是保存全部内容，而是保存会影响决策的内容：金额、日期、身份、状态、来源、时间、错误、冲突和人工判断。只要系统能在长流程结束时回答“这个结论从哪里来、现在还可靠吗、失败时谁接手”，它就具备了可靠性的基础。

也可以把 Domain 5 看成系统的“记忆与交接”能力。记忆负责在长对话和多智能体之间保留事实，交接负责在人类、协调者、子智能体和下游系统之间传递状态。记忆不可靠，模型会忘；交接不可靠，人工会重做；来源不可靠，报告不能信；置信度不可靠，自动化会误放行。可靠系统必须同时解决这四个问题，并把每一次关键判断都留下可审计的依据和清晰证据链。

我会问一句很土但有用的话：三天后回来查这件事，还能不能说清楚当时为什么这么判断？

---

## 临考速查

1. **"Lost in the Middle"** → 重要信息放开头或结尾，不放中间
2. **Case Facts** → 将关键事实持久化，每轮注入，防止长对话中丢失
3. **工具结果精简** → 只保留当前任务相关字段进入上下文
4. **立即升级** → 用户明确要求人工时，不再尝试继续解决
5. **升级信号** → 政策空白 > 情绪分析；多匹配 → 请求额外标识符
6. **错误传播** → 结构化错误 + 部分结果 + 替代方案，而非 `"operation failed"`
7. **空结果 ≠ 失败** → `success: true, results: []` 是有效响应
8. **上下文退化** → Scratchpad + 阶段总结 + `/compact` 命令
9. **总体准确率陷阱** → 必须按文档类型和字段分层分析
10. **冲突数据** → 保留两个值 + 来源 + 时间戳，不要武断选一个
