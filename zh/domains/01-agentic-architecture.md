# Domain 1: Agentic Architecture & Orchestration（智能体架构与编排）

> **权重：27%** — 最高权重域，优先掌握  
> 官方文档：[Tool use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | [Multi-agent](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/multi-agent) | [Agent SDK](https://docs.anthropic.com/en/docs/claude-code/sdk)

---

## Task Statement 覆盖范围

| Task | 主题                                            |
| ---- | ----------------------------------------------- |
| 1.1  | 设计并实现用于自主任务执行的智能体循环          |
| 1.2  | 使用协调者-子智能体模式编排多智能体系统         |
| 1.3  | 配置子智能体调用、上下文传递与生成              |
| 1.4  | 实现带有强制执行和交接模式的多步骤工作流        |
| 1.5  | 应用 Agent SDK 钩子进行工具调用拦截和数据规范化 |
| 1.6  | 为复杂工作流设计任务分解策略                    |
| 1.7  | 管理会话状态、恢复与分叉                        |

---

### Task Statement 1.1: Design and implement agentic loops for autonomous task execution

#### Knowledge of:

- The agentic loop lifecycle: sending requests to Claude, inspecting stop_reason ("tool_use" vs "end_turn"), executing requested tools, and returning results for the next iteration
  - 智能体循环的控制信号来自 `stop_reason` 字段，而非模型生成的自然语言文本。`tool_use` 表示模型请求调用工具，循环应继续；`end_turn` 表示本轮任务完成，循环可终止。把 "任务完成" 等文本内容当作终止信号是反模式，因为模型输出内容不稳定。
- How tool results are appended to conversation history so the model can reason about the next action
  - 工具执行后，结果必须以 `tool_result` 格式追加到消息历史（role 为 user）。若跳过这一步，模型在下一轮无法感知工具执行结果，会基于过期状态推理，可能导致重复调用同一工具或逻辑断链。
- The distinction between model-driven decision-making (Claude reasons about which tool to call next based on context) and pre-configured decision trees or tool sequences
  - 模型驱动决策允许 Claude 根据上下文动态选择工具，适合开放性、高歧义任务；预配置决策树把步骤固化在代码中，适合需要确定性合规的场景（如金融操作）。核心权衡是灵活性与可预测性，不同业务场景有不同最优解。

#### Skills in:

- Implementing agentic loop control flow that continues when stop_reason is "tool_use" and terminates when stop_reason is "end_turn"
  - 控制流实现时，应对 `stop_reason` 做完整的分支判断：`tool_use` 分支负责执行工具并回写结果，`end_turn` 分支终止循环。迭代上限（如 `max_iterations`）只能作为防护性上限，不能作为主停止条件。
- Adding tool results to conversation context between iterations so the model can incorporate new information into its reasoning
  - 每轮工具执行后，系统需先将 assistant 的完整响应（含 `tool_use` 块）追加到历史，再将 `tool_result`（必须包含匹配的 `tool_use_id`）以 user role 追加。顺序错误或 ID 不匹配都会导致 API 报错或模型推理出错。
- Avoiding anti-patterns such as parsing natural language signals to determine loop termination, setting arbitrary iteration caps as the primary stopping mechanism, or checking for assistant text content as a completion indicator
  - 解析模型输出文本（如检测 "任务完成"）来控制循环终止不可靠，因为模型措辞每次略有差异。固定轮次上限也不够，因为任务在轮数耗尽时可能尚未完成。`stop_reason` 是唯一的结构化终止依据，应作为主控制信号。

### Task Statement 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

#### Knowledge of:

- Hub-and-spoke architecture where a coordinator agent manages all inter-subagent communication, error handling, and information routing
  - Hub-and-spoke 架构中，协调者负责所有路由、聚合和错误处理；子智能体之间不直接通信。这种设计保证所有信息流都有可追踪的中心节点，便于审计、调试和统一错误策略。
- How subagents operate with isolated context—they do not inherit the coordinator's conversation history automatically
  - 子智能体在独立会话中运行，不自动继承协调者的对话历史。每次调用 Task 工具时，必须将任务目标、范围约束、已有发现、输出格式和来源元数据显式传入子智能体的 prompt。
- The role of the coordinator in task decomposition, delegation, result aggregation, and deciding which subagents to invoke based on query complexity
  - 协调者的职责包括任务分解、动态委托、结果聚合、覆盖缺口识别和错误处理决策。简单查询可以直接响应，无需启动完整多智能体管道；复杂或大规模任务才值得按需启动专门的子智能体。
- Risks of overly narrow task decomposition by the coordinator, leading to incomplete coverage of broad research topics
  - 任务分解粒度过细会导致覆盖缺口（Coverage Gap）：部分子主题因没有对应的子智能体分配而被遗漏。这是多智能体系统中常见的质量缺陷，根因不是单个子任务执行错误，而是任务边界设计遗漏了重要领域。

#### Skills in:

- Designing coordinator agents that analyze query requirements and dynamically select which subagents to invoke rather than always routing through the full pipeline
  - 协调者应根据查询复杂度动态选择处理路径：简单查询直接响应，无需委托子智能体；复杂的多领域研究任务才按需生成专门的搜索、分析、合成子智能体。固定地让所有请求走完整管道会造成不必要的延迟和资源浪费。
- Partitioning research scope across subagents to minimize duplication (e.g., assigning distinct subtopics or source types to each agent)
  - 子智能体的工作范围应互斥且互补，按主题、来源类型、时间范围或证据类型划分，避免重复调查相同领域。重叠范围不仅浪费资源，还会在聚合阶段产生冲突结论，增加协调成本。
- Implementing iterative refinement loops where the coordinator evaluates synthesis output for gaps, re-delegates to search and analysis subagents with targeted queries, and re-invokes synthesis until coverage is sufficient
  - 协调者在完成初次合成后，应评估输出是否存在主题缺口。发现缺口后，应带着具体针对性问题重新委托搜索或分析子智能体，再次触发合成。这个迭代精炼循环是保证研究覆盖质量的核心机制。
- Routing all subagent communication through the coordinator for observability, consistent error handling, and controlled information flow
  - 所有子智能体输出统一汇聚到协调者，便于追踪每条结论的来源、错误的传播路径、以及数据边界。分散的点对点通信会打破可观测性，使调试和治理复杂化。

### Task Statement 1.3: Configure subagent invocation, context passing, and spawning

#### Knowledge of:

- The Task tool as the mechanism for spawning subagents, and the requirement that allowedTools must include "Task" for a coordinator to invoke subagents
  - Task 工具是生成子智能体的唯一机制。协调者的 `allowedTools` 字段中必须包含 `"Task"`，否则无法调用子智能体——即使系统提示中明确说明可以，运行时也会因缺少工具权限而失败。
- That subagent context must be explicitly provided in the prompt—subagents do not automatically inherit parent context or share memory between invocations
  - 子智能体不共享父会话记忆，每次 Task 调用都是一次全新会话。传入的 prompt 应包含完成任务所需的全部上下文：目标、范围约束、已知事实、来源元数据、失败记录（如果有）和期望的输出格式。
- The AgentDefinition configuration including descriptions, system prompts, and tool restrictions for each subagent type
  - AgentDefinition 通过描述、系统提示和 `allowed_tools` 字段为每类子智能体设定角色边界。搜索代理只持有搜索工具，合成代理只持有格式化工具，防止角色漂移和越权调用。
- Fork-based session management for exploring divergent approaches from a shared analysis baseline
  - `fork_session` 从共同分析基线创建独立分支，适合需要并行探索多个互斥方案的场景（如重构策略 A vs B）。在同一上下文中交替推理不同方案容易造成方案之间的上下文污染。

#### Skills in:

- Including complete findings from prior agents directly in the subagent's prompt (e.g., passing web search results and document analysis outputs to the synthesis subagent)
  - 合成子智能体没有搜索子智能体的会话历史访问权限。协调者需要将所有搜索结果、文档分析输出、来源 URL、置信度和已知限制一并传入合成代理的 prompt，确保合成基于完整信息。
- Using structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution
  - 上下文传递时应将内容（摘录、结论）与元数据（来源 URL、文档名、页码、日期）分离，使用结构化格式（如 JSON）而非自然语言段落。摘要后来源链丢失是多智能体合成中常见的可追溯性缺陷。
- Spawning parallel subagents by emitting multiple Task tool calls in a single coordinator response rather than across separate turns
  - 并行执行要求协调者在单次响应中发出多个 Task 工具调用。若跨多轮顺序发起，即使任务逻辑上独立，也会退化为串行执行，总延迟是各子任务的累加。
- Designing coordinator prompts that specify research goals and quality criteria rather than step-by-step procedural instructions, to enable subagent adaptability
  - 协调者的提示应描述研究目标、质量门槛（如每个结论至少有一个来源）和交付格式，而不是逐步骤的操作指令。目标导向的提示使子智能体能根据中间发现自适应地调整搜索策略。

### Task Statement 1.4: Implement multi-step workflows with enforcement and handoff patterns

#### Knowledge of:

- The difference between programmatic enforcement (hooks, prerequisite gates) and prompt-based guidance for workflow ordering
  - 提示指令属于概率约束，模型有非零概率不遵守。Hooks 和前提条件门控属于确定性约束，通过代码层面阻断来保证执行顺序。涉及合规、权限、财务的工作流顺序必须使用程序化控制。
- When deterministic compliance is required (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate
  - 当业务流程要求确定性合规（如退款前必须验证身份），仅靠提示指令会产生非零失败率。必须在代码层面实现前提条件检查，阻断高风险工具调用，直到前置步骤完成并返回有效结果。
- Structured handoff protocols for mid-process escalation that include customer details, root cause analysis, and recommended actions
  - 人工交接协议应包含结构化摘要（客户 ID、问题类型、根因、已尝试动作、推荐操作），而不是原始对话记录。人工接手时通常没有时间阅读完整 transcript，结构化 handoff 能使其立即了解案情并继续处理。

#### Skills in:

- Implementing programmatic prerequisites that block downstream tool calls until prerequisite steps have completed (e.g., blocking process_refund until get_customer has returned a verified customer ID)
  - 程序化前提条件在工具调用拦截钩子（PreToolUse）中实现：检查是否存在已验证的客户 ID，若未满足则返回错误信息并指示需要先完成哪个步骤，阻止下游工具调用继续执行。
- Decomposing multi-concern customer requests into distinct items, then investigating each in parallel using shared context before synthesizing a unified resolution
  - 客户的复合请求（如同时涉及账单、退货、账户）应拆解为独立事项，使用共享上下文并行调查，最后合成统一响应。串行处理会增加响应延迟，也无法利用并行子智能体的效率优势。
- Compiling structured handoff summaries (customer ID, root cause, refund amount, recommended action) when escalating to human agents who lack access to the conversation transcript
  - 结构化 handoff 摘要的标准字段包括：客户 ID、问题类型、根因分析、退款金额、已尝试的动作序列、推荐下一步和紧急程度。当人工接手系统与对话系统不共享历史记录时，这份摘要是唯一的上下文来源。

### Task Statement 1.5: Apply Agent SDK hooks for tool call interception and data normalization

#### Knowledge of:

- Hook patterns (e.g., PostToolUse) that intercept tool results for transformation before the model processes them
  - `PostToolUse` Hook 在工具执行完成后、模型处理结果前触发，可对结果进行格式转换（如时间戳格式统一、状态码语义化、字段名规范化）。这种前处理是确定性的，不依赖模型自行理解不同格式。
- Hook patterns that intercept outgoing tool calls to enforce compliance rules (e.g., blocking refunds above a threshold)
  - `PreToolUse` Hook 在工具调用发出前触发，可检查参数是否违反业务规则（如退款金额超过阈值）并阻断调用。拦截后应返回替代路径（如升级至人工审批），而不只是返回错误信息。
- The distinction between using hooks for deterministic guarantees versus relying on prompt instructions for probabilistic compliance
  - Hook 提供确定性保证，适合违反后会产生财务、合规或安全后果的规则；提示指令提供概率性引导，适合风格偏好、最佳实践等低风险场景。两者不可互换，选择标准是失败成本。

#### Skills in:

- Implementing PostToolUse hooks to normalize heterogeneous data formats (Unix timestamps, ISO 8601, numeric status codes) from different MCP tools before the agent processes them
  - 不同 MCP 工具可能返回异构格式（Unix 时间戳 vs ISO 8601、数字状态码 vs 字符串），`PostToolUse` Hook 在这些数据进入模型上下文前统一格式，消除模型因格式差异产生的误解风险。
- Implementing tool call interception hooks that block policy-violating actions (e.g., refunds exceeding $500) and redirect to alternative workflows (e.g., human escalation)
  - 工具调用被拦截后，Hook 应返回包含替代路径的说明（如 "金额超过自动审批限额，已转入人工审批流程"），使模型能够向用户解释下一步，而不是停留在错误状态。
- Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance
  - 凡是违反后会产生财务、隐私或合规后果的业务规则，应通过 Hook 或后端校验来保障，而不只是写在系统提示中。提示中的规则依赖模型遵守，存在失败概率；代码层面的检查在工具执行路径中强制执行。

### Task Statement 1.6: Design task decomposition strategies for complex workflows

#### Knowledge of:

- When to use fixed sequential pipelines (prompt chaining) versus dynamic adaptive decomposition based on intermediate findings
  - 固定顺序的提示链适合步骤已知、每步输入输出稳定的工作流（如：先安全审查 → 再性能审查 → 最后可读性审查）；动态分解适合开放调查任务，因为下一步依赖当前发现，无法提前固定。
- Prompt chaining patterns that break reviews into sequential steps (e.g., analyze each file individually, then run a cross-file integration pass)
  - 将大型代码审查拆分为两个 pass：第一步逐文件进行本地分析，专注于单文件内的问题；第二步独立执行跨文件集成分析，专注于接口契约、数据流和模块间依赖。一次性分析所有文件会导致注意力稀释，降低每个文件的审查质量。
- The value of adaptive investigation plans that generate subtasks based on what is discovered at each step
  - 动态分解的价值在于：任务的中间发现（如发现关键依赖、未预期的代码结构）可以触发新的子任务生成。遗留系统补测、事故根因调查、未知代码库探索都属于这类场景，提前固化所有步骤反而会错失关键路径。

#### Skills in:

- Selecting task decomposition patterns appropriate to the workflow: prompt chaining for predictable multi-aspect reviews, dynamic decomposition for open-ended investigation tasks
  - 任务分解方式的选择应匹配工作流的不确定性：步骤可预测的任务使用提示链保证执行顺序；步骤依赖中间发现的任务使用动态分解以保持适应性。错误地对开放任务使用固定流水线会导致漏掉重要发现。
- Splitting large code reviews into per-file local analysis passes plus a separate cross-file integration pass to avoid attention dilution
  - 两阶段代码审查架构：第一阶段每个文件独立分析，捕捉本地 bug；第二阶段专门检查跨文件问题（接口兼容性、状态共享、权限传播、循环依赖）。两阶段必须独立执行，混合会导致两种维度的问题都处理不深入。
- Decomposing open-ended tasks (e.g., "add comprehensive tests to a legacy codebase") by first mapping structure, identifying high-impact areas, then creating a prioritized plan that adapts as dependencies are discovered
  - 开放式任务的推荐分解流程：先用 Grep/Glob 构建代码库结构地图，再识别高影响区域（核心模块、高频调用路径），然后根据依赖和风险动态生成优先级排序的子任务计划，随着执行中发现新依赖持续调整。

### Task Statement 1.7: Manage session state, resumption, and forking

#### Knowledge of:

- Named session resumption using --resume <session-name> to continue a specific prior conversation
  - `--resume <session-name>` 恢复指定名称的会话，保留原有对话历史和上下文。适合跨工作周期延续仍然有效的调查，避免重新探索已分析过的内容。前提是旧上下文中的工具结果尚未过期。
- fork_session for creating independent branches from a shared analysis baseline to explore divergent approaches
  - `fork_session` 从共同的分析基线创建独立会话分支，适合需要并行比较不同实现方案的场景（如重构策略 A vs 策略 B、测试框架 X vs 框架 Y）。独立分支防止不同方案的推理上下文互相干扰。
- The importance of informing the agent about changes to previously analyzed files when resuming sessions after code modifications
  - 恢复会话后，模型仍持有旧的工具调用结果和文件内容缓存。如果代码在两次会话之间发生了修改，必须明确告知具体变更文件，要求重新分析受影响路径，否则模型会基于过期信息推理，得出错误结论。
- Why starting a new session with a structured summary is more reliable than resuming with stale tool results
  - 当旧会话中大量工具结果已过期（如大规模重构、依赖升级后），硬恢复可能比开新会话风险更高，因为模型会将过期状态与新信息混合推理。新会话 + 结构化摘要能提供更干净、可靠的起点。

#### Skills in:

- Using --resume with session names to continue named investigation sessions across work sessions
  - 跨工作周期的长时调查可使用命名会话保留背景，但恢复前应评估会话中的工具结果是否仍有效。会话名称本身不保证信息的时效性，需要主动确认上下文是否反映当前代码库状态。
- Using fork_session to create parallel exploration branches (e.g., comparing two testing strategies or refactoring approaches from a shared codebase analysis)
  - 方案 A/B 并行对比时，应从同一分析基线 fork 出两个独立分支，分别探索各方案，最终由协调者或开发者比较输出结果。在同一会话中交替推理两种方案会污染上下文，使每种方案的评估都不纯粹。
- Choosing between session resumption (when prior context is mostly valid) and starting fresh with injected summaries (when prior tool results are stale)
  - 恢复 vs 新建的判断标准是旧上下文的事实有效性：文件修改较小（仅影响少数已知文件）时，恢复并告知变更更高效；文件大规模修改或依赖升级后，新会话 + 结构化摘要能避免过期信息影响后续推理。
- Informing a resumed session about specific file changes for targeted re-analysis rather than requiring full re-exploration
  - 恢复会话后提供具体变更清单（修改了哪些文件、变更了哪些接口），使模型可以进行针对性的局部重分析，而不是重新探索整个代码库。这样既保留了原有调查成果，又确保新信息得到正确处理。

---

## Task 1.1：智能体循环（Agentic Loop）

### 这题怎么考

Task 1.1 的核心不是“会调用工具”，而是能把 Claude API 返回值当作一个结构化控制协议来实现。考试会区分两类实现：一类是由 `stop_reason`、`tool_use_id`、`tool_result` 驱动的可靠循环；另一类是解析自然语言、固定循环次数或按文本内容猜测是否完成的脆弱实现。

实战中，agentic loop 是所有自主执行系统的最小闭环：模型负责基于上下文决定下一步，宿主程序负责执行工具并把结果正确写回历史。只要少追加一次 assistant 的 `tool_use` 响应、漏掉一个并发 `tool_use`、或者工具结果没有匹配 `tool_use_id`，循环就会失去状态一致性。备考时要把“模型推理”和“程序控制”分清：模型决定做什么，程序用结构化字段保证循环可恢复、可审计、可终止。

### 先把概念说清楚

智能体循环是让 Claude 自主完成多步骤任务的核心机制，通过 `stop_reason` 驱动循环：

```
发送请求 → Claude 响应 → 检查 stop_reason
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                  ▼
         "tool_use"        "end_turn"      "max_tokens" / 其他
              │                 │                  │
    执行工具，追加结果     返回最终响应        处理截断/暂停
              │
         返回循环顶部
```

### stop_reason 完整枚举

| 值                | 含义                               | 循环动作                                 |
| ----------------- | ---------------------------------- | ---------------------------------------- |
| `"tool_use"`      | Claude 请求调用工具                | **继续**：执行工具 → 追加结果 → 再次调用 |
| `"end_turn"`      | 任务正常完成                       | **终止**：返回最终文本响应               |
| `"max_tokens"`    | 达到 `max_tokens` 限制，响应被截断 | **处理**：增大 `max_tokens` 或分批处理   |
| `"stop_sequence"` | 遇到了预设的停止序列               | **终止**：按业务逻辑处理                 |
| `"pause_turn"`    | 工具调用被外部暂停（流式场景）     | **恢复**：解决暂停原因后继续             |

> 这里容易考混：`max_tokens` 被截断时 `stop_reason` 是 `"max_tokens"` 而非 `"tool_use"`，不能简单地继续循环。

### 正确实现

```python
messages = [{"role": "user", "content": user_query}]

while True:
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=4096,
        tools=tools,
        messages=messages
    )

    if response.stop_reason == "end_turn":
        # 正常完成，提取文本响应
        final_text = next(
            (b.text for b in response.content if hasattr(b, "text")), ""
        )
        break

    if response.stop_reason == "max_tokens":
        # 响应被截断，需要处理（增大 max_tokens 或重设计）
        raise ValueError("响应被截断，请增大 max_tokens")

    if response.stop_reason == "tool_use":
        tool_use_blocks = [b for b in response.content if b.type == "tool_use"]

        # Step 1：将 Claude 的响应（包含 tool_use 块）追加到历史
        messages.append({"role": "assistant", "content": response.content})

        # Step 2：执行每个工具调用
        tool_results = []
        for tool_call in tool_use_blocks:
            result = execute_tool(tool_call.name, tool_call.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": tool_call.id,  # 必须匹配 tool_use 的 id
                "content": result             # 字符串或内容块列表
            })

        # Step 3：将工具结果追加到历史（role 为 user）
        messages.append({"role": "user", "content": tool_results})
```

### tool_result 内容格式

```python
# 简单字符串结果
{
    "type": "tool_result",
    "tool_use_id": "toolu_abc123",
    "content": "订单状态：已发货，预计3天到达"
}

# 结构化内容块（支持图片）
{
    "type": "tool_result",
    "tool_use_id": "toolu_abc123",
    "content": [
        {"type": "text", "text": "以下是截图："},
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "..."}}
    ]
}

# 工具执行失败时（设置 is_error: true）
{
    "type": "tool_result",
    "tool_use_id": "toolu_abc123",
    "content": "数据库连接超时",
    "is_error": True  # Claude 会据此调整后续行为
}
```

### 反模式

| 反模式                                     | 问题                                                                              |
| ------------------------------------------ | --------------------------------------------------------------------------------- |
| 检测文本中的 "任务完成" 来终止循环         | Claude 输出不可预测，会漏判或误判                                                 |
| `for i in range(20):` 作为**主要**停止机制 | 任意上限不可靠；`max_iterations` 只能作为**安全备用**，主控制必须是 `stop_reason` |
| 不将工具结果追加到消息历史                 | Claude 无法感知工具结果，会反复重试同一工具                                       |
| 忘记将 assistant 响应追加到历史            | 工具结果没有对应的 tool_use，API 返回 400 错误                                    |
| 多个 tool_use 只处理第一个                 | Claude 可以在一次响应中发起多个工具调用，必须全部处理                             |

### 模型驱动 vs 预配置决策树

| 方式             | 描述                                  | 适用场景                      |
| ---------------- | ------------------------------------- | ----------------------------- |
| **模型驱动**     | Claude 自主决定调用哪个工具、何时停止 | 高歧义性任务、需要灵活判断    |
| **预配置决策树** | 固定工具调用顺序（程序化控制流）      | 需要确定性合规的金融/医疗流程 |

### 何时不使用多智能体架构

多智能体增加复杂性，以下情况单智能体更优：

- 任务可在单个上下文窗口内完成
- 步骤必须严格串行且互相依赖
- 延迟敏感（每个子智能体调用都会增加调度、上下文传递和结果聚合开销）
- 调试困难度不可接受

延迟判断不要只看“能否并行”。并行子智能体可以降低独立子任务的总耗时，但如果任务本身很小、必须串行、或用户在交互链路上等待，启动多个子智能体的固定开销可能超过收益。

---

## Task 1.2：协调者-子智能体模式（Hub-and-Spoke）

### 这题怎么考

Task 1.2 考的是多智能体系统的编排边界。协调者不是简单地“把所有子智能体都跑一遍”，而是根据查询复杂度决定是否拆分、拆给谁、传什么上下文、如何聚合冲突结果，以及发现覆盖缺口后如何重新委托。官方强调 hub-and-spoke，是因为这种架构把通信、错误处理、可观测性和信息流控制集中在协调者，避免子智能体之间形成不可追踪的隐式依赖。

实战中，多智能体适合研究范围大、上下文超限、工具权限需要隔离、或者需要并行专业化分析的任务。它的主要风险不是“跑得不够多”，而是拆分粒度错误：拆得太窄会漏领域，拆得重复会浪费上下文并产生冲突结论。合格设计应让协调者先定义覆盖地图，再让子智能体在互补范围内工作，最后由协调者做覆盖检查、冲突消解和必要的二次委托。

### Hub-and-Spoke 架构

```
                    ┌─────────────┐
                    │  协调智能体  │  ← 统一管理通信、错误、路由
                    └──────┬──────┘
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
      ┌──────────┐  ┌──────────┐  ┌──────────┐
      │ 搜索子   │  │ 分析子   │  │ 合成子   │
      │ 智能体   │  │ 智能体   │  │ 智能体   │
      └──────────┘  └──────────┘  └──────────┘
```

**协调智能体核心职责：**

- **任务分解**：将复杂任务拆分为可并行的子任务
- **委托**：动态决定调用哪些子智能体、传递什么上下文
- **结果聚合**：合并子智能体输出，处理冲突
- **错误处理**：统一管理失败情况（重试 / 使用部分结果 / 升级）
- **覆盖检查**：识别任务覆盖缺口并重新委托

### 容易踩坑的地方

**子智能体不会自动继承协调智能体的上下文！**  
子智能体在完全独立的会话中运行，上下文必须显式传递到子智能体的提示中。

**任务分解过窄**会导致研究覆盖不完整，这类题经常用看似合理的拆分来诱导：

- 错：将"AI对创意产业的影响"只分解为"数字艺术、平面设计、摄影"（遗漏音乐、写作、电影等）
- 对：分解时覆盖完整范围，协调者负责识别并填补缺口

### 多智能体 vs 单智能体决策

```
任务特征评估：
├─ 可并行化 + 超出单个上下文窗口 → 多智能体
├─ 需要专业化子智能体（不同工具集） → 多智能体
├─ 单一线性流程 + 上下文够用 → 单智能体
└─ 高延迟敏感 → 优先单智能体
```

---

## Task 1.3：子智能体生成与上下文传递

### 这题怎么考

Task 1.3 关注“子智能体如何被正确生成”。考试会看你是否知道 `Task` 本身是生成子智能体的工具，因此协调者的 `allowedTools` 必须包含 `"Task"`；也会看你是否知道子智能体默认不会继承父会话历史。换句话说，系统提示里说“你可以派发子任务”不等于运行时真的有权限，父会话里曾经发现的信息也不会自动出现在子智能体上下文里。

实战中，上下文传递质量决定多智能体质量。给子智能体的 prompt 需要包含目标、范围、已知事实、来源元数据、输出格式和质量标准；给得太少会导致重复探索，给得太乱会丢失归因。对于需要并行的任务，协调者应在同一轮响应中发出多个 `Task` 调用，而不是多轮串行发起。对于需要比较不同方案的工作，应从共同分析基线 fork 出独立分支，避免方案之间互相污染。

### Task 工具机制

协调智能体想要调用子智能体，**`allowedTools` 中必须包含 `"Task"`**：

```python
coordinator = AgentDefinition(
    system_prompt="你是研究协调智能体...",
    allowed_tools=["Task", "search_web"],  # 必须包含 "Task"
)
```

如果 `allowedTools` 中没有 `"Task"`，协调者将无法生成子智能体——即使系统提示中说可以。

### 并行生成子智能体

在**单次响应**中发出多个 `Task` 调用实现并行执行（而非多轮串行）：

```python
# 并行：协调者单次响应中包含多个 Task 调用
response_content = [
    Task(prompt="搜索主题A的最新论文", context=shared_context),
    Task(prompt="搜索主题B的统计数据", context=shared_context),
    Task(prompt="搜索主题C的案例研究", context=shared_context),
]
# 串行：多轮对话中依次发起（效率低，延迟高）
```

### 结构化上下文传递（保留来源出处）

传递给子智能体的上下文应包含充分信息，使其无需再次请求：

```json
{
  "task": "分析AI对音乐创作产业的影响",
  "scope": "重点关注独立音乐人，时间范围2022-2024",
  "already_found": ["工具A已找到：..."],
  "findings": [
    {
      "claim": "AI使艺术家工作效率提高了40%",
      "evidence": "根据2024年创意AI报告...",
      "source_url": "https://...",
      "document_name": "Creative_AI_Report_2024.pdf",
      "page": 23
    }
  ]
}
```

### 子智能体的系统提示设计

子智能体的系统提示应该专注且受限：

```python
# 专注的子智能体提示
search_agent = AgentDefinition(
    system_prompt="""你是网络搜索专家。
    你的唯一任务是搜索并提取相关信息。
    不要分析或综合，只返回原始数据和来源。
    返回格式：JSON，包含 title, url, snippet, date""",
    allowed_tools=["search_web", "fetch_url"],  # 仅必要工具
)
```

---

## Task 1.4：多步骤工作流强制执行

### 这题怎么考

Task 1.4 的关键是区分“提示建议”和“程序强制”。如果流程顺序只是最佳实践，例如“先总结再回答”，提示通常够用；如果流程顺序涉及金融、身份、权限、隐私或合规，例如“验证客户后才能退款”，就必须用 hook、状态门控或后端校验来阻止违规工具调用。官方措辞中的 “non-zero failure rate” 很重要：只要靠提示，就不能提供确定性合规。

实战中，多步骤工作流还要处理复杂请求和人工交接。用户一次请求可能包含多个 concern，系统应拆成独立事项并行调查，再合成统一解决方案。需要升级给人工时，不能只说“请人工处理”，而要提供结构化 handoff：客户 ID、问题类型、根因、已尝试动作、退款金额、推荐操作和紧急程度。这样即使人工看不到完整对话，也能继续处理。

### 程序化强制执行 vs 提示引导

| 方式                                      | 可靠性                     | 适用场景                           |
| ----------------------------------------- | -------------------------- | ---------------------------------- |
| **程序化强制执行**（Hooks、前提条件门控） | **确定性**（100%）         | 金融操作前的身份验证、强制合规流程 |
| **提示引导**                              | **概率性**（有非零失败率） | 最佳实践建议、偏好设置、风格规范   |

> 这句话要记牢：当需要确定性合规时（如退款前必须验证身份），**仅靠提示指令有非零失败率**，必须使用程序化强制执行。题目里出现“非零失败率”时，通常就是在提示你不要只靠 prompt。

### 程序化前提条件实现

```python
def intercept_tool_call(self, tool_name: str, tool_input: dict):
    """
    工具调用拦截钩子（PreToolUse）
    在 lookup_order 或 process_refund 之前强制先调用 get_customer
    """
    protected_tools = ["lookup_order", "process_refund"]

    if tool_name in protected_tools:
        if not self.verified_customer_id:
            # 返回阻止消息，Claude 会理解需要先验证身份
            return {
                "error": "必须先调用 get_customer 验证客户身份",
                "required_action": "请先调用 get_customer 工具"
            }
    return None  # None = 允许继续
```

### 结构化交接协议（人工升级）

升级给人工时，摘要应包含完整上下文：

```json
{
  "escalation_summary": {
    "customer_id": "CUST-1234",
    "issue_type": "refund_dispute",
    "amount_disputed": 299.99,
    "root_cause": "客户声称未收到商品，但系统显示已签收",
    "attempted_actions": ["查询了订单状态（已签收）", "核实了配送地址（匹配）"],
    "recommended_action": "联系配送服务商核查签收记录",
    "urgency": "high"
  }
}
```

---

## Task 1.5：Agent SDK Hooks

### 这题怎么考

Task 1.5 把 hook 放在 agent 架构里的两个关键位置：工具调用发出前和工具结果进入模型前。`PreToolUse` 适合拦截违规动作，例如超过阈值的退款、未验证身份的账户操作、危险命令或越权数据访问；`PostToolUse` 适合把不同 MCP 工具返回的时间戳、状态码、金额单位、字段名和错误格式统一成模型更容易理解的 schema。

实战中，hook 的价值是把确定性规则从 prompt 中移到代码层。模型可以理解“不要退款超过 500 美元”，但不能保证每次都遵守；hook 可以保证违规调用不会执行，并能返回替代流程，例如升级人工、请求额外验证或进入审批。备考时要能判断：如果规则违反会造成财务、合规、安全或数据质量后果，就应该使用 hook 或后端校验，而不是只写在系统提示里。

### 钩子类型与触发时机

| 钩子类型      | 触发时机                   | 常见用途                           |
| ------------- | -------------------------- | ---------------------------------- |
| `PreToolUse`  | 工具调用发出**前**         | 合规规则执行、参数验证、访问控制   |
| `PostToolUse` | 工具执行**后**、模型处理前 | 数据格式规范化、结果过滤、日志记录 |

### PostToolUse：数据规范化

解决不同工具返回格式不一致的问题（确定性保证，不依赖模型自行处理）：

```python
def post_tool_use_hook(tool_name: str, tool_result: dict) -> dict:
    """
    在工具结果进入模型上下文前进行规范化
    确定性：不依赖模型自行处理格式差异
    """
    if tool_name == "get_customer":
        # Unix时间戳 → ISO 8601
        if "created_at" in tool_result and isinstance(tool_result["created_at"], int):
            tool_result["created_at"] = datetime.fromtimestamp(
                tool_result["created_at"]
            ).isoformat()

        # 数字状态码 → 语义字符串
        status_map = {1: "active", 2: "suspended", 3: "closed"}
        if isinstance(tool_result.get("status"), int):
            tool_result["status"] = status_map.get(tool_result["status"], "unknown")

        # 货币金额：分 → 元
        if "balance_cents" in tool_result:
            tool_result["balance_yuan"] = tool_result.pop("balance_cents") / 100

    return tool_result
```

### PreToolUse：访问控制与合规

```python
def pre_tool_use_hook(tool_name: str, tool_input: dict) -> dict:
    """
    工具调用前的合规检查
    返回 {"blocked": True, ...} 则阻止调用；{"blocked": False} 则允许
    """
    if tool_name == "process_refund":
        amount = tool_input.get("amount", 0)
        if amount > 500:
            return {
                "blocked": True,
                "redirect_to": "escalate_to_human",
                "reason": f"退款金额 ¥{amount} 超过自动审批限额 ¥500"
            }

        if not self.customer_verified:
            return {
                "blocked": True,
                "reason": "客户身份未验证，拒绝退款操作"
            }

    return {"blocked": False}  # 允许继续
```

### Hooks vs 提示引导对比

| 方面     | Hooks                | 提示引导             |
| -------- | -------------------- | -------------------- |
| 可靠性   | 确定性（100%）       | 概率性（非零失败率） |
| 适用场景 | 合规、验证、格式转换 | 风格、偏好、建议     |
| 实现位置 | 代码层（拦截执行）   | 系统提示             |

---

## Task 1.6：任务分解策略

### 这题怎么考

Task 1.6 考的是如何根据任务不确定性选择分解方式。固定顺序的 prompt chaining 适合步骤已知、每一步输入输出稳定的工作，例如逐文件审查后做跨文件集成检查；动态自适应分解适合开放式任务，例如给遗留代码库补测试、排查未知故障、研究大主题。错误做法是把开放调查硬编码成固定流水线，或者把确定流程交给模型临场自由发挥。

实战中，好的任务分解先降低认知负载，再保持覆盖完整。大型代码审查应先做局部分析，避免单次上下文塞入太多文件导致注意力稀释；随后单独做跨文件 pass，专门检查接口契约、数据流、状态共享和集成风险。开放任务则应先映射结构、识别高影响区域、建立优先级，再随着依赖和风险发现动态调整计划。

### 两种分解方式

| 方式                   | 描述                                 | 适用场景                                             |
| ---------------------- | ------------------------------------ | ---------------------------------------------------- |
| **提示链（固定顺序）** | 预定义的顺序步骤，每步输出是下步输入 | 已知流程的多方面审查（如代码审查：安全→性能→可读性） |
| **动态自适应分解**     | 基于中间发现动态生成下一步子任务     | 开放式调查（遗留代码库探索、未知领域研究）           |

**开放式任务分解流程：**

```
1. 映射结构（Glob/Grep 遍历，了解代码库概貌）
   ↓
2. 识别高影响区域（基于第一步发现，动态调整优先级）
   ↓
3. 深入分析（按优先级分配子智能体）
   ↓
4. 覆盖检查（协调者验证是否有遗漏领域）
```

**大型代码审查避免注意力稀释：**

- **第一阶段**：逐文件本地分析（每次只关注一个文件的局部问题）
- **第二阶段**：独立的跨文件整合分析（专注于文件间数据流、接口兼容性）
- 两个阶段必须独立运行，不能混合进同一个上下文

---

## Task 1.7：会话状态管理

### 这题怎么考

Task 1.7 关注长任务中的上下文新鲜度。`--resume <session-name>` 适合继续一个事实基本未变的调查；`fork_session` 适合从共同基线探索不同方案；新会话加结构化摘要适合旧会话里很多工具结果已经过期的情况。考试常见陷阱是把“恢复上下文”当作总是更好，实际上旧工具结果可能比没有上下文更危险。

实战中，恢复会话后必须主动告诉 agent 哪些文件、配置或外部状态已经变化，让它做针对性重查。不要指望模型自己意识到旧分析已经失效，也不要让它为了确认全部状态重新探索整个代码库。判断标准很直接：旧上下文仍然可信就恢复并告知差异；旧上下文大量过期就开新会话，把已完成工作、关键发现、待处理问题和当前文件状态用结构化摘要注入。

### 三种会话操作

| 操作                  | 命令/机制                        | 使用场景                           |
| --------------------- | -------------------------------- | ---------------------------------- |
| **恢复命名会话**      | `claude --resume <session-name>` | 延续先前对话，上下文仍有效         |
| **会话分叉**          | `fork_session`                   | 从同一基线探索不同方案（A/B 对比） |
| **新会话 + 摘要注入** | 新建会话 + 结构化摘要            | 先前工具结果已过期、代码大幅修改   |

### 选择决策树

```
先前上下文仍有效？
├─ 是 → --resume 恢复会话
│       └─ 有文件发生变更？
│           ├─ 否 → 直接继续
│           └─ 是 → 告知具体变更文件，请求针对性重新分析
│                   （不要让模型自行发现变更——效率低且可能遗漏）
└─ 否（代码已大幅修改 / 工具结果过期）
    └─ 新会话 + 注入结构化摘要
        摘要应包含：已完成工作、关键发现、待处理问题

需要对比两种方案？→ fork_session 创建并行分支
```

### 摘要注入格式

当需要开新会话延续先前工作时，注入结构化摘要而非完整历史：

```python
structured_summary = """
## 先前工作摘要
- 已完成：分析了 auth、payment 两个模块
- 关键发现：JWT 验证缺少过期检查（auth.py:45）
- 待处理：notification 模块尚未分析
- 已知依赖：payment → notification（事件驱动）
"""
messages = [
    {"role": "user", "content": structured_summary + "\n请继续分析 notification 模块"}
]
```

---

## 讲透这一域：如何真正理解智能体架构与编排

Domain 1 是整套 CCA-F 中权重最高的部分，因为它考的不是“知道 Claude 能调用工具”，而是你是否能把 Claude 放进一个可靠的软件系统里。很多备考资料会把 agentic architecture 讲成“模型 + 工具 + 循环”，但考试关注的是更细的工程判断：什么时候让模型自主决策，什么时候必须用程序化逻辑限制它；什么时候拆成多个子智能体，什么时候单智能体更稳；什么时候恢复旧会话，什么时候旧上下文反而是风险。这些判断决定了生产系统能不能稳定运行。

理解 Domain 1 时，先把它看成三层架构。第一层是单智能体循环，也就是 Task 1.1：Claude 观察上下文，决定是否调用工具，系统执行工具，再把结果返回给 Claude，直到 `stop_reason` 表示结束。第二层是多智能体编排，也就是 Task 1.2 和 1.3：协调者负责拆任务、分配子智能体、传上下文、合成结果。第三层是生产级可靠性，也就是 Task 1.4 到 1.7：用 hooks 和 gates 做确定性约束，用任务分解避免注意力稀释，用 session resume/fork 管理长任务状态。这三层之间不是独立知识点，而是同一个系统从最小闭环逐步扩展到复杂生产工作流的过程。

### 一、智能体循环不是 while 循环，而是状态机

很多人第一次实现 agentic loop 时，会写一个 `while True`，然后看到模型输出“任务完成”就退出。这是考试明确反对的做法。真正的智能体循环应当由 API 返回的结构化状态驱动，尤其是 `stop_reason`。`tool_use` 表示模型需要外部动作；`end_turn` 表示模型完成本轮任务；`max_tokens` 表示输出被截断，不应该当作正常完成；`pause_turn` 和 `stop_sequence` 也需要按场景处理。把这些状态当成状态机，比把它们当成普通字段更容易答对题。

在生产系统里，状态机思维意味着每一种停止原因都要有明确处理策略。`tool_use` 不是“继续生成”，而是“暂停模型、执行工具、构造 tool_result、追加历史、再次请求模型”。`end_turn` 不是“文本里说完成”，而是 API 确认模型本轮结束。`max_tokens` 不是“再循环一次就好”，而是说明设计上可能需要增加 token、拆分任务、压缩上下文或分阶段处理。考试常见干扰项会把 `max_tokens` 当作继续调用工具的信号，或者把 assistant 文本内容当作完成信号，这些都违背官方要求。

工具结果追加历史也是状态机的一部分。Claude 发出的 `tool_use` 块有唯一 ID，系统返回的 `tool_result` 必须引用这个 ID。缺少 assistant 的 tool_use 消息、漏掉 tool_use_id、把工具结果放错 role，都会让对话历史在结构上不一致。即使 API 没有立刻报错，模型也无法正确理解“这个结果对应哪个动作”。备考时要形成一个固定顺序：先保存 assistant 响应，再执行所有 tool_use，再用 user role 回传 tool_result，再进入下一轮。

### 二、模型驱动决策和程序化决策的边界

Domain 1 反复考一个核心权衡：让 Claude 自主判断，还是由代码固定流程。模型驱动决策适合开放问题，例如研究主题拆分、选择查询关键词、判断下一步需要哪种证据。程序化决策适合合规、权限、财务、身份验证这类不能出错的步骤。题目里如果出现“退款前必须验证身份”“超过金额必须升级”“不能在未授权时访问数据”，正确方向通常不是“在 prompt 里提醒 Claude”，而是用 hook、gate、后端校验或工具拦截实现确定性约束。

这个边界也决定多智能体系统怎么设计。协调者可以让模型判断“是否需要搜索代理、文档代理、合成代理”，但不应该让模型绕过关键审批。子智能体可以自主选择搜索词，但不应该拥有不属于角色的高风险工具。系统可以让模型决定是否还需要补充证据，但覆盖检查和最终交付条件最好结构化，例如要求每个 claim 至少有来源、每个缺口必须标注、每个失败必须有 error context。

### 三、协调者不是路由器，而是负责质量的总编辑

Hub-and-spoke 架构里，协调者经常被误解成“把任务转发给子智能体”。这太浅了。协调者的真正职责包括：判断问题复杂度、决定是否需要多智能体、拆分任务范围、避免重复工作、传递上下文、合并结果、发现覆盖缺口、处理错误、决定是否继续迭代。换句话说，协调者是研究负责人、项目经理和质量门禁的结合体。

考试特别喜欢考“过窄任务分解”。例如用户要求研究“AI 对创意产业的影响”，协调者只拆成“数字艺术、平面设计、摄影”，看起来合理，但漏掉音乐、写作、电影、游戏、广告等重要领域。这个错误不是某个子智能体能力差，而是协调者的任务覆盖设计失败。正确做法是先定义研究边界，再按互斥维度拆分，例如按行业、来源类型、时间范围、利益相关者或证据类别拆。拆完后还要有覆盖检查：哪些领域已经有证据，哪些只是推测，哪些完全缺失，需要重新委托。

协调者还要控制信息流。子智能体之间不直接通信，不只是为了架构整洁，更是为了可观测性和错误控制。所有子结果经协调者汇总，系统才能知道某个结论来自哪个子任务、哪个工具、哪个来源。否则当最终报告出现错误时，很难追溯是搜索不全、文档解析错、合成误读，还是来源冲突没有处理。

### 四、子智能体上下文必须显式传递

“子智能体不会自动继承父上下文”是 Domain 1 很常见的考法。很多错误设计会让协调者对搜索代理说“继续刚才的研究”，但子智能体没有“刚才”。它只看到本次 Task prompt。正确委托应包含任务目标、范围边界、已有发现、禁止重复的内容、期望输出格式、来源保留要求、质量标准和失败上报格式。

例如，合成代理不应该只收到“写一份总结”。它应该收到结构化输入：每个搜索结果的标题、URL、发布时间、相关摘录、可信度、已知限制；每个文档分析结果的 claim-source mapping；以及协调者要求的报告结构。这样做的目的不是让 prompt 更长，而是让子智能体具备完成任务所需的完整工作包。上下文传递越结构化，最终合成越可追溯。

并行 spawn 也要理解清楚。官方说多个 Task tool calls 应在单个协调者响应中发出，才是真正并行。如果协调者一轮调用搜索 A，下一轮调用搜索 B，再下一轮调用搜索 C，本质是串行。并行适合互相独立的子主题，串行适合后一步依赖前一步结果的流程。考试里看到“降低研究系统延迟”或“并行调查互不依赖子主题”，优先选择单轮多个 Task 调用；但如果题目强调“低延迟交互”“任务很小”或“步骤强依赖”，优先单智能体或串行流程。

### 五、handoff 不是聊天记录转交，而是结构化交接

Task 1.4 的 handoff 很容易被低估。用户升级到人工、流程中途转交、子系统无法继续处理时，最差的做法是把整段 transcript 交给人工。人工没有时间重新阅读所有上下文，而且 transcript 里可能混有无关工具输出。结构化 handoff 应该包含：客户 ID、问题摘要、已验证事实、根因分析、已尝试动作、失败原因、金额或订单等关键字段、推荐下一步、风险提示。

对于客户支持场景，handoff 的质量直接影响用户体验。一个好的 handoff 能让人工接手后第一句话就是“我看到您订单 ORD-123 的退款因为超过自动额度需要人工审批，我已经拿到验证信息”，而不是“请问您遇到了什么问题”。考试会把“升级”与“重新开始”混在选项里，正确答案通常强调保留结构化上下文，而不是简单转人工。

### 六、hooks 是确定性护栏，不是提示增强

Agent SDK hooks 在 Domain 1 中承担两个角色：结果规范化和动作拦截。结果规范化处理的是工具返回不一致，例如一个 MCP 返回 Unix timestamp，另一个返回 ISO 日期，一个返回数字状态码，另一个返回字符串。把这些差异交给模型理解会增加误判概率；用 `PostToolUse` 统一格式更稳。

动作拦截处理的是高风险行为。例如退款超过 500 美元、删除生产数据、访问敏感客户记录、执行危险 shell 命令。这里不能依赖系统提示“请不要”。如果业务规则要求保证执行，就必须在 hook 或后端层阻断。优秀设计还会在阻断后返回替代路径，例如“金额超过阈值，已转人工审批”，而不是只返回错误。这样模型能继续对用户解释下一步，而不是停在失败状态。

### 七、任务分解要匹配不确定性

固定流水线和动态分解的选择，是架构判断题的常见核心。固定流水线适合步骤已知、顺序稳定的任务，比如文档提取：先分类、再抽取、再校验、再输出。动态分解适合开放调查，比如“给遗留系统补全面测试”“研究一个新市场”“找出线上故障根因”。这些任务一开始不知道全部路径，必须先探索结构，再根据发现调整下一步。

大型代码审查的正确模式是“局部分析 + 跨文件整合”。如果把几十个文件和所有要求一次塞给 Claude，模型会注意力稀释，既漏掉局部 bug，也看不清跨文件契约。逐文件 pass 负责本地问题，integration pass 负责数据流、接口兼容、权限传播、状态一致性。考试中看到“大 PR 审查质量差、发现互相矛盾、漏掉跨文件问题”，通常要选择多 pass 架构。

### 八、session resume 和 fork 的风险管理

恢复会话不是越多越好。`--resume` 的优势是保留上下文，风险是旧工具结果可能过期。代码已经改了、依赖更新了、外部系统状态变了，模型如果继续相信旧结果，会做出错误判断。因此恢复时必须告诉它哪些文件发生变化，让它定向重查。变化很大时，开新会话并注入结构化摘要反而更安全。

`fork_session` 的价值是并行比较方案。比如同一个重构目标，可以 fork 出“最小变更方案”和“架构重写方案”；同一个测试目标，可以 fork 出“单元测试优先”和“集成测试优先”。每个分支从同一分析基线出发，但独立探索，最后由协调者或人工比较。不要在同一个会话里让模型同时维护多个互斥方案，那会污染上下文。

### 九、答题时的快速判断框架

遇到 Domain 1 题目，可以按五步判断。第一，看任务是否需要工具循环：如果需要外部信息和多步动作，答案应围绕 `stop_reason`、tool_use、tool_result。第二，看是否需要多智能体：如果任务可并行、范围大、需要专门工具或上下文超限，考虑协调者-子智能体；如果任务线性、上下文够用、延迟敏感，单智能体更合适。第三，看是否有合规或财务约束：有就程序化 enforcement，不靠 prompt。第四，看是否存在上下文传递：子智能体必须显式传。第五，看是否存在长任务状态：判断 resume、fork、新会话摘要哪个更安全。

最容易错的选项通常有共同特征：把自然语言当控制信号、把 prompt 当硬约束、让子智能体自动共享上下文、所有问题都走完整多智能体流水线、错误时直接终止全局流程、恢复旧会话但不告知文件变化。只要能识别这些反模式，Domain 1 的题目会稳定很多。

最后要把 Domain 1 当成“控制权分配”来复习。Claude 适合处理语义判断、开放探索、动态分解和结果综合；代码适合处理状态机、权限、合规、金额阈值、会话恢复策略和错误边界。优秀架构不是让模型完全自由，也不是把模型限制成死板脚本，而是在每个决策点明确谁负责、失败如何表达、上下文如何传递、结果如何验证。考试题中的最佳答案通常都体现这种分工：模型负责推理，系统负责保证。

复习时不要只背术语，要能把一个场景画成流程图：用户请求进入协调者，协调者判断复杂度，必要时生成子智能体，子智能体带着显式上下文调用受限工具，工具结果被规范化后返回，协调者检查覆盖和错误，再决定继续、合成或升级。只要能稳定画出这条链路，就能看出题目里哪个环节缺失。

我会这样记：先看谁掌控流程，再看谁保留状态，最后看失败后有没有路可走。

---

## 临考速查

1. **stop_reason 完整集**：`end_turn`（完成）、`tool_use`（调工具）、`max_tokens`（截断）、`stop_sequence`、`pause_turn`
2. **循环终止**：始终检查 `stop_reason == "end_turn"`，不解析文本
3. **工具结果格式**：`type: "tool_result"` + `tool_use_id`（必须匹配）+ `content`
4. **is_error 标志**：工具执行失败时设置 `"is_error": True`，Claude 会据此调整行为
5. **多 tool_use**：单次响应可包含多个 tool_use 块，**必须全部处理**
6. **子智能体上下文**：不自动继承，必须显式传递
7. **Task 工具**：`allowedTools` 必须包含 `"Task"` 才能生成子智能体
8. **并行执行**：单次响应中发出多个 Task 调用（非多轮串行）
9. **程序化 vs 提示**：关键合规必须用程序化约束（Hooks）
10. **PostToolUse**：工具结果规范化（格式转换确定性 > 让模型自行处理）
11. **任务分解**：开放式任务用动态分解，已知流程用提示链
12. **会话恢复**：告知变更文件，不要让模型自行发现
