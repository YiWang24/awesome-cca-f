# Domain 2: Tool Design & MCP Integration（工具设计与MCP集成）

> **权重：18%**  
> 官方文档：[Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | [MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)

---

## Task Statement 覆盖范围

| Task | 主题                                           |
| ---- | ---------------------------------------------- |
| 2.1  | 设计具有清晰描述和边界的有效工具接口           |
| 2.2  | 为 MCP 工具实现结构化错误响应                  |
| 2.3  | 在智能体间合理分配工具并配置工具选择           |
| 2.4  | 将 MCP 服务器集成到 Claude Code 和智能体工作流 |
| 2.5  | 有效选择和应用内置工具                         |

---

### Task Statement 2.1: Design effective tool interfaces with clear descriptions and boundaries

#### Knowledge of:

- Tool descriptions as the primary mechanism LLMs use for tool selection; minimal descriptions lead to unreliable selection among similar tools
  - 工具描述是模型进行工具选择的主要依据。描述模糊或多个工具描述相近时，模型无法可靠区分，会导致频繁误选。工具名称、描述、schema 共同构成路由界面。
- The importance of including input formats, example queries, edge cases, and boundary explanations in tool descriptions
  - 工具描述应包含四个要素：接受的输入格式、典型使用场景、边界情况处理、与相似工具的区分说明。缺少任何一项都会扩大模型误选的空间。
- How ambiguous or overlapping tool descriptions cause misrouting (e.g., analyze_content vs analyze_document with near-identical descriptions)
  - 当两个工具的名称或描述存在功能重叠时，模型会将它们视为近似同义词。通过重命名（如 `analyze_content` → `extract_web_results`）和更新描述来显式区分职责，是解决误路由的首选手段。
- The impact of system prompt wording on tool selection: keyword-sensitive instructions can create unintended tool associations
  - 系统提示中的特定关键词可能建立隐式工具关联（如 "处理 content 时调用 analyze_content"），覆盖本来写得清晰的工具描述。排查误路由时应同时检查工具描述和系统提示两个层面。

#### Skills in:

- Writing tool descriptions that clearly differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives
  - 一个合格的工具描述应同时说明：唯一职责、输入格式与示例、返回字段、必须调用的业务场景，以及应该使用哪个相似工具替代的边界说明。
- Renaming tools and updating descriptions to eliminate functional overlap (e.g., renaming analyze_content to extract_web_results with a web-specific description)
  - 当误路由持续存在时，重命名工具往往比在提示中添加更多说明更有效。工具名称应直接反映具体业务动作（如 `refund_order` 而非 `process`），减少歧义。
- Splitting generic tools into purpose-specific tools with defined input/output contracts (e.g., splitting a generic analyze_document into extract_data_points, summarize_content, and verify_claim_against_source)
  - 将一个带有 `analysis_type` 参数的泛化工具拆分为多个专用工具（如 `extract_data_points`、`summarize_content`、`verify_claim_against_source`），可以让每个工具的选择条件更清晰，降低模型在选择工具时的决策复杂度。
- Reviewing system prompts for keyword-sensitive instructions that might override well-written tool descriptions
  - 工具描述清晰但仍出现误选时，应检查系统提示是否包含 keyword-sensitive 指令（如将某个词与特定工具强关联），这类指令可能覆盖工具描述建立的路由逻辑。

### Task Statement 2.2: Implement structured error responses for MCP tools

#### Knowledge of:

- The MCP isError flag pattern for communicating tool failures back to the agent
  - MCP 工具失败时应返回带 `isError: true` 的结构化响应，而非仅返回含糊的错误文本。结构化错误使模型能够根据错误类别做出具体的恢复决策。
- The distinction between transient errors (timeouts, service unavailability), validation errors (invalid input), business errors (policy violations), and permission errors
  - 错误类别决定恢复策略：`transient`（超时、服务不可用）通常可重试；`validation`（输入格式错误）、`permission`（权限不足）、`business`（业务规则拒绝）不应盲目重试，需要不同的处理路径。
- Why uniform error responses (generic "Operation failed") prevent the agent from making appropriate recovery decisions
  - 返回 "Operation failed" 这类通用错误，使模型无法判断应该重试、修改参数、升级人工还是告知用户。结构化错误的价值在于为每种失败类型提供明确的恢复路径。
- The difference between retryable and non-retryable errors, and how returning structured metadata prevents wasted retry attempts
  - `isRetryable: false` 可防止模型对永久性失败（如权限错误、业务规则拒绝）发起无效重试；`isRetryable: true` 配合退避策略可使瞬时错误在本地恢复，减少不必要的上报。

#### Skills in:

- Returning structured error metadata including errorCategory (transient/validation/permission), isRetryable boolean, and human-readable descriptions
  - 结构化错误响应应包含：`errorCategory`（机器可读分类）、`isRetryable`（布尔值）、`description`（用户可读说明）、`attemptedAction`（失败时正在执行的操作）和 `partialResults`（超时等场景下的部分结果）。
- Including retriable: false flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately
  - 业务规则拒绝的错误响应应包含面向用户的解释（如 "退款金额超过自动审批限额"），使模型能够向用户给出合规、清晰的回复，而不是只返回技术性错误码。
- Implementing local error recovery within subagents for transient failures, propagating to the coordinator only errors that cannot be resolved locally along with partial results and what was attempted
  - 瞬时错误应优先在子智能体内部通过退避重试本地恢复。只有重试耗尽或无法本地解决时，才上报协调者，同时附上已尝试的动作序列和部分结果，避免协调者重复无效委托。
- Distinguishing between access failures (needing retry decisions) and valid empty results (representing successful queries with no matches)
  - 查询成功但无匹配结果，应返回 `success: true, count: 0, results: []`，而非将空结果吞掉或返回错误标志。混淆两者会导致模型将正常的"无数据"情况误判为需要恢复的失败。

### Task Statement 2.3: Distribute tools appropriately across agents and configure tool choice

#### Knowledge of:

- The principle that giving an agent access to too many tools (e.g., 18 instead of 4-5) degrades tool selection reliability by increasing decision complexity
  - 工具数量增加会提高模型在选择时的决策复杂度，进而降低选择可靠性。每个子智能体只应持有完成其角色任务所必需的工具（通常 4-5 个），而非所有可用工具。
- Why agents with tools outside their specialization tend to misuse them (e.g., a synthesis agent attempting web searches)
  - 当合成代理持有搜索工具时，它可能绕过研究代理直接查资料，导致来源筛选标准不统一、引用格式混乱、协调者失去对信息流的可观测性。按角色限制工具集可防止这类角色漂移。
- Scoped tool access: giving agents only the tools needed for their role, with limited cross-role tools for specific high-frequency needs
  - 跨角色工具应窄化为仅覆盖高频且低风险的需求（如给合成代理一个 `verify_fact` 工具，而非完整的 `fetch_url`）。复杂的跨角色数据需求应通过协调者重新委托来处理。
- tool_choice configuration options: "auto", "any", and forced tool selection ({"type": "tool", "name": "..."})
  - `tool_choice` 有三种模式：`auto`（模型自主决定是否调用工具及调用哪个）、`any`（必须调用某个工具，但哪个由模型决定）、`{"type":"tool","name":"..."}` 强制指定特定工具。三种模式的强制程度依次递增。

#### Skills in:

- Restricting each subagent's tool set to those relevant to its role, preventing cross-specialization misuse
  - 为每类子智能体通过 `allowed_tools` 字段配置工具白名单，确保其只能执行角色内的操作。搜索代理持有搜索和抓取工具，文档代理持有加载和抽取工具，防止越权调用。
- Replacing generic tools with constrained alternatives (e.g., replacing fetch_url with load_document that validates document URLs)
  - 用带语义边界的专用工具替换泛化工具（如用 `load_document(document_url)` 替换 `fetch_url(url)`，并限制只允许文档库域名），可降低误用风险，同时使工具的适用范围更易于模型理解。
- Providing scoped cross-role tools for high-frequency needs (e.g., a verify_fact tool for the synthesis agent) while routing complex cases through the coordinator
  - 合成代理可持有窄化的核验工具（如 `verify_fact`）以满足高频轻量核验需求；需要重新搜索或深度补充证据的复杂情况应回传协调者重新委托，保持信息流的可控性。
- Using tool_choice forced selection to ensure a specific tool is called first (e.g., forcing extract_metadata before enrichment tools), then processing subsequent steps in follow-up turns
  - 当工作流的第一步必须固定时（如必须先执行 `extract_metadata` 才能进行 enrich 操作），使用 forced tool selection 锁定第一步；第二步再切换到 `auto` 或 `any`，让模型根据 metadata 结果做后续决策。
- Setting tool_choice: "any" to guarantee the model calls a tool rather than returning conversational text
  - `tool_choice: "any"` 保证模型必须调用工具（从可用集合中选一个），不允许返回纯文本响应。适用于文档类型未知但必须结构化提取的场景，防止模型用自然语言直接回答而绕过工具调用。

### Task Statement 2.4: Integrate MCP servers into Claude Code and agent workflows

#### Knowledge of:

- MCP server scoping: project-level (.mcp.json) for shared team tooling vs user-level (~/.claude.json) for personal/experimental servers
  - 项目级 `.mcp.json` 适合团队共享的 MCP 工具，应提交到版本控制，保证所有成员和 CI 使用相同的服务器配置。用户级 `~/.claude.json` 的 `mcpServers` 字段适合个人实验，不应作为团队依赖。
- Environment variable expansion in .mcp.json (e.g., ${GITHUB_TOKEN}) for credential management without committing secrets
  - 认证凭据（API Key、Token、数据库连接串）必须通过环境变量展开（如 `${GITHUB_TOKEN}`），不得硬编码写入 `.mcp.json`。硬编码凭据随仓库提交是安全事故。
- That tools from all configured MCP servers are discovered at connection time and available simultaneously to the agent
  - 所有配置的 MCP 服务器工具在连接时会同时加入工具池。多个服务器并存时，工具命名和描述必须避免冲突，否则模型无法可靠区分来自不同服务器的同名或相似工具。
- MCP resources as a mechanism for exposing content catalogs (e.g., issue summaries, documentation hierarchies, database schemas) to reduce exploratory tool calls
  - MCP Resources 以只读内容目录的形式暴露数据（如 issue 摘要列表、文档树、数据库 schema），使模型可以在不发起探索性工具调用的前提下了解可用数据范围，减少不必要的 API 调用次数。

#### Skills in:

- Configuring shared MCP servers in project-scoped .mcp.json with environment variable expansion for authentication tokens
  - 团队共享的 MCP 配置应写入项目根目录的 `.mcp.json` 并提交版本控制；认证值通过 `${ENV_VAR}` 在运行时从环境变量读取，使配置文件可以安全共享而不暴露凭据。
- Configuring personal/experimental MCP servers in user-scoped ~/.claude.json
  - 仍在实验阶段或只属于个人的 MCP 服务器应写入用户级配置，不应影响项目配置。若写入项目级，则要求所有团队成员都在相同环境变量中配置对应凭据，造成不必要的环境依赖。
- Enhancing MCP tool descriptions to explain capabilities and outputs in detail, preventing the agent from preferring built-in tools (like Grep) over more capable MCP tools
  - 若 MCP 工具描述过于简短（如只写 "search docs"），模型可能优先选择内置的 Grep 或 Bash。描述应说明工具的独特能力、权限范围、返回字段和适用场景，使模型能够准确判断何时应选择 MCP 工具而非内置工具。
- Choosing existing community MCP servers over custom implementations for standard integrations (e.g., Jira), reserving custom servers for team-specific workflows
  - GitHub、Jira、Slack、PostgreSQL 等标准集成优先使用成熟的社区 MCP 服务器；仅当涉及内部私有系统、团队专属工作流或特定权限模型时，才值得自研 MCP 服务器。
- Exposing content catalogs as MCP resources to give agents visibility into available data without requiring exploratory tool calls
  - 将 issue 列表、文档层级、数据库 schema、API 目录等目录型信息暴露为 MCP Resources，可在模型调用工具前提供数据全貌，提高后续工具调用的目标性，减少盲目探索。

### Task Statement 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively

#### Knowledge of:

- Grep for content search (searching file contents for patterns like function names, error messages, or import statements)
  - Grep 搜索文件内容中的模式，适合查找函数调用位置、错误消息来源、import 语句、变量引用等以内容为目标的搜索。
- Glob for file path pattern matching (finding files by name or extension patterns)
  - Glob 按文件路径和名称模式匹配文件，适合查找所有 `*.test.tsx` 文件、`src/api/**/*.py` 模块、特定目录下所有配置文件等以路径为目标的搜索。
- Read/Write for full file operations; Edit for targeted modifications using unique text matching
  - Edit 使用唯一文本锚点进行局部修改，适合精确定位代码段；Read/Write 适合读取完整文件或覆盖写入整个文件。当目标文本在文件中不唯一时，Edit 会失败。
- When Edit fails due to non-unique text matches, using Read + Write as a fallback for reliable file modifications
  - Edit 因锚点不唯一失败时，不应反复尝试模糊锚点，而应使用 Read 读取完整文件内容，确认目标位置后再用 Write 或带更精确上下文的 Edit 完成修改。

#### Skills in:

- Selecting Grep for searching code content across a codebase (e.g., finding all callers of a function, locating error messages)
  - 追踪函数调用者、错误来源或 import 路径时，Grep 是正确工具，可以直接在文件内容中搜索，避免无目的读取大量文件消耗上下文。
- Selecting Glob for finding files matching naming patterns (e.g., \*_/_.test.tsx)
  - 需要按文件命名规范定位特定文件（如所有测试文件、所有迁移脚本）时，Glob 的路径模式匹配比逐目录 Read 更高效。
- Using Read to load full file contents followed by Write when Edit cannot find unique anchor text
  - 当编辑目标文本在文件中出现多次（非唯一锚点）时，Edit 无法确定应修改哪一处，必须先 Read 全文件内容，确认正确位置后再执行修改。
- Building codebase understanding incrementally: starting with Grep to find entry points, then using Read to follow imports and trace flows, rather than reading all files upfront
  - 大型代码库的高效探索策略：先用 Grep 定位入口点（main、handler、router），再 Read 入口文件沿 import 链追踪，不要一开始全量读取所有文件——这会用低价值内容占满上下文窗口。
- Tracing function usage across wrapper modules by first identifying all exported names, then searching for each name across the codebase
  - Wrapper 模块和 barrel export 会使函数以别名导出，仅搜索原始函数名会遗漏实际调用点。正确做法是先 Read wrapper 文件列出所有 exported names，再对每个名称分别执行 Grep。

---

## Task 2.1：工具接口设计

### 工具描述是工具选择的首要机制

> 工具选择错误时，**第一步改工具描述**，不是加 few-shot 或改路由逻辑。

**优质工具描述的四要素：**

| 要素         | 说明                     | 示例                                  |
| ------------ | ------------------------ | ------------------------------------- |
| **输入格式** | 接受什么类型/格式的输入  | "customer_id 格式 CUST-XXXX"          |
| **使用场景** | 什么情况下调用此工具     | "处理退款前必须先调用"                |
| **边缘情况** | 特殊边界条件             | "不存在的 ID 返回 null 而非报错"      |
| **边界说明** | 何时用此工具 vs 相似工具 | "不用于订单查询，请使用 lookup_order" |

```python
# 糟糕的描述（工具间界限模糊，导致选择混乱）
{"name": "get_customer",  "description": "获取客户信息"}
{"name": "lookup_order",  "description": "获取订单信息"}

# 优质描述（清晰区分，防止混用）
{
    "name": "get_customer",
    "description": (
        "通过客户ID或邮箱验证并获取客户账户信息。"
        "输入：customer_id（格式 CUST-XXXX）或 email 地址。"
        "输出：客户姓名、账户状态、注册日期、退款资格。"
        "使用场景：验证客户身份。在处理订单或退款前必须先调用此工具。"
        "注意：不用于查询订单详情，订单查询请使用 lookup_order。"
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "customer_id": {
                "type": "string",
                "description": "客户ID，格式 CUST-XXXX。与 email 二选一。"
            },
            "email": {
                "type": "string",
                "description": "客户邮箱地址。与 customer_id 二选一。"
            }
        }
        # 注意：customer_id 和 email 都不在 required 中，但至少提供一个
    }
}
```

### 工具 Input Schema 设计

```python
# 嵌套对象
{
    "name": "create_order",
    "input_schema": {
        "type": "object",
        "properties": {
            "customer_id": {"type": "string"},
            "shipping_address": {
                "type": "object",           # 嵌套对象
                "properties": {
                    "street": {"type": "string"},
                    "city":   {"type": "string"},
                    "zip":    {"type": "string"}
                },
                "required": ["street", "city", "zip"]
            },
            "items": {
                "type": "array",            # 数组
                "items": {
                    "type": "object",
                    "properties": {
                        "product_id": {"type": "string"},
                        "quantity":   {"type": "integer", "minimum": 1}
                    },
                    "required": ["product_id", "quantity"]
                },
                "minItems": 1
            }
        },
        "required": ["customer_id", "items"]
    }
}

# enum 约束（限定有效值，防止无效输入）
{
    "priority": {
        "type": "string",
        "enum": ["low", "medium", "high", "urgent"],
        "description": "处理优先级"
    }
}

# nullable 字段（可能不存在的信息）
{
    "discount_code": {
        "type": ["string", "null"],
        "description": "优惠码，无优惠码时为 null"
    }
}
```

### 将泛化工具拆分为专用工具

```python
# 泛化工具（analysis_type 参数导致行为不可预测）
analyze_document(doc_id, analysis_type="summary"|"data"|"verify")

# 专用工具（职责单一，描述精准）
extract_data_points(doc_id)                    # 仅提取数据点
summarize_content(doc_id)                      # 仅生成摘要
verify_claim_against_source(claim, doc_id)     # 仅验证声明
```

### 这题怎么考

#### 工具误选的排查顺序

Domain 2.1 的核心不是“工具越多越强”，而是让模型在相似工具之间能稳定做出选择。考试题通常会给出两个功能接近的工具，例如 `analyze_content` 和 `analyze_document`，然后描述模型经常选错。优先判断点是：工具名是否过于抽象、描述是否没有写输入/输出/边界、系统提示是否用某些关键词把模型引向错误工具。

**推荐排查顺序：**

1. 先看工具描述是否明确说明“什么时候用”和“什么时候不用”。
2. 再看相似工具之间是否存在功能重叠，必要时重命名或拆分。
3. 然后检查 input schema 是否足够约束输入格式。
4. 最后检查 system prompt 是否出现 keyword-sensitive 指令，例如“遇到 content 都调用 analyze_content”，这类提示会覆盖本来写得不错的工具描述。

```python
# 系统提示制造了错误关联
system_prompt = "When the user asks about content, use analyze_content."

# 系统提示保留决策边界
system_prompt = (
    "Use extract_web_results only for crawled web search results. "
    "Use summarize_document only for uploaded documents or document IDs. "
    "If the source type is unclear, ask for clarification or inspect metadata first."
)
```

放到项目里看，工具设计不是只写 JSON schema。名称、description、schema、系统提示共同构成路由界面。只要其中一个层面给出模糊或相互矛盾的信号，模型就可能误选工具。

---

## Task 2.2：结构化错误响应

### MCP 错误字段设计

| 字段              | 类型       | 说明                                         |
| ----------------- | ---------- | -------------------------------------------- |
| `isError`         | boolean    | 是否为错误（`true`）vs 有效空结果（`false`） |
| `errorCategory`   | string     | 错误类别（决定是否重试）                     |
| `isRetryable`     | boolean    | 是否可以重试                                 |
| `description`     | string     | 人类可读的错误描述                           |
| `attemptedAction` | string     | 失败时正在执行的操作                         |
| `partialResults`  | array/null | 超时等情况下的部分结果                       |

### 错误类型与可重试性

| 错误类型         | `errorCategory` | `isRetryable` | 示例                       |
| ---------------- | --------------- | ------------- | -------------------------- |
| **瞬时错误**     | `"transient"`   | `true`        | 超时、服务不可用、网络抖动 |
| **验证错误**     | `"validation"`  | `false`       | 无效输入格式、参数缺失     |
| **业务逻辑错误** | `"business"`    | `false`       | 退款超过限额、余额不足     |
| **权限错误**     | `"permission"`  | `false`       | 无访问权限、token 过期     |
| **资源不存在**   | `"not_found"`   | `false`       | 订单不存在、用户不存在     |

```json
// 瞬时错误（可重试）
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "数据库连接超时，请在5秒后重试",
  "attemptedAction": "查询订单 #ORD-12345",
  "partialResults": null
}

// 权限错误（不可重试）
{
  "isError": true,
  "errorCategory": "permission",
  "isRetryable": false,
  "description": "无权访问此订单，该订单属于不同账户",
  "attemptedAction": "查询订单 #ORD-99999"
}
```

### 区分"访问失败"与"有效空结果"

```python
# 混淆两者（模型无法判断是失败还是"真的没有数据"）
def search_orders(customer_id: str):
    try:
        results = db.query(customer_id)
        return results  # 空列表和失败看起来一样
    except Exception:
        return []       # 将失败伪装成空结果

# 明确区分（模型可以做出正确的后续决策）
def search_orders(customer_id: str):
    try:
        results = db.query(customer_id)
        return {
            "success": True,
            "orders": results,      # 空列表 = 查询成功但无匹配
            "count": len(results)
        }
    except TimeoutError:
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "description": "查询超时，请稍后重试"
        }
    except PermissionError:
        return {
            "isError": True,
            "errorCategory": "permission",
            "isRetryable": False,
            "description": "无权查询此账户的订单"
        }
```

### 这题怎么考

Domain 2.2 的判断重点是：工具错误必须给模型足够信息做恢复决策。只返回“失败”没有用；合格答案应说明错误类型、是否可重试、用户可读解释、已尝试动作和部分结果。考试里要特别区分“访问失败”和“成功但无结果”，前者需要恢复策略，后者是有效业务结果。

#### 子智能体本地恢复与错误上报

官方强调 transient failure 应先在 subagent 内部局部恢复，不要把所有失败都直接抛给 coordinator。原因是 coordinator 负责全局计划，如果每个短暂超时都升级，会造成无意义的重规划和重复调用。

| 情况                       | 子智能体动作                               | 是否上报 coordinator |
| -------------------------- | ------------------------------------------ | -------------------- |
| API timeout、503、网络抖动 | 按退避策略重试，保留 attemptedAction       | 重试耗尽后上报       |
| 输入格式错误               | 不重试，返回 validation error              | 直接上报或请求修正   |
| 业务规则拒绝               | 不重试，返回 business error 和面向用户解释 | 直接上报             |
| 权限不足                   | 不盲目重试，说明缺少权限/凭据              | 直接上报             |
| 查询成功但无结果           | 返回 success/count=0                       | 不作为错误上报       |

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": false,
  "description": "连续 3 次调用搜索服务超时，已停止本地重试",
  "attemptedAction": "search_web(query='MCP Jira integration')",
  "attempts": 3,
  "partialResults": [
    { "title": "MCP server overview", "url": "https://example.com/mcp" }
  ]
}
```

**考试判断方法**：如果选项只是返回 `"Operation failed"`，通常是错的；如果选项区分 `errorCategory`、`isRetryable`、业务解释、部分结果和已尝试动作，通常更符合官方要求。

---

## Task 2.3：工具分配与 tool_choice

### 工具数量与可靠性

给一个智能体配置过多工具会增加决策复杂性，降低工具选择可靠性。官方例子强调的是 **18 个工具 vs 4-5 个工具**：4-5 个与角色高度相关的工具通常比 18 个混杂工具更可靠。

**按角色范围化分配工具：**

| 子智能体角色     | 工具配置                                   | 工具数量 |
| ---------------- | ------------------------------------------ | -------- |
| 网络搜索子智能体 | `[search_web, fetch_url]`                  | 2        |
| 文档分析子智能体 | `[load_document, extract_data, summarize]` | 3        |
| 报告生成子智能体 | `[generate_report, format_citations]`      | 2        |
| 协调智能体       | `[Task, search_web]`（子智能体工具）       | 2-3      |

### 这题怎么考

Domain 2.3 的核心是“最小必要工具集 + 正确 tool_choice”。考试题通常会用“给了太多工具导致误用”“合成代理绕过搜索代理”“必须先调用某个工具”来考你是否能限制角色边界，并选择 `auto`、`any` 或 forced tool selection。

#### 受限工具集与跨角色工具

每个 subagent 应只拿到完成角色任务必需的工具。一个 synthesis agent 如果也拿到 `search_web` 和 `fetch_url`，它可能绕过 research agent 自己查资料，导致证据来源、引用格式和任务分工失控。

| 角色                  | 应给的工具                                              | 不应默认给的工具              | 原因                                                   |
| --------------------- | ------------------------------------------------------- | ----------------------------- | ------------------------------------------------------ |
| Research agent        | `search_web`, `fetch_url`, `load_document`              | `generate_final_report`       | 负责收集证据，不负责最终综合                           |
| Synthesis agent       | `summarize_findings`, `format_citations`, `verify_fact` | `search_web`, `fetch_url`     | 负责综合，可做轻量事实核验，但复杂检索应回 coordinator |
| Data extraction agent | `load_document`, `extract_data_points`                  | `send_email`, `create_ticket` | 只做结构化抽取，不执行外部副作用                       |
| Coordinator           | `delegate_task`, `route_to_agent`                       | 低层细粒度业务工具            | 负责调度，不应亲自执行所有细节                         |

**受限替代工具的思想**：不要给通用 `fetch_url(url)`，而是给 `load_document(document_url)` 并校验 URL 是否来自允许的文档域名。工具越贴近业务语义，模型越不容易误用。

```python
# 过宽：任何 URL 都能抓，容易抓错来源或绕过流程
fetch_url(url: str)

