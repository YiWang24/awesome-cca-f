# Domain 4: Prompt Engineering & Structured Output（提示工程与结构化输出）

> **权重：20%**  
> 官方文档：[Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) | [Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

---

## Task Statement 覆盖范围

| Task | 主题                                      |
| ---- | ----------------------------------------- |
| 4.1  | 用显式标准提升精度并减少误报              |
| 4.2  | 用 few-shot prompting 提升输出一致性      |
| 4.3  | 用 tool use 与 JSON Schema 强制结构化输出 |
| 4.4  | 为提取质量设计验证、重试与反馈闭环        |
| 4.5  | 设计高效的批处理策略                      |
| 4.6  | 设计多实例与多轮次评审架构                |

---

## Domain 4 学习主线

Domain 4 考的不是“会写提示词”这一件事，而是能否把 Claude 的输出变成可控、可验证、可规模化运行的系统组件。考试会反复区分三类能力：第一，如何用显式标准、few-shot 和结构化 schema 提高输出稳定性；第二，如何用验证、重试、反馈闭环处理模型仍然会犯的语义错误；第三，如何在批处理、多实例、多轮审查架构中根据 SLA、成本、上下文限制和误报风险做工程取舍。

备考时要把每个 Task 都理解成一个设计题：看到“误报高”，优先想到显式 report/skip 标准、禁用高误报类别、严重级别示例；看到“格式不一致”，优先想到 few-shot，而不是继续堆长指令；看到“必须结构化”，优先想到 tool use + JSON Schema，而不是要求模型“输出合法 JSON”；看到“提取结果有错”，要判断是可重试的格式/结构/语义错误，还是源文档缺失导致重试无效；看到“大量离线任务”，要考虑 Batch API 的成本和 24 小时处理窗口；看到“大型代码审查”，要拆成独立实例和多 pass，避免自审偏差和注意力稀释。

---

### Task Statement 4.1: Design prompts with explicit criteria to improve precision and reduce false positives

#### Knowledge of:

- The importance of explicit criteria over vague instructions (e.g., "flag comments only when claimed behavior contradicts actual code behavior" vs "check that comments are accurate")
  - 可判定标准的核心是模型能根据代码内容直接做出 true/false 决策，而非依赖主观语气词。抽象指令（如"更谨慎"）不改变决策边界，只会让模型困惑；显式 report/skip 列表能让相同输入得到相同输出。标准越具体，误报率越低，输出越一致。
- How general instructions like "be conservative" or "only report high-confidence findings" fail to improve precision compared to specific categorical criteria
  - "高置信"对模型来说没有操作定义，因为模型的置信自报未经校准，不等于实际准确率。按类别设定证据门槛（如"只有当注释描述的行为与代码逻辑可验证矛盾时才报告"）才能真正约束输出范围。模糊语气词适合对话，不适合生产审查系统。
- The impact of false positive rates on developer trust: high false positive categories undermine confidence in accurate categories
  - 信任是系统级属性，不是类别级属性。开发者只要被噪音刷够几次，就会停止认真阅读所有输出，包括真正的 critical finding。降低单个高误报类别的误报率，对整体系统可信度的提升效果，往往超过单纯提高其他类别的准确率。

#### Skills in:

- Writing specific review criteria that define which issues to report (bugs, security) versus skip (minor style, local patterns) rather than relying on confidence-based filtering
  - 没有明确跳过列表，模型会把代码风格、命名偏好、TODO 注释都当成潜在问题上报。在 prompt 里明确"跳过：单纯的风格改进建议、团队内部约定的命名方式"，能有效减少低价值 finding。report/skip 列表要在 prompt 中和 few-shot 里双重强化。
- Temporarily disabling high false-positive categories to restore developer trust while improving prompts for those categories
  - 暂时关闭高误报类别是一种信任恢复策略，而非放弃该类别。持续上报低质量 finding 会导致开发者完全忽略系统，比暂时不报危害更大。在关闭期间，应收集该类别的正负样本、改进显式标准、增加 few-shot，待误报率达到可接受水平后再启用。
- Defining explicit severity criteria with concrete code examples for each severity level to achieve consistent classification
  - 严重级别定义缺少代码示例时，模型会根据语言表述做模糊推断，导致同一安全漏洞今天判 High、明天判 Critical。每个级别至少需要一个正例代码片段，以及一个"看起来像但不到这个级别"的边界示例。示例锚定判断边界，文字标准只提供框架。

### Task Statement 4.2: Apply few-shot prompting to improve output consistency and quality

#### Knowledge of:

- Few-shot examples as the most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results
  - 详细说明能表达规则，但无法展示边界。模型在处理模糊案例时会对规则做不同解读，导致相同输入产生不同输出。Few-shot 通过具体示例锁定"规则在真实数据上如何应用"，特别适用于两个答案都看似合理但只有一个符合业务需求的场景；此时堆砌更多文字说明通常无效，示例才是有效干预。
- The role of few-shot examples in demonstrating ambiguous-case handling (e.g., tool selection for ambiguous requests, branch-level test coverage gaps)
  - 边界案例是误判高发区，也是 few-shot 最有价值的地方。比如 `config.get("key", default)` 和 `config["key"]` 在不同上下文下分别是安全访问和 KeyError 风险，文字难以清晰区分，但代码示例能让模型精确识别差异。每个 few-shot 示例最好包含取舍理由，解释为什么选 A 而不是 B。
- How few-shot examples enable the model to generalize judgment to novel patterns rather than matching only pre-specified cases
  - 低质量 few-shot 只展示具体案例，模型学到的是"见过这个才能答"；高质量 few-shot 展示判断依据，模型能把原则泛化到新情况。在示例中写明"因为 X，所以报告/跳过"，让模型学到规则，而不只是记住样本。好的泛化能力意味着模型遇到训练示例中没见过的新模式时也能正确判断。
- The effectiveness of few-shot examples for reducing hallucination in extraction tasks (e.g., handling informal measurements, varied document structures)
  - 提取任务的幻觉风险来自 schema 中 required 字段与文档中实际缺失信息之间的张力。Few-shot 示例可以展示"源文档没有字段 X 时应返回 null，而非猜测"，明确处理缺失信息的策略。示例还应覆盖非标准版式（如总额在页眉而非表格底部），让模型知道从哪些位置寻找字段。

#### Skills in:

- Creating 2-4 targeted few-shot examples for ambiguous scenarios that show reasoning for why one action was chosen over plausible alternatives
  - 2-4 个高质量示例通常优于 20 个泛泛示例，因为示例过多会稀释注意力、增加上下文开销，甚至让模型陷入模板匹配而非判断推理。每个示例应针对一个已知的误判模式或边界场景，并在示例中解释"为什么选这个而不是那个"。覆盖率优先于数量。
- Including few-shot examples that demonstrate specific desired output format (location, issue, severity, suggested fix) to achieve consistency
  - 在 prompt 里描述输出格式很容易产生歧义，比如"包含 location 字段"可能被理解成多种形态。在 few-shot 示例中直接展示包含所有期望字段的完整输出，能消除格式歧义，让模型学到精确的字段名称、嵌套层级和值类型。字段稳定性对下游系统解析至关重要。
- Providing few-shot examples distinguishing acceptable code patterns from genuine issues to reduce false positives while enabling generalization
  - 只给正例（"应该报告的情况"）会导致模型过度激活，把相关但不需要报告的模式也上报。在 few-shot 中加入"看起来像但应该跳过"的反例，明确表示"这种情况不报告、理由是 X"，能有效收窄报告范围。正反对照比单独写"不要报告 Y"的文字规则更有效，因为模型能从示例中直接感知边界。
- Using few-shot examples to demonstrate correct handling of varied document structures (inline citations vs bibliographies, methodology sections vs embedded details)
  - 同一类信息在不同文档中可能以截然不同的格式出现：引文可以是 inline [1]，也可以是 bibliography；方法论可以在独立章节，也可以嵌在正文脚注。如果 few-shot 只覆盖一种格式，模型在遇到其他版式时会提取失败或产生 null。示例集应系统覆盖文档的主要结构变体。
- Adding few-shot examples showing correct extraction from documents with varied formats to address empty/null extraction of required fields
  - 字段返回 null 不一定是因为文档里没有信息，也可能是因为模型没在正确位置找。当特定版式下的字段提取失败时，要先诊断是真正缺失还是格式匹配失败，再决定是加示例还是允许 null。针对已知高失败率版式添加对应示例，是提升字段召回率的直接手段。

### Task Statement 4.3: Enforce structured output using tool use and JSON schemas

#### Knowledge of:

- Tool use (tool_use) with JSON schemas as the most reliable approach for guaranteed schema-compliant structured output, eliminating JSON syntax errors
  - 在 prompt 里要求"输出 JSON"只是提示性约束，模型可能在 JSON 前后加说明文字、输出 Markdown 代码块、或产生语法错误。Tool use 配合 `input_schema` 是 API 层面的强制约束：模型生成的 tool call input 必须符合 schema，天然消除 JSON 语法错误。生产环境需要稳定字段时，应优先选择 tool use 而非 prompt-only JSON 提取。
- The distinction between tool_choice: "auto" (model may return text instead of calling a tool), "any" (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)
  - `tool_choice: "auto"` 允许模型自主决定是否调用工具，遇到它认为可以直接文本回答的输入时可能绕过 tool call，导致结构化输出缺失。`tool_choice: "any"` 强制模型从可用工具中选一个调用，适合文档类型未知但必须结构化输出的场景。Forced tool selection（`{"type": "tool", "name": "X"}`）在必须按特定顺序执行多步骤时使用，例如先提取元数据再增强。
- That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)
  - JSON Schema 验证的是字段存在性、类型和枚举范围，不验证字段之间的业务关系。发票行项目金额相加不等于 total_amount、日期字段格式正确但来自错误文本位置、类别枚举合法但语义选错，这些都不会被 schema 捕获。结构化输出后必须叠加业务校验层，检查计算一致性和语义正确性。
