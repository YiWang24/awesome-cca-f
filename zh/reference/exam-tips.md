# 备考建议与考试范围说明

---

## 考试心态

CCA-F 不是记忆测试，是架构判断题。

干扰项通常也"能用"，但正确答案是那个**在生产环境最可靠、最确定、最官方的**。选项排序大致是：确定性 > 概率性、程序化 > 提示引导、显式 > 隐式。

---

## 五大答题原则

**1. 程序化 > 提示（涉及确定性时）**
"如何保证某步骤一定执行"——Hooks 或前提条件门控，不是 few-shot 或系统提示。
提示词有非零失败率，审计、合规、金融操作不能接受概率性保障。

**2. stop_reason > 文本解析（循环控制）**
检测 `stop_reason == "end_turn"` 是正确方式。解析响应文本找"任务完成"字样是脆弱方案，容易被输出变体打穿。

**3. 工具描述 > 系统提示（工具选错时）**
智能体频繁路由错工具，第一步改工具描述（加输入格式、触发场景、示例、边界说明）。改系统提示是第二步。

**4. 显式传递 > 隐式继承（多智能体）**
子智能体不自动获取协调者的历史。每次调用 Task 工具时，上下文必须主动传进去。

**5. 动态分解 > 固定流水线（开放式任务）**
探索遗留代码库、分析未知数据结构——先侦察再计划，不是提前写死所有步骤。

---

## 考题信号词 → 答案方向

| 信号词 / 场景 | 正确方向 | 排除方向 |
|---|---|---|
| "确定性"、"合规"、"必须先…才能…" | Hooks / 前提条件门控 | few-shot、系统提示 |
| "工具选择错误"、"频繁误路由" | 改工具描述 | 改系统提示、加 few-shot |
| "无限循环"、"智能体不终止" | 检查 `stop_reason`、确保 `tool_result` 正确追加 | 加系统提示约束轮数 |
| "子智能体缺少信息" | 协调者显式传递上下文 | 子智能体自行推断 |
| "遗留系统"、"结构未知"、"开放式任务" | 动态分解（先探索） | 固定提示链 |
| "多个独立子任务"、"需要加速" | 单次响应调用多个 Task = 并行 | 串行顺序调用 |
| "工具调用成功但数据格式不一致" | PostToolUse Hook 规范化 | PreToolUse Hook |
| "禁止某类工具被调用" | PreToolUse Hook 拦截 | PostToolUse Hook |
| "需要在 CI/CD 中运行" | `--print` / `-p` 标志 + `--output-format json` | 交互式模式 |
| "结构化输出字段偶尔缺失" | `required` + `nullable` 组合约束 | 在提示词中描述格式 |
| "低置信度时触发人工审核" | 添加 `confidence` 字段 + 阈值判断 | 直接返回结果 |
| "多步骤、有风险操作" | Plan Mode 先确认 | 直接执行 |
| "上下文超长、历史太多" | 摘要 + 分层压缩，或 `/compact` | 直接截断 |
| "多智能体交接" | 结构化摘要而非完整历史 | 传整个消息列表 |
| "跨会话恢复工作" | `--resume`、`fork_session` | 新建会话重来 |
| "Batch 请求，延迟可接受" | Batch API（50% 成本，最长 24h 处理窗口） | 实时 API |
| "从长文档提取特定信息" | 相关片段前置 + XML 标签定位 | 直接全文塞入 |
| "MCP 服务跑在本地进程" | `stdio` 传输 | SSE 传输 |
| "MCP 服务跑在远程 HTTP" | SSE 传输 | `stdio` 传输 |
| "Agent 步骤输出需要给下一步" | 把输出追加到消息历史 | 让 Agent 自己记 |
| "温度为 0 的场景" | 结构化提取、确定性输出 | 创意写作 |

---

## 各 Domain 备考重点

### Domain 1（27%）— 最高权重，优先吃透