# 受限：只允许加载文档库 URL，并返回标准化元数据
load_document(document_url: str)  # validates docs.example.com/*
```

### `tool_choice` 配置选项

```python
# auto（默认）：模型自行决定是否调用工具、调用哪个
# 适用：通用对话，允许直接文本回复
tool_choice = {"type": "auto"}

# any：模型必须调用至少一个工具（不允许纯文本响应）
# 适用：必须结构化输出，不能返回对话文本
tool_choice = {"type": "any"}

# tool（强制指定）：模型必须调用这个特定工具
# 适用：已知必须先运行特定步骤
tool_choice = {"type": "tool", "name": "extract_metadata"}
```

**`tool_choice: "any"` 的典型使用场景：**

```python
# 文档类型未知，但必须结构化提取（不允许模型说"我不知道这是什么类型"）
response = client.messages.create(
    tools=[invoice_extractor, receipt_extractor, contract_extractor],
    tool_choice={"type": "any"},   # 保证调用其中一个，而非返回纯文本
    messages=messages
)
```

**forced tool selection 的典型流程：**

```python
# 第一步必须先抽取 metadata，再进入后续 enrichment
first_turn = client.messages.create(
    tools=[extract_metadata],
    tool_choice={"type": "tool", "name": "extract_metadata"},
    messages=messages
)