- Schema design considerations: required vs optional fields, enum fields with "other" + detail string patterns for extensible categories
  - 将字段设为 required 会给模型压力，当源文档没有对应信息时，模型可能为了满足 schema 而生成看起来合理的假值。设为 nullable（`["string", "null"]`）允许模型诚实地返回 null。Enum 没有 `"other"` 选项时，模型可能把无法归类的值硬塞进最接近的 enum，而不是标记为未知。

#### Skills in:

- Defining extraction tools with JSON schemas as input parameters and extracting structured data from the tool_use response
  - 使用 tool use 的目的是让模型以结构化方式返回数据，而不是在自然语言回复中生成 JSON。结构化结果位于响应 `content` 数组中 type 为 `tool_use` 的 block 的 `input` 字段，类型已经是 Python dict，不需要再做 JSON 解析。若代码仍在尝试解析 `text` 类型 block 的 JSON，说明对 tool use 响应格式理解有误。
- Setting tool_choice: "any" to guarantee structured output when multiple extraction schemas exist and the document type is unknown
  - 当系统需要处理多种文档类型（如发票、合同、收据）但事先不知道具体类型时，可以定义多个提取工具，并将 `tool_choice` 设为 `"any"`，让模型根据文档内容选择最合适的工具调用。这比用单一 universal schema 更精确，因为不同文档类型有不同的字段集合，用最匹配的 schema 能减少 null 字段和编造风险。
- Forcing a specific tool with tool_choice: {"type": "tool", "name": "extract_metadata"} to ensure a particular extraction runs before enrichment steps
  - 在多步骤提取流程中，某些步骤有严格的执行顺序依赖，例如必须先调用 `extract_metadata` 获取文档类型，才能据此决定后续增强策略。Forced tool selection（`{"type": "tool", "name": "extract_metadata"}`）保证这一步一定执行，避免模型自行判断"可以跳过"或选择其他工具。这是实现有序多步骤工作流的可靠手段。
- Designing schema fields as optional (nullable) when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields
  - Schema 中字段被标记为 required 时，模型会尝试填充它——即使源文档完全没有对应信息，也会生成一个看起来合理的假值。将可能缺失的字段设为可选并允许 null（`"type": ["string", "null"]`，不放入 `required` 数组），给模型一个合法的"无法确定"出口，能有效减少幻觉。
- Adding enum values like "unclear" for ambiguous cases and "other" + detail fields for extensible categorization
  - Enum 字段如果没有 `"unclear"` 或 `"other"` 选项，当模型遇到无法确定归类的情况时，会从现有枚举中硬选一个最近似的值，导致分类错误。加入 `"unclear"` 允许模型表达不确定性，`"other"` 加 detail 字段允许模型记录具体原因。下游系统可针对这两类值路由到人工审核，而不是接受错误的强制分类。
- Including format normalization rules in prompts alongside strict output schemas to handle inconsistent source formatting
  - JSON Schema 可以约束字段类型为 string，但无法约束日期是 ISO 8601（`YYYY-MM-DD`）还是 `MM/DD/YYYY`。在 prompt 中明确"日期统一使用 ISO 8601 格式，金额统一为不含货币符号的数字，单位统一使用国际单位制"，能避免字段合法但格式不统一导致的下游解析失败。格式规范和 schema 需要并列使用，缺一不可。

### Task Statement 4.4: Implement validation, retry, and feedback loops for extraction quality

#### Knowledge of:

- Retry-with-error-feedback: appending specific validation errors to the prompt on retry to guide the model toward correction
  - 不带错误信息的重试等于用相同输入再次调用，模型没有新信息，大概率产出相同错误。有效重试要将校验失败的具体描述（如"line_items 合计 128.50 但 total_amount 为 132.50，差值 4.00"）附加到原始文档和失败结果后，让模型知道错在哪里、改什么。错误越具体，模型自我修正的成功率越高。
- The limits of retry: retries are ineffective when the required information is simply absent from the source document (vs format or structural errors)
  - 重试的前提是错误来自模型的输出问题（格式错误、字段位置错、计算不一致），而不是源文档本身的信息缺失。当验证失败原因是"文档中根本没有这个字段"时，给模型再多次提示都无法生成真实信息，只会增加幻觉概率。系统需要明确停止条件：字段缺失时标记为 null 并停止重试，信息在外部文档中时升级人工处理。
- Feedback loop design: tracking which code constructs trigger findings (detected_pattern field) to enable systematic analysis of dismissal patterns
  - 在每条 finding 中加入 `detected_pattern` 字段（如触发规则的代码片段或模式描述），能把每次误报转化为可分析数据。统计哪些 pattern 被开发者反复忽略，可以识别系统性问题类别——这些类别要么标准定义过宽，要么 few-shot 示例不足。没有结构化模式字段，误报分析只能靠人工评论，无法系统改进。
