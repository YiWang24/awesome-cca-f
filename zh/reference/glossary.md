# 技术词汇表（中英对照）

> CCA-F 考试涉及的核心技术术语，按字母顺序排列。

---

## A

| 英文 | 中文 | 说明 |
|------|------|------|
| **Agent** | 智能体 | 能够自主执行多步骤任务的 AI 系统 |
| **Agent SDK** | 智能体软件开发工具包 | Claude 用于构建智能体应用的 SDK |
| **Agentic Loop** | 智能体循环 | 基于 `stop_reason` 驱动的循环：`tool_use` 继续，`end_turn` 终止 |
| **AgentDefinition** | 智能体定义 | 配置子智能体的系统提示、工具列表和描述的对象 |
| **allowedTools** | 允许的工具列表 | AgentDefinition 中限制智能体可调用工具的字段；生成子智能体必须包含 `"Task"` |
| **API (Application Programming Interface)** | 应用程序接口 | Claude 提供的编程接口 |
| **argument-hint** | 参数提示 | SKILL.md frontmatter 字段，调用者未传参数时显示的提示信息 |

---

## B

| 英文 | 中文 | 说明 |
|------|------|------|
| **Batch API / Message Batches API** | 消息批处理 API | 异步批量处理请求，约 50% 成本折扣，最长 24 小时处理窗口；不支持多轮工具调用 |
| **Bash** | Bash 工具 | Claude Code 内置工具，执行 shell 命令；唯一能运行测试、构建和安装的工具 |

---

## C

| 英文 | 中文 | 说明 |
|------|------|------|
| **Case Facts** | 案件事实块 | 关键事实（金额、日期、订单号、状态）的持久化结构，每轮注入防止长对话丢失 |
| **CI/CD** | 持续集成/持续部署 | 自动化软件构建、测试和部署流水线 |
| **Claim-Source Mapping** | 声明-来源映射 | 多源综合中将每个结论绑定到具体来源（URL、文档名、摘录、日期）的结构 |
| **CLAUDE.md** | Claude 配置文件 | Claude Code 读取的项目/用户级配置；用户级不通过版本控制共享 |
| **Claude API** | Claude API | 与 Claude 模型交互的 REST API |
| **Claude Code** | Claude Code | Anthropic 的命令行代码智能体工具 |
| **Confidence Calibration** | 置信度校准 | 用带标签的验证集校准模型输出的置信分，使其反映真实准确率 |
| **context: fork** | 上下文分叉 | SKILL.md frontmatter 选项，使 Skill 在隔离子智能体上下文中运行，防止输出污染主对话 |
| **Context Window** | 上下文窗口 | 模型单次处理的最大 token 数量（因模型版本不同而异） |
| **Coordinator** | 协调者/协调智能体 | Hub-and-Spoke 架构中统一管理任务分解、委托、聚合和错误处理的主智能体 |
| **Coverage Gap** | 覆盖缺口 | 多智能体研究中某个子主题因来源不可用或任务分解过窄而未被覆盖的区域 |
| **custom_id** | 自定义 ID | Batch API 中用于匹配请求与结果的标识符 |

---

## D

| 英文 | 中文 | 说明 |
|------|------|------|
| **Dynamic Task Decomposition** | 动态任务分解 | 基于探索中间发现自适应生成子任务，适合遗留系统或开放式研究 |

---

## E

| 英文 | 中文 | 说明 |
|------|------|------|
| **Edit** | Edit 工具 | Claude Code 内置工具，基于唯一文本锚点进行局部文件修改；锚点不唯一时失败 |
| **end_turn** | 结束轮次 | `stop_reason` 的值，表示模型已完成当前任务，循环终止 |
| **enum** | 枚举约束 | JSON Schema 中限制字段只能取预定义值集合中的一个 |
| **Escalation** | 升级 | 将问题从智能体移交人工的过程；用户明确要求人工时应立即触发 |
| **errorCategory** | 错误类别 | 结构化错误中标识错误性质的字段：`transient`/`validation`/`permission`/`business` |
| **Explore Subagent** | 探索子智能体 | 用于隔离详细发现输出、保留主智能体上下文的子智能体模式 |

---

## F