- [ ] **智能体循环完整形态**：`stop_reason` 五种值（`end_turn`、`tool_use`、`max_tokens`、`stop_sequence`、`pause_turn`）+ 对应处理逻辑
- [ ] **`tool_result` 消息格式**：必须包含 `tool_use_id`，支持 `is_error: true`，必须追加到 messages 后才能继续请求
- [ ] **Hub-and-Spoke**：协调者不直接执行，子智能体权限隔离，每个子智能体只能访问自己需要的工具
- [ ] **Task 工具**：`allowedTools` 必须显式包含 `"Task"` 才能启动子智能体；`context` 参数传递上下文
- [ ] **并行 vs 串行**：单次响应里 Claude 调用多个 Task = 并行；循环顺序调用 = 串行
- [ ] **Hooks**：PreToolUse（拦截/阻断工具调用），PostToolUse（规范化输出）；配置在 `.claude/hooks/`
- [ ] **程序化强制 vs 提示引导**：合规/审计场景必须用程序化方式，提示词有失败概率
- [ ] **会话恢复**：`--resume`（继续具体会话）、`fork_session`（新建分支）、新会话+摘要（隔离场景）
- [ ] **停止条件**：`max_turns` 限制、错误阈值检测、`pause_turn` 等待人工输入

### Domain 2（18%）

- [ ] **工具描述质量**：必须说清楚"什么时候用"、"输入格式"、"触发条件"和"限制"
- [ ] **工具数量**：超过 ~10 个工具时可靠性下降，考虑拆分成多个专用 Agent
- [ ] **`tool_choice`**：`auto`（默认）/ `any`（必须用某个工具）/ `{"type":"tool","name":"..."}` 强制指定
- [ ] **`tool_use` block 结构**：`id`、`name`、`input` 三个字段，访问方式 `block.id`、`block.name`、`block.input`
- [ ] **错误类型分类**：`errorCategory` 的四个值（`transient`、`validation`、`business`、`permission`）+ `isRetryable`
- [ ] **MCP 传输类型**：`stdio`（本地子进程）vs SSE（远程 HTTP），`stdio` 不跨网络
- [ ] **MCP 配置位置**：`.mcp.json`（项目级）vs `~/.claude.json`（用户级）
- [ ] **Partial Results 模式**：长时间工具调用返回中间状态，防止超时
- [ ] **JSON Schema 约束**：`required` vs 可选字段，`nullable`，`enum` 枚举，`other+detail` 组合模式

### Domain 3（20%）— 细节多，容易失分

- [ ] **CLAUDE.md 三层作用域**：用户级（`~/.claude/CLAUDE.md`）> 项目级（项目根目录）> 目录级（`.claude/CLAUDE.md`）
- [ ] **CLAUDE.md 内容结构**：项目概述、技术栈、目录结构、命名规范、禁止事项（`NEVER do X`）
- [ ] **`.claude/rules/`**：支持 `paths` 字段（glob 匹配），作用于特定文件/目录的细粒度规则
- [ ] **Hooks 触发时机**：`PreToolUse`（工具调用前）、`PostToolUse`（工具调用后）、`Stop`（会话结束）
- [ ] **Plan Mode**：`/plan` 命令进入，或响应里出现规划后用户手动确认，高风险操作前使用
- [ ] **斜杠命令**：定义在 `.claude/commands/`，文件名即命令名，Markdown 格式
- [ ] **CI/CD 集成**：`-p`/`--print`（非交互模式）、`--output-format json`（机器可读）、`--allowedTools`（权限白名单）、`--model`（指定模型）
- [ ] **`/compact` 命令**：压缩上下文，保留关键内容，续跑大型任务
- [ ] **`/memory` 命令**：查看和诊断当前加载的 CLAUDE.md 内容
- [ ] **`context: fork` in SKILL.md**：子智能体运行在隔离上下文中，不污染主会话
- [ ] **`--resume`**：恢复指定会话 ID；`--headless`：无终端交互模式

### Domain 4（20%）— 同等优先