- The difference between semantic validation errors (values don't sum, wrong field placement) and schema syntax errors (eliminated by tool use)
  - Tool use 配合 JSON Schema 能在 API 层面消除 JSON 语法错误（花括号不匹配、字段类型错误），但不能检测语义错误。语义错误是指结构上合法但业务含义错误的输出，例如行项目相加与总额不符、日期字段来自标题而非正文、分类枚举选了语义错误的项。语义校验需要程序逻辑：计算交叉验证、正则格式检查、业务规则断言。

#### Skills in:

- Implementing follow-up requests that include the original document, the failed extraction, and specific validation errors for model self-correction
  - 重试 prompt 需要包含三个要素：原始文档（让模型重新参考来源）、失败的提取结果（让模型知道之前输出了什么）、具体校验错误（让模型知道改哪里）。缺少任何一个都会降低修正成功率——没有原文模型无法重新提取，没有失败结果模型不知道改什么，没有具体错误模型可能改错地方。
- Identifying when retries will be ineffective (e.g., information exists only in an external document not provided) versus when they will succeed (format mismatches, structural output errors)
  - 区分"可重试错误"和"不可重试错误"是反馈闭环设计的关键。格式错误（日期格式不符、字段嵌套层级错、枚举值大小写不对）是可修正的，重试配合具体错误反馈通常能修复。信息缺失（源文档没有该字段、信息在另一个文档中但未提供）是不可修正的，重试只会产生编造值，应直接返回 null 或标记为 unclear 并停止。
- Adding detected_pattern fields to structured findings to enable analysis of false positive patterns when developers dismiss findings
  - `detected_pattern` 字段记录的是触发该 finding 的具体代码结构或文本模式，例如"未检查键存在直接访问 dict"或"参数不含类型注解"。当开发者 dismiss 某条 finding 时，系统能把 dismiss 动作和具体触发模式关联起来，统计哪类模式的 finding 被持续忽略。这是把人工反馈转化为 prompt 改进数据的最小可行闭环。
- Designing self-correction validation flows: extracting "calculated_total" alongside "stated_total" to flag discrepancies, adding "conflict_detected" booleans for inconsistent source data
  - 在 schema 中同时要求 `calculated_total`（行项目相加结果）和 `stated_total`（文档中声明的总额），加上 `conflict_detected` 布尔字段，让模型自己做交叉计算并标记不一致，而非只提取一个总额字段。这把原本需要外部校验的逻辑内嵌到提取步骤，一次 API 调用就能同时获得数据和数据质量指标。下游系统可据此决定是重试还是升级人工。

### Task Statement 4.5: Design efficient batch processing strategies

#### Knowledge of:

- The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA
  - Message Batches API 用异步处理换取约 50% 的成本折扣；代价是最长 24 小时处理窗口，且没有低延迟 SLA。它适合离线、非阻塞、对延迟不敏感的任务，例如夜间文档处理、历史数据标注和批量测试生成；不适合用户正在等待结果、合并前检查、前置条件校验，或需要中途工具调用的 agentic loop。
- Batch processing is appropriate for non-blocking, latency-tolerant workloads (overnight reports, weekly audits, nightly test generation) and inappropriate for blocking workflows (pre-merge checks)
  - 决定是否使用 Batch API 的核心判据是延迟容忍度。PR 合并前检查需要开发者等待结果，属于阻塞流程，必须使用同步 API。夜间代码质量报告、每周成本审计、月度文档分类这类任务，结果稍后交付即可，适合用 Batch API 降低大规模处理成本。这是“SLA 优先，成本次之”的工程决策原则。
- The batch API does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)
  - Message Batches API 的每个请求是独立的单轮对话：发出 prompt，得到一个完整响应。如果任务需要模型调用工具、等待工具返回结果、然后继续推理（agentic loop），Batch API 无法支持这种模式，因为 batch 内的请求不能在处理过程中交换信息。涉及多轮工具调用的工作流必须使用同步 API，逐步进行。
- custom_id fields for correlating batch request/response pairs
  - Batch 请求的响应不保证顺序，且批次内部分请求可能失败而其他成功。`custom_id` 是用户自定义的标识符，用于将输出中的每条结果精确匹配回对应的输入文档。没有 `custom_id`，失败后只能全量重提交；有了 `custom_id`，可以筛选出失败项、分析失败原因、按原因修改后只重提交这部分文档，节省成本和时间。

#### Skills in:

- Matching API approach to workflow latency requirements: synchronous API for blocking pre-merge checks, batch API for overnight/weekly analysis
  - 成本优化不能牺牲关键路径延迟。只要调用阻塞用户、开发者合并或下游自动化流程，就应使用同步 API；只有结果可以稍后交付的离线任务才考虑 Batch API。判断顺序应是：先确认 SLA 和阻塞关系，再判断 24 小时 batch 窗口是否可接受，最后再比较成本。
- Calculating batch submission frequency based on SLA constraints (e.g., 4-hour windows to guarantee 30-hour SLA with 24-hour batch processing)
  - Batch API 的上限是“提交后最多 24 小时完成”，因此总延迟还要加上任务等待下一次提交窗口的时间。若每 4 小时提交一批，最坏端到端延迟约为 4 + 24 = 28 小时。设计批处理频率时要倒推业务 SLA，并为失败重跑预留余量。
- Handling batch failures: resubmitting only failed documents (identified by custom_id) with appropriate modifications (e.g., chunking documents that exceeded context limits)
  - Batch 请求失败的原因可能各不相同：有些是文档超过 context window（需要 chunk）、有些是 prompt 格式问题（需要调整 prompt）、有些是模型返回错误结构（需要改 schema 或 few-shot）。用 `custom_id` 筛选出失败项后，要按失败原因分组处理，不能一律用同样的修改重提交所有失败文档。精准处理失败项是 batch 系统可靠性的关键。
- Using prompt refinement on a sample set before batch-processing large volumes to maximize first-pass success rates and reduce iterative resubmission costs
  - 直接在完整数据集上跑 batch 而不先验证 prompt，一旦 prompt 有缺陷，整个批次都会失败，且失败后才能得到反馈，浪费最多 24 小时处理时间和相应成本。标准做法是从数据集中抽取 20-50 个有代表性的样本，用同步 API 快速迭代 prompt，直到抽样通过率达到预期，再提交完整 batch。抽样调试的成本远低于批量返工。

### Task Statement 4.6: Design multi-instance and multi-pass review architectures

#### Knowledge of:

- Self-review limitations: a model retains reasoning context from generation, making it less likely to question its own decisions in the same session
  - 模型在生成代码或内容时会形成内部推理路径，这些路径会保留在同一会话的上下文中。当被要求审查自己刚生成的内容时，模型倾向于按照生成时的推理验证输出，而不是独立重新评估。这种确认偏差（confirmation bias）导致生成者容易忽略自己最初判断中的错误，即使加入 extended thinking 也难以完全消除。
- Independent review instances (without prior reasoning context) are more effective at catching subtle issues than self-review instructions or extended thinking
  - 独立的 Claude 实例不携带原始生成的推理上下文，在收到待审查内容时会进行独立的一次评估，类似于让另一名工程师 review 同一段代码。这种隔离让审查实例更容易注意到生成实例因先入为主而忽略的问题，例如隐含的前置条件假设、接口契约违反或边界处理缺失。独立审查的质量通常高于同一模型的自审指令。
- Multi-pass review: splitting large reviews into per-file local analysis passes plus cross-file integration passes to avoid attention dilution and contradictory findings
  - 把数十个文件一次性输入进行代码审查，会导致注意力分散（attention dilution），模型可能在分析后续文件时遗忘早期文件的细节，产生矛盾或不完整的 finding。局部 pass 每次只处理一个文件，专注于本地 bug、异常处理、边界条件；集成 pass 汇总局部结果后专注于跨文件的数据流、接口契约和状态一致性，两类 pass 分别使用独立的调用。