| 英文 | 中文 | 说明 |
|------|------|------|
| **Few-Shot Prompting** | 少样本提示 | 在提示中包含少量示例（通常2-4个）引导模型输出稳定格式；格式不稳定时优先于堆砌说明 |
| **fork_session** | 分叉会话 | 从共享基线创建独立会话分支，用于并行探索不同实现方案 |

---

## G

| 英文 | 中文 | 说明 |
|------|------|------|
| **Glob** | Glob 工具 | Claude Code 内置工具，按文件**路径/名称模式**查找文件（记忆：Glob 查路径） |
| **Grep** | Grep 工具 | Claude Code 内置工具，在文件**内容**中搜索模式（记忆：Grep 查内容） |

---

## H

| 英文 | 中文 | 说明 |
|------|------|------|
| **Handoff** | 交接 | 升级给人工时的结构化传递，应包含客户ID、根因、已尝试动作、推荐行动 |
| **Hooks** | 钩子 | 在特定事件触发时执行的代码：`PostToolUse`（规范化）、`PreToolUse`（拦截/阻止）|
| **Hub-and-Spoke** | 枢纽辐射架构 | 协调者统一管理所有子智能体通信；子智能体之间不直接通信 |
| **Hierarchical Summarization** | 分层摘要 | 两阶段压缩策略：先对每个章节或文档独立摘要，再将章节摘要合并为最终综述。比单次压缩保留更多结构，每条结论仍可追溯到具体来源。 |
| **Human-in-the-Loop** | 人在回路 | 在自动化流程关键节点加入人工审核的设计模式 |

---

## I

| 英文 | 中文 | 说明 |
|------|------|------|
| **Interview Pattern** | 访谈模式 | 让 Claude 先提问暴露设计约束，再开始实现；适合缓存策略、数据迁移等复杂设计 |
| **isError** | 是否错误 | MCP 工具响应中标记操作失败的布尔字段（`true` = 失败，`false` 或缺失 = 成功） |
| **isRetryable** | 是否可重试 | 错误响应中标记该错误是否值得重试（瞬时错误 `true`，权限/业务错误 `false`）|

---

## J

| 英文 | 中文 | 说明 |
|------|------|------|
| **JSON Schema** | JSON 模式 | 用于定义和验证 JSON 数据结构的规范；通过 `tool_use` + Schema 可保证结构化输出 |

---

## L

| 英文 | 中文 | 说明 |
|------|------|------|
| **Lost in the Middle** | 中间丢失效应 | 模型对长输入开头和结尾更稳，中间信息容易被遗漏；重要信息应前置或后置 |

---

## M

| 英文 | 中文 | 说明 |
|------|------|------|
| **Manifest** | 清单文件 | 长任务的状态文件，记录已完成/待处理模块和发现文件，用于崩溃恢复 |
| **max_tokens** | 最大生成 token 数 | API 参数；`stop_reason` 为 `"max_tokens"` 时表示响应被截断，不等于任务完成 |
| **MCP (Model Context Protocol)** | 模型上下文协议 | Anthropic 制定的标准化工具和资源接口协议 |
| **MCP Resources** | MCP 资源 | 只读内容目录（issue列表、文档树、数据库schema），减少探索性工具调用 |
| **MCP Server** | MCP 服务器 | 实现 MCP 协议、提供工具和资源的服务；分 stdio（本地）和 SSE（远程）两种传输类型 |
| **MCP Tools** | MCP 工具 | 通过 MCP 协议暴露的可调用操作，与内置工具一起出现在工具池中 |
| **messages** | 消息历史 | 发送给 API 的对话历史数组，包含 `user`、`assistant` 和 `tool_result` 消息 |
| **multi-agent** | 多智能体 | 多个 Claude 实例协作完成任务的系统架构 |

---

## N

| 英文 | 中文 | 说明 |
|------|------|------|
| **nullable** | 可空字段 | JSON Schema 中 `"type": ["string", "null"]` 允许字段为 null；防止模型编造不存在的信息 |

---

## O

| 英文 | 中文 | 说明 |
|------|------|------|
| **Orchestrator** | 编排者 | 协调者（Coordinator）的同义词；负责分解任务、调度子智能体、聚合结果 |
| **Overlapping Chunks** | 重叠分块 | 文档分块时相邻块保留部分重叠内容，防止信息被截断在块边界处 |