- [ ] **XML 标签**：用于区隔文档、指令、示例（`<document>`, `<examples>`, `<output_format>`）
- [ ] **Prefill 技巧**：在 messages 最后追加 `{"role":"assistant","content":"["}` 强制 Claude 从特定符号开始输出
- [ ] **Few-shot 最佳实践**：2-4 个正例，覆盖边缘情况；正例与反例对比效果更好
- [ ] **`temperature=0`**：结构化提取/确定性场景；高温度用于创意任务
- [ ] **JSON Schema 结构化输出**：`properties`、`required`、`type`、`enum`、`nullable`、`description` 字段
- [ ] **置信度标记**：在 schema 里加 `confidence` 字段（0.0-1.0），低于阈值触发人工审核
- [ ] **Batch API**：50% 成本、最长 24h 处理窗口、不支持多轮工具调用、`custom_id` 追踪
- [ ] **Case Facts 模式**：把关键事实前置到上下文开头，避免 Lost in the Middle
- [ ] **提示链设计**：每步骤单一职责，上一步输出作为下一步输入
- [ ] **系统提示 vs 用户提示**：持久指令（角色、格式、限制）放系统提示；具体任务放用户消息
- [ ] **Claim-Source Mapping**：多源合成时每个主张附上来源，追踪引用置信度

### Domain 5（15%）

- [ ] **上下文窗口大小**：不同 Claude 模型窗口不同（Claude 3.5 Sonnet 是 200K，但不同模型不同，不是所有 Claude 3 都是 200K）
- [ ] **"Lost in the Middle" 效应**：关键信息放开头或结尾，中间位置信息最容易被忽视
- [ ] **上下文优先级顺序**：System prompt > 早期对话 > 最近消息（重要内容放开头）
- [ ] **多智能体交接策略**：传结构化摘要，不传完整消息历史；子智能体独立上下文防串扰
- [ ] **长文档分块处理**：重叠分块（Overlapping Chunks）防止信息在边界被截断
- [ ] **分层摘要**：先摘要各部分，再摘要汇总，压缩比高且保留结构
- [ ] **Manifest 模式**：大型代码库探索时先建立文件/模块索引，再按需读取
- [ ] **Scratchpad 模式**：智能体维护一个工作记事本，跨步骤积累发现
- [ ] **置信度分层**：高置信度直接输出，低置信度暂停等人工确认，防止幻觉传播
- [ ] **Stratified Sampling 验证**：从不同子集采样校验输出质量，而不是全量抽查

---

## 官方明确考查范围

- Claude Agent SDK（多智能体编排、子智能体生命周期）
- Claude Code（CLAUDE.md、Hooks、MCP 集成、Plan Mode、斜杠命令）
- Claude API（工具调用、结构化输出、消息历史管理、Batch API）
- Model Context Protocol（工具与资源接口设计、传输类型）
- 提示工程（JSON Schema、few-shot、结构化提取、温度控制）
- 上下文窗口管理与长文档处理
- CI/CD 集成模式
- 错误处理和人工审核工作流

## 官方明确不考范围

- Claude 模型内部工作原理（神经网络结构等）
- 模型训练、微调、RLHF 流程
- 其他 LLM 提供商的 API 细节
- 非 Anthropic 的 LLM 框架（LangChain、LlamaIndex 等）内部细节
- 底层 Python/JavaScript 语言特性

---

## 考试策略

**所有题目都要作答**——不倒扣，空题视为错误。

**遇到拿不准的题**：先排除明显错误选项（通常能去掉 1-2 个），剩下选项里找"最确定"的那个。如果两个选项都对，选更直接、更官方推荐的那个。

**注意题目问的是"第一步"还是"最佳方案"**——这两个问法答案不同。"第一步"通常是诊断/分析，"最佳方案"是完整解决。

**时间分配**：CCA-F 典型格式是 60 道题 90 分钟，平均每题 90 秒。先做有把握的，标记不确定的回头再看。

---

## 推荐备考资源

| 资源 | 说明 |
|------|------|
| [Anthropic 文档](https://docs.anthropic.com) | 官方权威文档 |
| [Tool Use Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | 工具调用完整指南 |
| [Multi-agent Systems](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/multi-agent) | 多智能体系统 |
| [Claude Code SDK](https://docs.anthropic.com/en/docs/claude-code/sdk) | Agent SDK 文档 |
| [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) | Hooks 配置 |
| [MCP 文档](https://docs.anthropic.com/en/docs/mcp) | Model Context Protocol |
| [Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) | 提示工程指南 |
| [Batch API](https://docs.anthropic.com/en/docs/build-with-claude/batch) | Message Batches API |