#### Skills in:

- Using a second independent Claude instance to review generated code without the generator's reasoning context
  - 实现独立审查时，审查实例只应接收待审查的代码和审查任务描述，不应接收生成实例的推理过程、设计说明或意图解释。生成过程的推理会隐性引导审查者认为设计是合理的，从而减少批评性发现。干净的上下文隔离是独立审查有效性的前提，也是与"要求同一会话再想想"的本质区别。
- Splitting large multi-file reviews into focused per-file passes for local issues plus separate integration passes for cross-file data flow analysis
  - 局部 pass 的审查范围严格限定在单个文件内：变量的正确性、异常处理的完整性、边界条件、本地逻辑错误。集成 pass 的输入是各局部 pass 的结构化摘要，关注跨模块维度：A 模块传出的数据类型是否符合 B 模块的期望、状态是否在多模块间一致传播、接口变更是否所有调用方都已同步更新。两类 pass 目标不同，混合会互相干扰。
- Running verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing
  - 模型自报的置信度（high/medium/low）未经校准前不等于实际准确率。"high confidence"可能对应 70% 真实准确率，也可能对应 95%，取决于模型、任务和领域。用带标签的验证集估计各置信档位的实际准确率后，才能设定安全的分流阈值：超过某准确率的可自动处理，低于阈值的进入人工复核。置信度未经校准就用来关闭人工审核，风险极高。

---

## Task 4.1：显式标准与减少误报

### 这题怎么考

这道题的核心是“精度来自可执行标准，不来自形容词”。考试如果给出一个代码审查、内容审核或分类场景，并说明输出误报太多，正确方向通常不是把 prompt 改成“更谨慎”“只输出高置信结果”，而是把问题类别拆清楚：哪些必须报告、哪些必须跳过、每个严重级别对应什么证据。实战中，误报会直接损害开发者信任，所以高误报类别可以先临时关闭，再用更明确的标准和示例恢复。

### 核心原则：模糊指令 → 不稳定结果

```python
# 模糊指令（结果不可预测）
system = "请更加谨慎，只报高置信度的问题"

# 显式分类标准（结果可预测）
system = """
报告以下类别的问题（必须报告）：
- 硬编码密钥、密码、API Token
- SQL注入、命令注入漏洞
- 未处理的 KeyError/IndexError 风险

跳过以下类别（不报告）：
- 性能优化建议（除非明显 O(n²) 或更差）
- 代码风格问题（缩进、命名偏好）
- TODO 注释
"""
```

### 误报率的实际影响

> 高误报率不只损伤该类别的信任度，还会**连带损伤所有类别的信任度**，导致开发者忽略所有输出，包括真正的严重问题。

### 减少误报的策略

1. **明确列出报告/跳过的类别**（而非模糊的"置信度过滤"）
2. **暂时关闭高误报类别**：先恢复用户对输出的信任，再逐步优化该类别的提示
3. **为每个严重级别定义明确标准**，并附上正面/负面代码示例
4. **区分"确定是问题"和"可能是问题"**：分开报告，不要混在一起

### 提示结构技巧

**XML 标签**用于清晰分隔提示的不同部分：

```xml
<system>
你是代码安全审查工具。

<report_categories>
必须报告的类别：
- 硬编码凭证
- 注入漏洞
- 权限绕过
</report_categories>

<skip_categories>
跳过的类别：
- 风格问题
- 性能建议
- TODO 注释
</skip_categories>

<output_format>
以 JSON 数组格式输出，每项包含 severity, category, description, line
</output_format>
</system>
```

**Prefill（预填充）**：在 assistant 轮次的开头预填充内容，引导特定输出格式：

````python
messages = [
    {"role": "user", "content": "分析以下代码：\n```python\n...\n```"},
    {"role": "assistant", "content": "["}  # 预填充 JSON 数组起始符
]
# Claude 会继续填充数组内容，而非先输出解释文字
````

---

## Task 4.2：Few-Shot Prompting

### 这题怎么考

Few-shot 的考点是“用示例教判断边界”。当详细说明仍然产出不一致、格式不稳定、边界案例误判或提取任务幻觉时，few-shot 往往比继续增加规则更有效。高质量示例要展示为什么在两个合理选项之间选择其中一个，也要展示“应该报”和“不应该报”的差异，这样模型学到的是判断原则，而不是机械匹配某几个固定样本。

### 何时使用

| 情况                             | 推荐策略      | 原因               |
| -------------------------------- | ------------- | ------------------ |
| 仅靠详细说明仍输出不一致         | Few-shot 示例 | 示例比说明更精确   |
| 模糊边界场景（可接受 vs 真问题） | 对照示例      | 展示边界两侧的例子 |
| 新模式的类比泛化                 | Few-shot      | 让模型理解模式意图 |
| 格式有具体要求                   | Few-shot      | 直接展示期望格式   |

### 有效 Few-Shot 的设计原则

1. **数量不是关键，代表性才是**：2-4 个高质量示例 > 20 个泛泛示例
2. **覆盖边界场景**：最好的示例针对模型最容易犹豫的情况
3. **正负对照**：同时展示"应该报告"和"不应该报告"
4. **格式一致**：所有示例使用完全相同的输入/输出格式

````python
FEW_SHOT_SYSTEM = """你是代码安全审查工具。

--- 示例1（必须报告）---
代码：
```python
API_KEY = "sk-prod-abc123-secret"
````

输出：{"severity": "critical", "category": "hardcoded_credential", "line": 1}

--- 示例2（跳过）---
代码：

```python
# TODO: 优化这个循环
for i in range(len(items)):
    process(items[i])
```

输出：{"severity": "skip", "reason": "性能优化建议，不在报告范围"}

--- 示例3（边界：可接受访问模式 vs 真问题）---
可接受：`config.get("timeout", 30)` → 跳过（安全的默认值访问）
真问题：`config["timeout"]` 在未验证键存在的情况下 → 报告（KeyError 风险）
"""

````

---

## Task 4.3：JSON Schema 强制结构化输出

### 这题怎么考

这道题重点区分“看起来像 JSON”和“被 API 机制保证的结构化输出”。生产系统需要稳定字段时，应使用 tool use 的 `input_schema`，再从 `tool_use.input` 读取结果；仅在 prompt 里要求 JSON 不能消除语法错误。还要记住 schema 只能约束形状，不能保证金额求和、字段语义、来源一致性都正确，所以后续仍需要业务校验。

### 三种结构化输出方式对比

| 方式 | 可靠性 | 适用场景 |
|------|--------|---------|
| 普通提示要求 JSON | 低（可能输出说明文字 + JSON）| 不推荐生产环境 |
| `tool_choice: "any"` + JSON Schema | 高（保证调用工具，返回结构化） | 文档类型未知时 |
| `tool_choice: {"type": "tool", "name": "..."}` | 最高（强制特定工具）| 已知必须先运行特定提取 |

### temperature 与结构化输出