---

## P

| 英文 | 中文 | 说明 |
|------|------|------|
| **Partial Results** | 部分结果 | 子智能体失败时已获取的不完整数据，应随结构化错误一起上报给协调者 |
| **paths** | 路径模式 | `.claude/rules/` 文件 frontmatter 字段，用 glob 模式指定规则只对匹配文件生效 |
| **Plan Mode** | 计划模式 | Claude Code 在执行变更前先探索和规划的工作模式；适合大规模变更、多方案选择 |
| **PostToolUse** | 工具使用后钩子 | 工具执行后、模型处理前触发；常用于数据格式规范化（时间戳转换、状态码映射） |
| **Prefill** | 预填充 | 在 assistant 轮次开头预填充内容，引导模型继续特定格式（如 `[` 引导 JSON 数组）|
| **PreToolUse** | 工具使用前钩子 | 工具调用前触发；可拦截阻止（返回错误）或验证前置条件 |
| **Programmatic Enforcement** | 程序化强制执行 | 用代码/hooks 实现确定性约束；相比提示引导，不存在非零失败率 |
| **Prompt Chaining** | 提示链 | 将复杂任务分解为按顺序执行的多个步骤，每步输出是下步输入 |
| **pause_turn** | 暂停轮次 | `stop_reason` 的值，表示工具调用被外部系统暂停；解决暂停原因后调用恢复接口继续 |

---

## R

| 英文 | 中文 | 说明 |
|------|------|------|
| **Read** | Read 工具 | Claude Code 内置工具，读取文件完整内容 |
| **Retry-with-Error-Feedback** | 携带错误反馈的重试 | 重试时将具体校验错误（不是模糊的"再试一次"）附入提示引导模型修正 |
| **--resume** | 恢复会话参数 | `claude --resume <session-name>` 恢复命名会话；先前上下文仍有效时使用 |

---

## S

| 英文 | 中文 | 说明 |
|------|------|------|
| **Scaled Score** | 标准化分数 | 考试采用100-1000分制，通过线720分 |
| **Scratchpad** | 暂存文件 | 智能体跨上下文边界持久化关键发现的文件，防止长会话上下文退化 |
| **SKILL.md** | 技能配置文件 | 定义 Claude Code Skill 的 Markdown 文件，frontmatter 控制 `context: fork`、`allowed-tools`、`argument-hint` |
| **Skills** | 技能 | 按需调用的工作流模板（对比 CLAUDE.md：始终加载的规范）|
| **Slash Commands** | 斜杠命令 | 以 `/` 开头的自定义命令；项目级放 `.claude/commands/`，个人级放 `~/.claude/commands/` |
| **SSE (Server-Sent Events)** | 服务器发送事件 | MCP 远程传输类型，用于 HTTP 长连接通信（对比 `stdio` 用于本地进程）|
| **stdio** | 标准输入输出 | MCP 本地传输类型，通过进程标准输入输出通信 |
| **stop_reason** | 停止原因 | API 响应字段：`"tool_use"`（继续调工具）、`"end_turn"`（完成）、`"max_tokens"`（截断）|
| **stop_sequence** | 停止序列 | `stop_reason` 的值，表示响应中遇到了预先设定的停止字符串；按业务逻辑处理后续 |
| **Stratified Sampling** | 分层抽样 | 按文档类型、字段维度分别统计准确率，不依赖整体平均值掩盖局部问题 |
| **Structured Error Propagation** | 结构化错误传播 | 子智能体失败时将 `isError`、`errorCategory`、`isRetryable` 和 Partial Results 一起上报给协调者 |
| **Subagent** | 子智能体 | 由协调智能体生成的专业智能体；上下文不自动继承，必须显式传递 |
| **system prompt** | 系统提示 | 对话开始前设置模型角色和约束的特殊提示；不变的规则放 system，可变的任务放 user |

---

## T