# 第二步再根据 metadata 决定调用 enrich_customer、enrich_order 或 summarize_document
```

**考试判断方法**：当题目要求“必须先调用某个工具”时，选择 forced tool selection；当题目要求“必须调用某个工具，但具体哪个由模型判断”时，选择 `tool_choice: "any"`；当允许模型直接回答时，选择 `auto`。

### tool_use 内容块结构

理解 API 响应中的 `tool_use` 内容块结构：

```python
# Claude 响应中的 tool_use 块
{
    "type": "tool_use",
    "id": "toolu_01XFDUDYJgAACTvnkyfe3yCN",  # 唯一ID，tool_result 中必须引用
    "name": "get_customer",
    "input": {
        "customer_id": "CUST-1234"
    }
}

# 访问方式
for block in response.content:
    if block.type == "tool_use":
        print(block.id)     # "toolu_01XFD..."
        print(block.name)   # "get_customer"
        print(block.input)  # {"customer_id": "CUST-1234"}
```

---

## Task 2.4：MCP 服务器集成

### MCP 配置作用域

| 作用域     | 配置文件位置                          | 提交到版本控制 | 用途              |
| ---------- | ------------------------------------- | -------------- | ----------------- |
| **项目级** | `.mcp.json`（项目根目录）             | 应提交         | 团队共享工具      |
| **用户级** | `~/.claude.json` 的 `mcpServers` 字段 | 不提交         | 个人/实验性服务器 |

### MCP 传输类型

| 传输类型                     | 说明                           | 适用场景                     |
| ---------------------------- | ------------------------------ | ---------------------------- |
| **stdio**                    | 通过标准输入输出通信，本地进程 | 本地工具（文件系统、数据库） |
| **SSE (Server-Sent Events)** | HTTP 长连接流式通信            | 远程服务、云端 API           |

```json
// .mcp.json 完整示例
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server", // stdio 传输
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}" // 环境变量，不硬编码密钥
      }
    },
    "database": {
      "command": "postgres-mcp",
      "args": ["--connection-string", "${DB_URL}"],
      "env": {
        "DB_URL": "${DATABASE_URL}"
      }
    },
    "remote-api": {
      "url": "https://api.example.com/mcp", // SSE 传输（用 url 而非 command）
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

### MCP 工具 vs MCP 资源

| 类型         | 用途                     | Claude 如何使用    |
| ------------ | ------------------------ | ------------------ |
| **MCP 工具** | 执行操作（有副作用）     | 智能体在循环中调用 |
| **MCP 资源** | 暴露内容目录（只读数据） | 注入上下文或供浏览 |

**MCP 资源的价值**：让智能体在不调用探索性工具的情况下了解可用数据，减少不必要的 API 调用。

示例资源：

- `issues://` — 暴露所有 Issue 的摘要列表
- `schema://` — 暴露数据库结构
- `docs://` — 暴露文档层次结构

### 社区 MCP vs 自定义 MCP

| 情况                                        | 选择                    |
| ------------------------------------------- | ----------------------- |
| 标准集成（GitHub、Jira、Slack、PostgreSQL） | 优先使用社区 MCP 服务器 |
| 内部私有系统、团队特定工作流                | 构建自定义 MCP 服务器   |

### 这题怎么考

Domain 2.4 的实战重点是 MCP 的可复现配置和工具可发现性。团队共享 server 应放项目级 `.mcp.json`，个人实验放用户级配置；凭据必须通过环境变量注入；资源目录应通过 MCP resources 暴露，减少模型盲目探索。

#### Claude Code 集成检查清单

Domain 2.4 的考试重点是“配置位置、凭据管理、工具发现、资源暴露”。项目级 `.mcp.json` 适合团队共享，并且应该通过 `${ENV_VAR}` 展开密钥；用户级 `~/.claude.json` 适合个人或实验性服务器，不能要求队友也安装。

**配置检查：**

1. 团队共享 MCP server 是否写在项目根目录 `.mcp.json`。
2. token、API key、数据库连接串是否通过环境变量展开，而不是硬编码。
3. 多个 MCP server 同时连接后，工具名和描述是否会冲突。
4. MCP 工具描述是否解释了能力、输入、输出和优势，避免模型退回内置 Grep。
5. 是否把 issue 摘要、文档层级、数据库 schema 这类目录型信息暴露为 MCP resources。

```json
{
  "mcpServers": {
    "jira": {
      "command": "jira-mcp-server",
      "env": {
        "JIRA_TOKEN": "${JIRA_TOKEN}",
        "JIRA_HOST": "${JIRA_HOST}"
      }
    }
  }
}
```

**工具描述不足的风险**：如果 MCP 工具只写 `"search docs"`，模型可能选择内置 Grep 去扫本地文件；如果描述写清楚“搜索远程知识库、返回标题/URL/更新时间/权限过滤后的摘要”，模型更容易选择 MCP 工具。

**社区 vs 自定义**：Jira、GitHub、Slack、PostgreSQL 等标准集成优先用成熟社区 MCP server；只有内部审批流、私有数据模型、团队专属操作需要自定义 MCP server。

---

## Task 2.5：内置工具选择

### 工具速查表

| 工具      | 用途                            | 典型场景                                     |
| --------- | ------------------------------- | -------------------------------------------- |
| **Grep**  | 搜索**文件内容**中的模式        | 查找函数调用者、找 TODO 注释、找 import 语句 |
| **Glob**  | 按**文件路径/名称模式**查找文件 | 找所有 `*.test.tsx`、找 `src/api/**/*.py`    |
| **Read**  | 读取完整文件内容                | 查看特定文件                                 |
| **Write** | 写入/覆盖整个文件               | 创建新文件或完全替换                         |
| **Edit**  | 基于**唯一文本锚点**的局部修改  | 精确修改特定代码段（锚点不唯一时失败）       |
| **Bash**  | 执行 shell 命令                 | 运行测试、安装依赖、git 操作                 |

> 这组最容易混：Grep 查内容，Glob 查路径。

### Grep vs Glob 决策

```
需要找函数/变量名在哪些文件中出现？ → Grep（搜索内容）
需要找符合特定命名模式的文件？      → Glob（按路径）
需要找所有测试文件？                → Glob（**/*.test.ts）
需要找哪里调用了某个 API？          → Grep（搜索调用语句）
需要找 src/components/ 下所有文件？  → Glob（目录模式）
需要找含有特定错误码的文件？         → Grep（搜索内容关键词）
```

### Edit 失败时的回退策略

```
Edit 工具（文本锚点在文件中不唯一）→ 失败
           ↓
    Read 读取完整文件内容
           ↓
    Write 写入修改后的完整内容
```

### 渐进式代码库理解策略

```
1. Grep（查找入口点：main、app、handler、router 等关键字）
   ↓
2. Read（读取入口文件，追踪 import 链）
   ↓
3. Grep（搜索关键函数的调用者，理解数据流）
   ↓
4. 按需 Read 相关文件（不要一开始就全量读取代码库）
```

### 这题怎么考

Domain 2.5 的考试重点是选择正确内置工具，而不是机械读取文件。`Grep` 查内容，`Glob` 查路径；`Edit` 需要唯一锚点，失败时用 `Read + Write` 回退；理解大型代码库时应先找入口和导出名，再逐步追踪调用链。

#### wrapper module 的调用追踪

官方特别提到 wrapper modules：真实业务函数可能先从一个模块导出，再被多个入口改名引用。如果只搜索原函数名，容易漏掉调用链。

```ts
// api/index.ts
export { createCustomer as createUser } from "./customers";
export { refundOrder } from "./orders";

// handlers/signup.ts
import { createUser } from "../api";
```

**正确追踪方式：**

1. 先 Read wrapper module，识别所有 exported names。
2. 对每个导出名分别 Grep，例如 `createCustomer`、`createUser`。
3. 再沿 import/read 路径追踪调用者和数据流。

**考试判断方法**：问“找某个函数所有调用点”时，不要只读一个文件；先 Grep 内容，再 Read 关键文件。如果涉及 `index.ts`、barrel export、wrapper module，要先找导出名再逐个搜索。

---

## 讲透这一域：工具设计、MCP 集成与工具选择的工程判断

Domain 2 的表面主题是工具和 MCP，但真正考点是“如何把模型的能力边界做成可调用、可解释、可恢复的接口”。在生产系统里，工具不是简单函数，也不是把后端 API 暴露给 Claude 就结束。工具描述决定模型是否选对工具，input schema 决定模型能否构造正确参数，错误结构决定模型能否恢复，工具分配决定子智能体是否越权，MCP 配置决定团队是否能复现。Domain 2 的题目通常不会问一个孤立定义，而会给出一个系统失效现象，让你判断根因和最佳修复。

### 一、工具接口是给模型看的产品界面

设计工具时，很多工程师只关注函数签名，例如 `lookup_order(order_id)` 或 `search_docs(query)`。但对 LLM 来说，工具描述才是“产品界面”。模型不会像人类开发者一样去读后端实现，它根据 name、description、schema、system prompt 共同判断何时调用。描述写得越模糊，模型越容易在相似工具中混淆。比如 `get_customer` 和 `lookup_order` 如果都写“获取信息”，模型很难知道退款前应该先验证客户，还是直接查订单。

一个好的工具描述至少要回答四个问题。第一，输入是什么，格式如何，例如 customer_id 是否必须是 `CUST-XXXX`，email 是否允许替代。第二，输出是什么，模型能拿到哪些字段。第三，什么场景下必须调用，例如处理退款前必须先验证客户。第四，什么时候不要调用，应该使用哪个相似工具替代。第四点经常被忽视，但考试很爱考，因为模型误选通常发生在边界模糊处。

工具命名也要具体。`analyze_content` 这种名字范围太大，容易吸附所有“分析”任务；`extract_web_results`、`summarize_uploaded_document`、`verify_claim_against_source` 更明确。工具拆分不是为了增加工具数量，而是为了减少单个工具内部的模糊分支。如果一个工具靠 `analysis_type` 参数决定完全不同的行为，模型既要选工具又要选模式，错误空间变大。拆成多个专用工具后，每个工具的描述、schema 和输出契约都更清楚。

### 二、schema 约束解决的是参数形状，不是业务语义

JSON Schema 能限制参数类型、必填字段、枚举值、数组结构，但它不能替你定义业务边界。比如 schema 可以要求 `refund_amount` 是 number，但不能自动判断该金额是否超过政策限制；可以要求 `order_id` 是 string，但不能保证这个订单属于当前客户。因此工具设计要把 schema 与描述、后端校验、错误响应结合起来。

字段设计要尽量表达真实业务可能性。可能不存在的信息不要强制 required，否则模型可能编造；有限选项要用 enum，避免下游无法处理自由文本；可扩展分类要用 `other + detail` 模式，既保留机器可读性，又允许新情况。数组字段必须定义 `items` schema，否则模型可能返回结构不一致的列表。嵌套对象要明确 required 边界，避免地址、商品项、权限范围等复杂字段变成松散文本。

考试里如果题目说“模型经常传入无效状态值”“下游系统无法解析分类”“字段不存在时模型乱填”，优先考虑 schema 设计问题。若题目说“模型选错工具”，优先考虑 description/name/system prompt。若题目说“工具执行后系统不知道是否该重试”，优先考虑错误结构。不同症状对应不同修复，不要一律加 few-shot。

### 三、结构化错误是智能体恢复能力的基础

工具调用失败不可避免，关键在于失败是否可解释。返回 `"Operation failed"` 几乎等于把恢复决策交给模型猜。一个生产级 MCP 工具应该区分 transient、validation、business、permission、not_found 等错误类别，并明确 `isRetryable`。超时、503、连接重置通常可重试；输入格式错误、权限不足、业务规则拒绝通常不可盲目重试。错误描述还要能面向用户或协调者解释，而不是只给开发者堆栈。

有效空结果也必须和失败区分。查询订单成功但没有匹配，应返回 `success: true, count: 0, orders: []`；查询服务超时，应返回 `isError: true, errorCategory: transient`。如果把失败吞掉返回空列表，模型会误以为“真的没有结果”；如果把空结果当错误，模型会做无意义重试。这个点在客户支持、搜索系统、文档检索场景都很常见。

子智能体错误传播还要有层级。瞬时错误应先在子智能体本地重试，重试耗尽后再上报协调者；上报时要带 attemptedAction、attempts、partialResults、alternatives。协调者拿到这些信息后，可以选择换查询词、换工具、带部分结果继续、或升级人工。如果子智能体只说“搜索不可用”，协调者无法做出智能恢复。

### 四、工具越多不等于能力越强

给模型 18 个工具看起来强大，实际会降低选择可靠性。工具选择是一种分类任务，候选项越多、描述越重叠，错误率越高。多智能体系统应按角色分配工具：搜索代理拿搜索和抓取工具，文档代理拿加载和抽取工具，合成代理拿格式化和轻量核验工具，协调者拿 Task 和路由工具。角色边界越清楚，系统越可解释。

“跨角色工具”要谨慎。合成代理可能确实需要核验一个事实，但不应该默认拥有完整 web search 能力，否则它会绕过搜索代理，导致引用标准、来源筛选、覆盖检查失控。更好的设计是给它一个窄化的 `verify_fact` 工具，复杂检索仍交回协调者重新委托。工具权限既是能力设计，也是治理设计。

受限替代工具也很重要。通用 `fetch_url` 可以抓任意网页，风险大；`load_document` 可以限制只加载公司文档库 URL，并返回标准化 metadata。通用 `run_sql` 风险很高；受限 `lookup_customer_orders(customer_id)` 更安全。考试里看到“模型用工具访问了不该访问的资源”或“抓取了错误来源”，通常应选择更受限、更语义化的工具。

### 五、tool_choice 是流程控制的一部分

`tool_choice` 的三个模式要和工作流需求绑定。`auto` 适合允许模型直接回答的场景，例如普通问答或工具可选任务。`any` 适合必须得到结构化工具输出，但多个工具都可能适用的场景，例如未知文档类型要在 invoice、receipt、contract extractor 中选择一个。forced tool selection 适合第一步必须固定的场景，例如先 `extract_metadata`，再根据 metadata 进行 enrich。

不要把 `tool_choice: "any"` 和 forced tool 混淆。`any` 只能保证调用某个工具，不能保证是哪一个；forced tool 保证调用指定工具，但不适合后续开放决策。复杂流程常见做法是第一轮 forced tool 获取元数据，第二轮放开为 auto 或 any，让模型基于结果继续。考试题如果强调“必须先调用 X”，答案应是 forced tool；如果强调“不允许纯文本，必须用某个 schema 输出”，答案多半是 `any`。

### 六、MCP 是团队级工具接入层，不只是本地插件

MCP 的价值在于把工具和资源标准化，让 Claude Code 和 agent workflow 能通过统一协议访问外部系统。项目级 `.mcp.json` 适合团队共享，因为它能随仓库提交，保证每个成员和 CI 环境看到同样的 server 配置。用户级 `~/.claude.json` 适合个人实验或私人工具，不应作为团队依赖。如果新同事无法使用某个 MCP 工具，首先检查它是否只配置在某个人的用户级文件里。

凭据管理必须通过环境变量展开，例如 `${GITHUB_TOKEN}`、`${JIRA_TOKEN}`、`${DATABASE_URL}`。把 token 写进 `.mcp.json` 是安全事故。更细一点，工具描述也要写清楚能力和输出，否则模型可能不用 MCP 工具，而退回内置 Grep 或 Bash。比如一个远程文档搜索 MCP 如果只写“search docs”，模型不知道它比本地 Grep 更适合；如果写清“搜索权限过滤后的远程知识库，返回 title/url/updated_at/snippet”，模型更容易正确选择。

MCP resources 和 MCP tools 要区分。tools 是动作，可能有副作用；resources 是只读内容目录，适合暴露 issue 摘要、文档树、数据库 schema、API catalog。resources 能减少探索性工具调用，因为模型先知道有哪些内容可用，再决定是否调用工具读取细节。大型系统里，先暴露目录再读取细节，比让模型盲目搜索更省 token、更可控。

社区 MCP 与自定义 MCP 的选择也体现工程判断。GitHub、Jira、Slack、PostgreSQL 这类标准系统，优先用成熟社区 server；内部审批流、私有知识库、团队专属权限模型才值得自研。自研 MCP 的成本不只是写代码，还包括 schema、错误结构、鉴权、部署、监控和版本兼容。考试里看到标准集成需求，通常不应选择从零写一个 server。

### 七、内置工具选择体现代码库探索策略

`Grep` 和 `Glob` 是最容易混淆、也最容易得分的点。Grep 查内容，适合找函数名、错误消息、import、调用者；Glob 查路径，适合找 `**/*.test.tsx`、`src/**/*.py`、迁移文件。Read 用于读完整文件，Edit 用于唯一锚点局部修改，Write 用于创建或整文件替换，Bash 用于运行测试和命令。工具选择应服务探索策略，而不是随手读文件。

大型代码库理解要渐进。先用 Grep 找入口点或关键标识，再 Read 入口文件，沿 import 和调用链继续 Grep。不要一开始全量读取，因为上下文会被低价值文件占满。wrapper module 和 barrel export 要特别注意：函数可能以别名导出，如果只搜原名会漏调用点。正确做法是先读 wrapper，列出 exported names，再逐个搜索。

Edit 失败时也有固定判断。Edit 依赖唯一文本锚点；锚点不唯一就可能改错位置或失败。可靠回退是 Read 全文件，确认目标位置，再 Write 或用更精确锚点 Edit。考试如果问“Edit 因非唯一匹配失败怎么办”，不要选择反复尝试模糊文本，而是 Read + Write 或构造唯一上下文。

### 八、Domain 2 的答题框架

遇到工具/MCP 题，可以按六类症状定位。第一，模型选错工具：检查 description、name、边界、system prompt keyword。第二，模型传错参数：检查 input schema、required、enum、nullable、示例。第三，失败后乱重试：检查结构化错误、isRetryable、errorCategory。第四，子智能体越权：检查工具分配和角色最小权限。第五，团队不可复现：检查 MCP 作用域和环境变量。第六，代码库探索低效：检查 Grep/Glob/Read/Edit 使用顺序。

最常见错误答案也很固定：给所有 agent 全部工具；用通用工具代替受限工具；工具描述只写一句话；错误统一返回 `"failed"`；把空结果当失败；把团队 MCP 配在用户级；把密钥提交到 `.mcp.json`；用 Glob 搜函数调用；Edit 锚点不唯一还继续硬改。识别这些反模式，Domain 2 的题目会非常稳定。

### 九、从生产事故角度理解 Domain 2

如果把 Domain 2 放到真实系统里，它对应的不是“工具好不好用”，而是很多线上事故的源头。工具描述不清，会导致模型把退款工具当查询工具；schema 太松，会导致参数通过模型生成但被后端拒绝；错误响应太泛，会导致模型对不可重试错误反复重试；工具权限太宽，会导致本该只读的代理执行写操作；MCP 配置放错作用域，会导致本地可用、CI 或同事机器不可用。这些问题单独看都像小配置错误，组合起来会让智能体系统不可预测。

因此设计工具时要采用“契约优先”的思路。每个工具都应有清楚的输入契约、输出契约、错误契约和权限契约。输入契约回答“模型能传什么”；输出契约回答“模型会看到什么”；错误契约回答“失败后模型如何恢复”；权限契约回答“这个工具在哪个角色、哪个环境、哪个条件下可用”。只要其中一个契约缺失，Claude 就会用自然语言常识补洞，而生产系统不应该依赖这种补洞。

还要注意工具和业务流程之间的关系。有些工具本身没有副作用，例如 `lookup_order`；有些工具有轻微副作用，例如创建草稿；有些工具有强副作用，例如退款、发邮件、删除数据。工具副作用越强，越需要后端校验、hook 拦截、审计日志和人类确认。不要因为工具 schema 写得漂亮，就默认模型不会误用它。Domain 2 与 Domain 1 的区别在于：Domain 1 关注谁来编排流程，Domain 2 关注每个可调用能力本身是否足够清晰、安全、可恢复。

### 十、工具描述的写作模板

备考时可以把每个工具描述写成固定模板：第一句说明唯一职责；第二句说明输入字段和格式；第三句说明返回字段；第四句说明必须调用的业务场景；第五句说明不要用于哪些相似场景以及替代工具；最后补充边缘情况，例如找不到返回什么、权限不足返回什么、是否会产生副作用。这个模板能覆盖官方要求的 input formats、example queries、edge cases、boundary explanations。

例如查询订单工具可以写成：`lookup_order` 仅用于通过订单 ID 查询单个订单的状态、金额、商品和退货资格；输入必须是 `ORD-XXXX` 格式；如果需要验证客户身份，先调用 `get_customer`；如果要执行退款，不要用本工具，使用 `process_refund` 且必须满足验证前置条件；订单不存在时返回 success true 和 null order，而不是抛出 transient error。这样的描述会显著降低误路由。

这个模板也能帮助识别坏选项。凡是选项只说“add more examples to the prompt”，但题目根因是工具描述重叠，通常不是最佳答案。凡是选项建议“让协调者在 prompt 中提醒 synthesis agent 不要搜索”，但 synthesis agent 仍然拥有搜索工具，也不是最可靠方案。真正可靠的修复应从接口和权限层降低错误可能性，而不是只在自然语言里劝模型。

### 十一、把工具设计和安全审计放在一起看

工具一旦接入智能体系统，就不只是开发便利功能，而是一个可能被模型反复调用的执行边界。每个工具都应该能回答三个审计问题：谁可以调用，调用前需要什么前置条件，调用后如何记录结果。只读工具的审计要求较低，但仍要避免泄露无关数据；写操作工具必须有明确权限和业务校验；外部副作用工具，例如发邮件、创建工单、退款、修改账户，更需要幂等性、审批和日志。

幂等性在工具设计中很重要。模型可能因为网络错误、上下文不清或重试逻辑再次调用同一工具。如果 `process_refund` 不是幂等的，重复调用可能造成重复退款。因此高风险工具应支持 idempotency key，或者在后端检测同一订单、同一金额、同一原因是否已处理。考试不一定直接问幂等性，但当题目出现“重试导致重复副作用”时，正确思路是让工具层和后端层防重复，而不是只告诉模型“不要重复调用”。

工具输出也要考虑最小披露。`get_customer` 不应把客户所有历史地址、完整支付信息、内部风控标签都返回给模型，除非当前任务确实需要。返回字段越多，隐私风险越高，上下文成本越高，模型误用概率也越高。更好的方式是按任务提供受限工具，例如 `get_refund_eligibility` 只返回退款判断需要的字段。这样既保护数据，也让模型更容易推理。

MCP server 同样需要版本管理。团队共享 `.mcp.json` 后，server 的工具名、schema、错误格式如果变化，Claude Code 的行为也会变化。生产团队应把 MCP server 当作 API 管理：有版本、变更记录、兼容策略和测试。自定义 MCP 尤其如此，因为它连接的是团队内部系统，错误不仅影响模型回答，还可能影响真实业务流程。

### 十二、Domain 2 与其他 domain 的交叉考法

Domain 2 经常和 Domain 1 一起考。比如客户支持 agent 处理退款：Domain 1 判断必须先验证身份并用 gate 阻止未验证退款；Domain 2 判断 `get_customer`、`lookup_order`、`process_refund` 的描述、schema 和错误结构是否足够清楚。只会一个 domain 容易选到半对答案。

它也会和 Domain 4 一起考结构化输出。工具 schema 可以强制模型返回结构化参数，但字段语义仍要靠验证。它会和 Domain 5 一起考错误传播和上下文裁剪：工具返回错误要结构化，返回成功结果要裁剪成相关字段，避免上下文膨胀。复习时应把工具看成系统边界：向上影响 agent 推理，向下连接后端业务，中间决定可靠性。

最后复习 Domain 2 时，可以用“工具是否让模型更少猜测”作为总标准。好的工具名减少模型猜用途，好的描述减少模型猜边界，好的 schema 减少模型猜参数，好的错误结构减少模型猜恢复方式，好的权限分配减少模型猜自己能不能做某事，好的 MCP resource 减少模型猜系统里有哪些数据。只要某个设计仍然让模型在关键环节靠猜，就还有改进空间。

在考场上，看到工具相关题不要急着选“更聪明的模型”或“更长的提示”。先问接口有没有把意图表达清楚，失败有没有结构化，权限有没有最小化，配置有没有共享，内置工具有没有选对。Domain 2 的正确答案往往不是最复杂的，而是最能降低歧义和恢复成本的。

一个简单自测方法是拿任意工具问自己：如果只看这个工具名和描述，不看代码实现，Claude 能否知道它该在什么场景调用、不要在什么场景调用、失败后下一步是什么？如果答案是否定的，这个工具就还不是合格的智能体接口。传统 API 文档主要写给人类开发者，Agent 工具描述还必须写给模型决策器，这是两者最大的差异。

再进一步，工具设计要避免把业务判断隐藏在工具内部却不告诉模型。例如工具内部会过滤无权限订单，但描述没有说明，模型可能把空结果理解成客户没有订单；工具内部会自动截断搜索结果，但描述没有说明，模型可能以为覆盖完整。工具行为越透明，模型越能做正确后续决策。Domain 2 的核心能力，就是把后端能力翻译成模型能可靠使用的接口语言，并让这种接口在团队、自动化和生产环境中长期稳定。

我会这样检查：把工具描述遮住实现，只看文字说明。如果我都要猜，Claude 多半也会猜。

---

## 临考速查

1. **工具选择错误** → 第一步改工具描述（不是加 few-shot 或改路由逻辑）
2. **工具描述四要素** → 输入格式 + 使用场景 + 边缘情况 + 与相似工具的边界
3. **工具数量** → 优先给每个角色 4-5 个高相关工具；过多工具会降低选择可靠性
4. **`tool_choice: "any"`** → 保证调用工具（不返回纯文本）
5. **`tool_choice: "tool"`** → 强制调用特定工具
6. **MCP `.mcp.json`** → 项目级（团队共享，应提交）；`~/.claude.json` → 用户级（个人）
7. **MCP 传输** → stdio（本地进程）vs SSE（远程/HTTP）
8. **结构化错误** → `isError` + `errorCategory` + `isRetryable`
9. **空结果 ≠ 错误** → `{success: true, orders: []}` 是正常响应
10. **tool_use 块** → 有唯一 `id`，`tool_result` 中必须通过 `tool_use_id` 引用
11. **Grep vs Glob** → Grep 查文件内容，Glob 查文件路径
12. **MCP 资源** → 只读数据目录，减少探索性工具调用
13. **系统提示误路由** → keyword-sensitive 指令可能覆盖工具描述
14. **子智能体错误恢复** → transient failure 先本地重试，无法恢复再带部分结果上报
15. **wrapper module 追踪** → 先找导出名，再逐个 Grep 引用