```python
# 结构化提取使用低 temperature（减少随机性，提高一致性）
response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=512,
    temperature=0,    # 0 = 最确定性；默认值通常是 1.0
    tools=[extraction_tool],
    tool_choice={"type": "any"},
    messages=messages
)

# 创意生成使用高 temperature（增加多样性）
response = client.messages.create(
    temperature=1.0,   # 较高随机性
    ...
)
````

### JSON Schema 设计模式

```python
extraction_tool = {
    "name": "extract_invoice_data",
    "description": "从发票文档中提取结构化数据",
    "input_schema": {
        "type": "object",
        "properties": {
            # 必须有的字段
            "invoice_number": {
                "type": "string",
                "description": "发票号码，如不存在填 'UNKNOWN'"
            },
            # nullable 字段（可能文档中不存在）
            "total_amount": {
                "type": ["number", "null"],  # 防止模型编造不存在的金额
                "description": "发票总金额（如文档中不明确则为 null）"
            },
            # enum 约束（有限选项）
            "currency": {
                "type": "string",
                "enum": ["USD", "EUR", "CNY", "GBP", "other"],
                "description": "货币类型"
            },
            # other + detail 模式（处理 enum 无法穷举的情况）
            "currency_detail": {
                "type": ["string", "null"],
                "description": "当 currency 为 'other' 时填写具体货币名称"
            },
            # 置信度字段（帮助下游判断是否需要人工审核）
            "confidence": {
                "type": "string",
                "enum": ["high", "medium", "low", "unclear"],
                "description": "提取置信度"
            },
            # 数组字段
            "line_items": {
                "type": ["array", "null"],
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "amount":      {"type": "number"}
                    },
                    "required": ["description", "amount"]
                }
            }
        },
        "required": ["invoice_number", "currency", "confidence"]
        # total_amount、currency_detail、line_items 不在 required 中
        # 它们是 nullable，可以返回 null
    }
}
```

### Schema 设计原则速查

| 设计决策           | 原则                                           |
| ------------------ | ---------------------------------------------- |
| **字段可能不存在** | 设为 `["type", "null"]` nullable，防止模型编造 |
| **有限选项**       | 用 `enum` 约束，确保值可机器处理               |
| **enum 无法穷举**  | 增加 `"other"` + `detail` 字段                 |
| **不确定情况**     | 加入 `"unclear"` 到 enum，不要让模型猜         |
| **数组**           | 定义 `items` schema，包含必须字段              |

### `tool_choice` 选择指南

```python
# 不确定文档类型，但必须结构化输出（最常见）
tool_choice = {"type": "any"}

# 必须先运行特定提取步骤（强制顺序）
tool_choice = {"type": "tool", "name": "extract_metadata"}

# 可能直接文本回答（不适合需要结构化的场景）
tool_choice = {"type": "auto"}  # 不要用于必须结构化输出的场景
```

---

## Task 4.4：验证、重试与反馈闭环

### 这题怎么考

验证与重试考的是“知道什么时候该重试，什么时候该停止”。如果错误来自格式、字段放错、结构不一致或可验证的语义冲突，重试时要带上原文、失败结果和具体错误；如果信息根本不在源文档里，继续重试只会增加编造风险。反馈闭环则要求把失败和误报变成可分析数据，例如用 `detected_pattern` 记录触发模式，用 `calculated_total`、`stated_total`、`conflict_detected` 暴露语义冲突。

### Retry-with-Error-Feedback 模式

重试时把**具体校验错误**追加到 prompt，引导模型修正：

```python
def retry_with_feedback(document: str, failed_result: dict, errors: list) -> dict:
    """携带具体错误信息重试，而非简单地"再试一次" """
    retry_prompt = f"""之前的提取结果有以下校验错误，请修正并重新提取：

错误列表：
{chr(10).join(f"- {e}" for e in errors)}

之前的（错误）结果：
{json.dumps(failed_result, ensure_ascii=False)}

原始文档：
{document}

请修正上述每一个错误后重新提取。"""

    return client.messages.create(
        tools=[extraction_tool],
        tool_choice={"type": "any"},
        messages=[{"role": "user", "content": retry_prompt}]
    )
```

### 验证类型与处理方式

| 验证失败类型         | 原因                                    | 处理方式                 |
| -------------------- | --------------------------------------- | ------------------------ |
| JSON 语法错误        | 极少发生（tool use 基本消除）           | 重试                     |
| 语义错误（格式不对） | 日期格式 "2024.01.15" 而非 "2024-01-15" | 携带具体错误重试         |
| 业务规则错误         | 金额为负数、日期超出合理范围            | 携带具体错误重试         |
| 信息不存在于文档     | 文档本身没有此字段                      | 不要重试，标记为 null    |
| 信息模糊无法确认     | 文档中的数字无法确定是税前还是税后      | 标记为 unclear，停止重试 |

### 自纠式验证（Computed vs Stated）

通过计算交叉验证提取结果：

```python
def validate_invoice(extracted: dict) -> list[str]:
    errors = []

    # 1. 格式验证
    if extracted.get("invoice_date"):
        if not re.match(r"\d{4}-\d{2}-\d{2}", str(extracted["invoice_date"])):
            errors.append(
                f"invoice_date 格式错误，应为 YYYY-MM-DD，"
                f"当前值: '{extracted['invoice_date']}'"
            )

    # 2. 数值范围验证
    if extracted.get("total_amount") is not None:
        if extracted["total_amount"] <= 0:
            errors.append("total_amount 必须大于 0")

    # 3. 交叉验证（计算总额 vs 声明总额）
    if extracted.get("line_items") and extracted.get("total_amount"):
        computed = sum(item["amount"] for item in extracted["line_items"])
        stated = extracted["total_amount"]
        if abs(computed - stated) > 0.01:  # 允许浮点误差
            errors.append(
                f"行项目之和 ({computed}) 与声明总额 ({stated}) 不符，"
                f"差值: {stated - computed}"
            )

    return errors
```

---

## Task 4.5：批处理策略

### 这题怎么考

批处理题的判断入口是 SLA。Message Batches API 用更长的处理窗口换成本优势，适合夜间报告、周审计、批量分类和批量测试生成；不适合 PR 合并前阻塞检查或用户等待中的交互流程。设计时必须用 `custom_id` 关联输入输出，并按失败原因只重跑失败项；大批量提交前先用样本集调 prompt，否则一次批量失败会放大等待时间和成本。

### Message Batches API

| 特性         | 说明                                 |
| ------------ | ------------------------------------ |
| **成本节省** | 约 50%（相比标准 API）               |
| **处理时间** | 最长 24 小时处理窗口（异步）         |
| **最大批量** | 每批最多 10,000 个请求               |
| **适用场景** | 延迟容忍、非阻塞、离线任务           |
| **不适用**   | 阻塞性工作流、多轮工具调用、实时交互 |

### 适合 vs 不适合 Batch API

```
适合（24小时内完成即可）：
- 夜间技术债分析报告
- 批量文档分类（大量非结构化文档）
- 大规模代码库扫描
- 历史数据标注

不适合（需要立即响应）：
- PR 合并前的阻塞性检查
- 实时客户服务响应
- 需要在单个请求中多轮工具调用的任务（Batch API 不能中途执行工具并把结果返回同一请求继续推理）
- 依赖上一个结果的顺序处理
```

### 批处理实现模式

```python
import anthropic

client = anthropic.Anthropic()

# 创建批量请求
batch_requests = [
    {
        "custom_id": f"doc_{doc_id}",   # 用于匹配请求和结果
        "params": {
            "model": "claude-haiku-4-5-20251001",
            "max_tokens": 512,
            "messages": [{"role": "user", "content": f"分类文档：{doc_text}"}]
        }
    }
    for doc_id, doc_text in documents.items()
]

# 提交批次
batch = client.messages.batches.create(requests=batch_requests)

# 轮询结果（可延迟到24小时后）
while True:
    batch_status = client.messages.batches.retrieve(batch.id)
    if batch_status.processing_status == "ended":
        break
    time.sleep(60)  # 等待1分钟再检查