| 英文 | 中文 | 说明 |
|------|------|------|
| **Task tool** | Task 工具 | Agent SDK 中生成子智能体的工具；`allowedTools` 必须包含 `"Task"` 才能使用 |
| **TDD (Test-Driven Development)** | 测试驱动开发 | 先写测试再实现，通过测试失败引导迭代；测试失败是精确的反馈，比自然语言描述更准确 |
| **temperature** | 温度参数 | 控制输出随机性；结构化提取用 `0`（确定性最高），创意生成用较高值 |
| **token** | 令牌 | 模型处理文本的基本单位，约4个英文字符或2个汉字 |
| **tool_choice** | 工具选择 | `"auto"`（可选择不调用）/ `"any"`（必须调用某个工具）/ `{"type":"tool","name":"..."}` （强制指定）|
| **tool_result** | 工具结果 | 工具执行结果消息类型，必须包含 `tool_use_id` 并追加到对话历史中 |
| **tool_use** | 工具使用 | `stop_reason` 的值（模型请求调工具）；也指响应内容中的工具调用块 |
| **tool_use_id** | 工具调用 ID | 工具调用的唯一标识符；`tool_result` 中通过 `tool_use_id` 与调用对应，必须匹配 |

---

## W

| 英文 | 中文 | 说明 |
|------|------|------|
| **WebFetch** | WebFetch 工具 | Claude Code 内置工具，抓取指定 URL 页面内容；用于读取外部文档和 API 参考 |
| **WebSearch** | WebSearch 工具 | Claude Code 内置工具，通过搜索引擎查找信息；适合需要最新或未知来源信息时 |
| **Write** | Write 工具 | Claude Code 内置工具，创建或覆盖写入文件；Edit 锚点不唯一时的回退方案 |

---

## X

| 英文 | 中文 | 说明 |
|------|------|------|
| **XML Tags** | XML 标签 | 提示工程中用于清晰分隔不同部分（约束、示例、格式）的标签，如 `<report_categories>` |

---

## 配置文件位置速查

| 文件/目录 | 位置 | 版本控制 | 用途 |
|----------|------|---------|------|
| `CLAUDE.md`（项目级） | `.claude/CLAUDE.md` 或项目根目录 | ✅ 提交 | 团队共享规范 |
| `CLAUDE.md`（用户级） | `~/.claude/CLAUDE.md` | ❌ 不共享 | 个人偏好设置 |
| 项目命令 | `.claude/commands/` | ✅ 提交 | 团队共享命令 |
| 用户命令 | `~/.claude/commands/` | ❌ 不共享 | 个人命令 |
| 路径规则 | `.claude/rules/` | ✅ 提交 | 按主题/路径的规则 |
| Skills | `.claude/skills/` | ✅ 提交 | 可复用工作流 |
| MCP（项目级） | `.mcp.json` | ✅ 提交（密钥用环境变量） | 团队 MCP 服务器 |
| MCP（用户级） | `~/.claude.json` 的 `mcpServers` 字段 | ❌ 不共享 | 个人 MCP 服务器 |

---

## stop_reason 速查

| 值 | 含义 | 循环动作 |
|----|------|---------|
| `"tool_use"` | 模型请求调用工具 | 执行工具 → 追加 `tool_result` → 继续循环 |
| `"end_turn"` | 任务正常完成 | 终止循环，返回最终响应 |
| `"max_tokens"` | 响应被截断 | 不等于完成；需要增大 `max_tokens` 或分批处理 |
| `"stop_sequence"` | 遇到预设停止序列 | 按业务逻辑处理 |
| `"pause_turn"` | 工具调用被外部暂停 | 解决暂停原因后恢复 |

---

## 工具选择速查（Grep vs Glob vs 其他）

| 需求 | 工具 |
|------|------|
| 找函数在哪些文件被调用 | Grep（搜索内容） |
| 找所有 `*.test.tsx` 文件 | Glob（按路径模式） |
| 找 TODO 注释 | Grep（搜索内容关键词） |
| 找 `src/api/` 下所有文件 | Glob（目录模式） |
| 找导入了某模块的文件 | Grep（搜索 import 语句） |
| 修改文件中特定代码段 | Edit（唯一文本锚点） |
| Edit 锚点不唯一时 | Read + Write（完整重写） |
| 运行测试/安装依赖 | Bash |