# 通过 custom_id 匹配结果
for result in client.messages.batches.results(batch.id):
    doc_id = result.custom_id.replace("doc_", "")
    if result.result.type == "succeeded":
        output = result.result.message.content[0].text
```

---

## Task 4.6：多实例评审架构

### 这题怎么考

多实例和多 pass 的考点是“隔离上下文，拆分注意力”。同一模型在同一会话里自查自己刚生成的代码，会继承原先的推理路径，更容易确认自己的决定；独立 Claude 实例没有生成上下文，更适合发现细微问题。大型 review 还要拆成逐文件本地分析和跨文件整合分析，前者找局部 bug，后者看数据流、接口契约和跨模块一致性，避免一次性塞入大量文件导致漏检或结论互相矛盾。

### 独立评审实例 vs 自评

| 方式                     | 可靠性 | 原因                                         |
| ------------------------ | ------ | -------------------------------------------- |
| **独立 Claude 实例评审** | 高     | 不保留生成时的推理上下文，没有"自圆其说"偏见 |
| **同会话自查**           | 低     | 模型倾向于认为自己生成的内容是正确的         |

### 系统提示 vs 用户提示

```python
# System prompt：始终有效的角色定义和约束
system = """你是代码安全审查专家。
只报告明确的安全漏洞，不报告风格或性能问题。
输出 JSON 数组。"""

# User prompt：具体任务
user = f"""请审查以下代码变更：
<code>
{diff_content}
</code>"""

# 将"不变的约束"放 system，"变化的内容"放 user
```

### 大型代码库评审策略

```
Phase 1: 逐文件本地分析（独立任务）
→ 每个文件一个独立 Claude 调用
→ 只报告该文件内部的问题
→ 防止上下文过载（文件数量多时尤为重要）

Phase 2: 跨文件整合分析（独立运行，传入 Phase 1 结果）
→ 专注于：文件间数据流、接口兼容性、跨模块一致性
→ 输入：Phase 1 的结构化摘要（而非完整代码）
→ 不要与 Phase 1 混合进同一上下文
```

### 置信度校准流程

```
1. 模型输出每条 finding 的置信度（high/medium/low）
        ↓
2. 用带标签的验证集（已知正确答案）校准阈值
   （不要直接使用未经校准的置信分，模型的"高"不等于实际高）
        ↓
3. 设定分流阈值（基于校准后的准确率）：
   - 高置信（>0.95 校准准确率）→ 自动处理
   - 中置信（0.7-0.95）→ 抽样人工审核
   - 低置信（<0.7）→ 全量人工审核
        ↓
4. 持续监控：定期从"高置信"样本中随机抽样，验证真实错误率
```

---

## 讲透这一域：提示工程、结构化输出与质量闭环

Domain 4 不是在考“会不会写 prompt”，而是在考你能不能把 prompt 当成可迭代、可验证、可治理的软件组件。生产系统里的提示不是一次性文案，它要有明确标准、示例、结构化输出、校验、重试、批处理策略和评审架构。考试题常见形式是：一个自动审查或数据提取系统误报多、格式不稳定、批量成本高、结构化结果语义错误，然后让你选择最有效的改进方案。正确答案通常不是“让 Claude 更仔细”，而是把目标转成具体标准、schema、few-shot、验证循环或多实例评审。

### 一、显式标准比抽象语气更可靠

“Be conservative”“only high confidence”“be careful”这些提示听起来合理，但对模型来说不可执行。它们没有告诉模型哪些类别要报、哪些类别要跳过、证据门槛是什么、严重级别如何划分。显式标准则不同：报告硬编码凭据、SQL 注入、权限绕过；跳过格式风格、局部命名偏好、TODO 注释；只有当注释声明的行为与代码实际行为矛盾时才报告注释问题。这样的规则能直接改变模型判断边界。

误报率会影响信任。开发者如果连续看到低价值 finding，会开始忽略所有输出，包括真正严重的问题。考试里如果题目说某个类别误报特别多，最佳策略可能是暂时关闭这个类别，先恢复用户对系统的信任，再用更具体的标准和 few-shot 优化。不要试图用一句“只报告高置信问题”解决误报，因为模型自报置信不是稳定过滤器。

严重级别也需要显式定义。Critical 是可利用安全漏洞或数据丢失，High 是明确功能错误或权限绕过，Medium 是边界条件下失败，Low 是可维护性建议。每个级别最好有代码例子。没有例子的严重级别会漂移，同类问题今天被判 High，明天被判 Medium。结构化审查系统要追求一致性，而不是每次语言流畅。

### 二、Few-shot 是教判断，不只是教格式

Few-shot 的价值不只是让输出 JSON 长得一样，更重要的是展示边界案例。比如什么样的访问模式是安全默认值，什么样的是 KeyError 风险；什么样的测试缺口值得报告，什么样只是分支覆盖形式主义；什么样的文档字段应提取，什么样应该返回 null。好的 few-shot 让模型学会原则，能泛化到新情况。

示例数量不必多，2-4 个高质量示例通常比 20 个普通示例更好。每个示例应针对模型容易犹豫的地方，并包含正反对照。如果只给正例，模型会倾向于多报；加上“应跳过”的负例，才能减少误报。示例还要格式一致，输入、推理依据、输出字段都稳定，否则模型会学习到混乱格式。

在提取任务中，few-shot 能减少幻觉。比如发票总额可能出现在表格底部，也可能在页眉，也可能以手写形式出现；引用可能是 inline citation，也可能是 bibliography；方法信息可能在 methodology section，也可能嵌在脚注。示例展示这些结构差异后，模型更容易找到真实字段，而不是在 required 字段压力下编造。

### 三、结构化输出要靠 tool use 和 JSON Schema

只在 prompt 里说“请输出 JSON”不够可靠。模型可能输出 Markdown 包裹、解释文字、尾注，甚至 JSON 语法错误。工具调用配合 JSON Schema 是更可靠的结构化输出方式，因为模型生成的是 tool input，天然受 schema 约束。`tool_choice: "any"` 能保证模型必须调用一个工具，forced tool 能保证调用指定工具。

但 schema 解决的是语法和形状，不解决语义。发票 line_items 都符合 schema，不代表它们相加等于 total；日期字段是字符串，不代表它来自正确位置；类别是 enum，不代表选择语义正确。因此结构化输出后仍需要业务校验。考试里如果选项说“用了 strict schema 所以不需要验证”，通常是错的。正确方案是 schema + semantic validation。

schema 设计要给模型合法出口。可能缺失的字段设为 nullable；模糊类别加 `unclear`；有限但可扩展类别用 `other + detail`；金额、日期、单位要在 prompt 中给 normalization rules。不要把所有字段都 required，否则模型会为了满足 schema 编造。结构化提取系统的目标不是填满表格，而是忠实表达源文档。

### 四、验证、重试和反馈闭环

重试有效的前提是错误可修正。日期格式错、字段位置放错、总额计算不一致、输出结构不符合要求，这些可以把具体错误反馈给模型重试。反馈必须具体，例如“line_items sum is 128.50 but stated_total is 132.50”，而不是“结果不对，请重试”。重试 prompt 应包含原文、失败结果和验证错误，让模型有依据自我修正。

重试无效的情况也要识别。如果信息根本不在提供的文档里，重试只会增加幻觉概率。正确做法是返回 null、unclear 或升级人工，而不是继续尝试。考试题如果说“需要的信息在外部文档中，但没有提供给模型”，最佳答案通常不是 retry，而是获取缺失来源或标记无法判断。

反馈闭环还包括分析误报模式。结构化 finding 中加入 `detected_pattern`，能统计哪些代码模式经常触发但被开发者 dismiss。比如所有“缺少注释”的 findings 都被忽略，就说明该类别低价值；某类 SQL 构造经常被误判，就需要调整标准或示例。没有结构化模式字段，团队只能看零散评论，无法系统改进。

### 五、批处理要匹配 SLA

Message Batches API 适合非阻塞、延迟容忍、大规模任务，例如夜间文档提取、周审计和离线测试生成。它的优势是成本和吞吐，限制是最长 24 小时处理窗口、没有低延迟 SLA、不能在处理中途执行多轮工具调用。预合并检查、实时用户请求、前置条件校验和多轮 agent loop 都不适合 batch。考试题如果强调“开发者正在等待合并结果”，通常应选同步 API；如果强调“每晚处理数万文档”，才适合 batch。

Batch 还需要 `custom_id` 关联输入输出。没有 custom_id，失败后很难知道哪个文档需要重跑。失败处理也不应全量重跑，而是只重提交失败文档，并按失败原因修改输入。超上下文的文档要 chunk，格式异常的文档要改 prompt 或预处理，schema 错误要修工具定义。大批量运行前先用样本集调 prompt，可以显著降低返工成本和等待时间。

批处理频率要倒推 SLA。比如业务要求 30 小时内完成，而 batch 可能花 24 小时，就不能等任务堆满后才提交；可能要每 4 小时或更短窗口提交，给排队和失败重跑留余量。考试里如果同时给出 SLA 和 batch window，不能只说“batch 最多 24 小时所以满足”，还要计算提交窗口、处理窗口和重试窗口。

### 六、多实例、多 pass 审查减少偏差

模型自审有天然限制。同一个会话生成代码后，模型保留了当时的推理上下文，更容易相信自己的设计是合理的。独立审查实例没有这些上下文，更像一个新 reviewer，能发现微妙问题。扩展思考不等于独立审查，长时间让同一个模型反思也可能只是强化原假设。

大型 review 要多 pass。逐文件 pass 关注本地 bug、异常处理、边界条件；跨文件 pass 关注数据流、权限传播、接口契约、状态一致性。把所有文件和所有审查维度一次塞进去，会注意力稀释。多 pass 还能减少矛盾 finding：局部 pass 不负责跨文件结论，integration pass 基于结构化摘要做全局判断。

置信度可以作为分流信号，但必须校准。模型说 high confidence 不代表实际高准确；要用带标签验证集估计不同置信度的真实正确率。高置信可自动处理或抽样审核，中低置信进入人工复核。置信度未经校准就用来自动关闭人工审核，是高风险做法。

### 七、Domain 4 的答题框架

遇到 Prompt Engineering 题，先判断失败类型。误报多，优先显式 report/skip 标准和 few-shot 反例；格式不稳定，优先 tool use + JSON Schema；语法正确但业务错，优先 semantic validation；可修正错误反复出现，优先 retry-with-error-feedback；信息缺失，停止重试并返回 null/unclear；大规模离线任务，考虑 batch；实时阻塞任务，使用同步；审查质量差，使用独立实例和多 pass。

错误答案通常有这些特征：只说“be more careful”；用自报置信当唯一过滤；所有字段 required；认为 schema 消除所有错误；对缺失信息无限 retry；把 batch 用于 pre-merge checks；让同一会话自审；把多文件审查塞进一次调用。正确答案会把语言问题转成结构化、可验证、可迭代的工程机制。

### 八、把 Prompt 当成可测试资产

生产级 prompt 应该像代码一样测试。一个 prompt 上线前，应有样例集、预期输出、评分标准、失败样本和版本记录。只拿一两个手工输入试一下，然后觉得结果不错就上线，是高风险做法。尤其是审查、提取、分类这类任务，prompt 的小改动可能显著改变误报率或漏报率。Domain 4 的很多题其实是在考“有没有 eval 思维”。

评估集不一定一开始就很大，但必须覆盖关键边界。代码审查 prompt 应包含真正漏洞、可接受模式、风格噪音、跨文件问题；文档提取 prompt 应包含标准文档、缺字段文档、格式混乱文档、冲突数据；结构化输出 prompt 应包含未知类别、other、unclear、nullable 字段。没有边界样本，就不知道 prompt 是否只在理想输入上有效。

Prompt 版本管理也重要。每次调整 report/skip 标准、few-shot、schema、temperature，都应记录为什么改、解决什么失败、是否引入新问题。自动化系统中，prompt 是行为的一部分；不管理 prompt 版本，就无法解释某天开始误报突然增加的原因。考试虽然不一定直接说“版本控制”，但凡题目提到持续改进和 dismissal pattern，就要想到反馈闭环和系统化分析。

### 九、结构化输出系统的端到端链路

一个完整的数据提取系统不是“Claude 输出 JSON”这么简单。输入层要做文档预处理和 chunk；提示层要给任务、字段定义、格式规则和 few-shot；工具层要用 JSON Schema 约束结构；校验层要检查 schema 之外的业务语义；重试层要带具体错误反馈；人工审核层要处理低置信和冲突；监控层要统计字段级准确率和错误模式。Domain 4 的每个 Task 都对应这条链路的一部分。

例如发票提取：模型先识别文档类型，选择 invoice extractor；schema 要求 invoice_number、currency、line_items；total_amount 允许 null；currency 有 enum 和 other_detail；prompt 说明日期统一 ISO、金额统一数字；校验器检查 line_items 合计与 stated_total；不一致时重试，并给出具体差异；仍不一致则输出 conflict_detected 并进入人工审核。这个流程比“请提取发票 JSON”可靠得多。

代码审查系统也类似：prompt 定义 report/skip 类别，few-shot 展示边界，输出 schema 包含 file、line、severity、category、detected_pattern、confidence、suggested_fix；后处理去重，历史 findings 传入避免重复评论；高误报类别通过 dismissal pattern 分析后调整。这样审查结果才能进入 CI 和 PR，而不是成为开发者忽略的噪音。

### 十、temperature 与确定性

结构化提取、分类、审查这类任务通常需要低 temperature，因为目标是稳定和可复现。创意生成、头脑风暴、文案探索可以提高 temperature，因为目标是多样性。考试里如果场景要求“稳定产生相同结构化输出”，选择低 temperature；如果要求“提出多种创意方向”，高 temperature 才合理。

但 temperature 不是万能稳定器。低 temperature 不能弥补模糊标准，也不能保证字段语义正确。它只是降低随机性。真正的可靠性来自明确标准、schema、few-shot、验证和反馈。题目如果把“把 temperature 调到 0”作为唯一修复误报或语义错误的方案，通常不完整。

### 十一、Prompt 与工具的分工

Prompt 适合表达判断标准、上下文、格式偏好和任务目标；工具/schema 适合强制结构；程序校验适合确定性规则；人工审核适合高风险和不确定场景。把所有事情都放 prompt，会让系统脆弱；把所有事情都放 schema，又无法表达复杂语义；只靠人工审核则成本太高。Domain 4 的最佳答案通常体现分层：prompt 指导，tool use 约束，validator 检查，retry 修正，human review 兜底。

例如“减少误报”不能只靠 schema，因为误报是语义判断问题；“保证 JSON 合法”不能只靠 prompt，因为格式约束更适合 tool use；“金额合计必须一致”不能只靠模型自觉，因为程序可以确定性检查；“来源冲突但业务风险高”不能让模型硬选，因为需要人工或协调者裁决。复习时要问：这个问题最适合在哪一层解决？

### 十二、考试中的排除法

Domain 4 选项里，凡是含糊词很多、机制很少的通常要警惕。例如“ask Claude to be more accurate”“tell it to avoid mistakes”“increase confidence threshold”“retry several times”都不是充分工程方案。更好的选项会具体说明 report/skip 标准、few-shot 边界、tool_choice、JSON Schema、validation errors、custom_id、independent instance、per-file pass 等机制。

另一个排除法是看是否承认限制。好方案会承认 schema 不防语义错，retry 不解决缺失信息，batch 不适合低延迟，多实例比自审更客观，置信度需要校准。坏方案往往声称某个单一手段解决全部问题。官方考试更偏向生产判断，因此能说明边界和权衡的选项通常更优。

### 十三、从两个典型系统理解 Domain 4

第一个典型系统是自动代码审查。它的失败模式通常是误报、重复评论、严重级别不一致、跨文件问题漏检。解决路径是：先定义明确 report/skip 标准，关闭高误报类别；用 few-shot 展示应报和不应报的边界；输出结构化 finding，包含位置、类别、严重级别、修复建议和 detected_pattern；对大型 PR 做逐文件 pass 和跨文件 pass；用独立实例审查；把旧 findings 传入避免重复。这里每一步都对应官方 task。

第二个典型系统是文档结构化提取。它的失败模式通常是 JSON 不合法、字段缺失时编造、不同版式提取不稳、总额不一致、批量处理成本高。解决路径是：用 tool use 和 JSON Schema 保证结构；字段可能缺失时设 nullable；对模糊类别加 unclear；用 few-shot 覆盖多种版式；用校验器检查金额、日期、字段位置；错误可修正时 retry-with-feedback；信息缺失时停止重试；离线大规模任务用 batch，并用 custom_id 重跑失败项。

这两个系统说明 Domain 4 的核心不是写漂亮 prompt，而是围绕输出质量建立闭环。prompt 定义目标，few-shot 定义边界，schema 定义形状，validator 定义正确性，retry 定义修复方式，batch 定义成本策略，multi-pass 定义审查架构。题目里的最佳答案通常会把这些机制组合起来，而不是只选一个孤立技巧。

### 十四、如何写出可维护的 Prompt

可维护 prompt 应该分区清楚。用 XML 或 Markdown 标题区分任务、输入、报告类别、跳过类别、严重级别、示例、输出格式、约束。变化频繁的输入放 user message，不变的角色和规则放 system。示例要短而精准，不要让示例淹没规则。输出格式要与下游系统一致，字段名稳定，枚举稳定，缺失值策略稳定。

可维护 prompt 还应避免隐含标准。例如“报告重要问题”不如“报告会导致安全漏洞、数据丢失、用户可见功能错误的问题”；“不要太啰嗦”不如“每条 finding 最多 80 字，必须包含一条可执行修复建议”；“提取所有信息”不如“只提取 schema 中列出的字段，源文档没有则返回 null”。越具体，越能评估和迭代。

### 十五、把错误样本变成资产

每次模型输出失败，都是改进数据。误报可以变成 negative few-shot；漏报可以变成 positive few-shot；格式错误可以变成 schema 或 prefill 调整；语义错误可以变成 validator；人工纠正可以进入校准集；批量失败可以按 custom_id 分类。不要只修当前一次输出，要把失败模式沉淀成测试样本。这样 prompt 会越来越稳，而不是每次靠人工临场补救。

考试如果问“如何持续降低误报”或“如何分析开发者 dismiss 的 findings”，答案应包含结构化字段和反馈分析，而不是只说“调整 prompt”。持续改进需要可聚合的数据，`detected_pattern`、category、severity、dismiss_reason 都是让 prompt 工程变成工程的关键。

### 十六、把“模型输出”拆成三种质量

评估 Claude 输出时，可以拆成格式质量、语义质量和业务质量。格式质量指 JSON 是否合法、字段是否齐全、枚举是否正确；语义质量指字段是否来自正确文本、分类是否符合含义、finding 是否真是问题；业务质量指这个结果对用户或流程是否有价值，例如审查意见是否可执行、提取结果是否能进入下游系统、低置信字段是否进入人工审核。不同质量层要用不同机制保证。

格式质量主要靠 tool use、JSON Schema、prefill、stop sequence 或输出解析；语义质量主要靠 explicit criteria、few-shot、验证规则和重试反馈；业务质量主要靠产品规则、人工审核、置信度校准、误报监控和用户反馈。考试里如果题目只说“JSON 经常坏”，不要优先讨论 few-shot；如果题目说“JSON 合法但金额不对”，不要以为 schema 已经解决；如果题目说“开发者不信任审查结果”，重点是误报类别和信任恢复。

### 十七、Prompt 工程的最终目标

Prompt 工程不是让单次回答看起来更聪明，而是让系统在大量输入、边界场景、多人协作、自动化执行中仍然稳定。真正成熟的 prompt 会降低解释空间，明确不确定性，给模型合法的“无法判断”出口，并把失败变成可改进的数据。它不是一次写完的文案，而是伴随评估集、schema、校验器和监控一起演进的系统资产。

### 十八、考场上的最终判断原则

如果一个选项只是改变措辞，而没有改变信息结构、约束机制或反馈路径，通常不是最佳答案。比如“请更准确”“请更保守”“请再检查一遍”都属于弱干预。强干预会改变系统行为：增加 report/skip 标准，加入 few-shot 反例，强制 tool_use，调整 schema nullable，加入校验器，把错误反馈给重试，把批处理失败按 custom_id 重跑，或用独立实例审查。

还要看方案是否适配任务成本和延迟目标。实时任务追求低延迟，不应为了省钱切到 batch；离线任务追求吞吐，可以接受 24 小时窗口；高风险输出需要人工审核，低风险高置信输出可以自动化；边界样本少时先建立 eval，不要直接扩大规模。Domain 4 的高分答案通常体现“质量、成本、延迟、风险”四个维度的平衡。

把这部分学透后，你会发现提示工程不是玄学。它有输入设计、示例设计、输出约束、验证反馈、批量执行和审查架构。考试真正想确认的是：你是否能把模型输出从一次性文本变成可依赖的生产流程。

我会把 prompt 当成一段会出 bug 的代码：有测试样本，有失败记录，也有回滚理由。

---

## 临考速查

1. **减少误报** → 明确列出报告/跳过的类别，不依赖模糊的"置信度过滤"
2. **格式不稳定** → Few-shot 示例（比长篇说明更有效）
3. **Few-shot 设计** → 针对边界场景，正负对照，2-4个为宜
4. **保证工具调用** → `tool_choice: "any"`（不是 `"auto"`）
5. **temperature=0** → 结构化提取、分类任务；高 temperature → 创意生成
6. **nullable 字段** → 设为 `["type", "null"]`，防止模型编造
7. **重试有效** → 携带具体校验错误（日期格式错、金额为负等）
8. **重试无效** → 信息本身不在文档中 → 标记为 null，停止重试
9. **批处理** → 约 50% 成本折扣，最长 24 小时处理窗口；**不适合**多轮工具调用、阻塞流程或实时任务
10. **独立评审** → 比同会话自查更可信；大型 PR 先逐文件后跨文件
11. **XML 标签** → 清晰分隔提示的不同部分（约束/示例/格式）
12. **Prefill** → 预填充 assistant 轮次引导特定格式
