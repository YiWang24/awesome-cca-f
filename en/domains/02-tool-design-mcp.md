# Domain 2: Tool Design & MCP Integration

> **Weight: 18%**
> Official documentation: [Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | [MCP](https://docs.anthropic.com/en/docs/claude-code/mcp)

---

## Task Statement Coverage

| Task | Topic |
| ---- | ----------------------------------------------- |
| 2.1 | Design effective tool interfaces with clear descriptions and boundaries |
| 2.2 | Implementing structured error responses for MCP tools |
| 2.3 | Reasonably allocate tools among agents and configure tool selection |
| 2.4 | Integrate MCP server into Claude Code and agent workflow |
| 2.5 | Select and apply built-in tools efficiently |

---

### Task Statement 2.1: Design effective tool interfaces with clear descriptions and boundaries

#### Knowledge of:

- Tool descriptions as the primary mechanism LLMs use for tool selection; minimal descriptions lead to unreliable selection among similar tools
  - Tool description is the main basis for tool selection for the model. When descriptions are fuzzy or when multiple tool descriptions are similar, models cannot be reliably distinguished, leading to frequent incorrect selections. The tool name, description, and schema together constitute the routing interface.
- The importance of including input formats, example queries, edge cases, and boundary explanations in tool descriptions
  - The tool description should contain four elements: accepted input formats, typical usage scenarios, edge case handling, and differentiation instructions from similar tools. The absence of any one of these expands the room for model error.
- How ambiguous or overlapping tool descriptions cause misrouting (e.g., analyze_content vs analyze_document with near-identical descriptions)
  - When two tools have overlapping names or descriptions, the model treats them as near synonyms. Explicitly distinguishing responsibilities by renaming (e.g. `analyze_content` → `extract_web_results`) and updating descriptions is the preferred means of resolving misrouting.
- The impact of system prompt wording on tool selection: keyword-sensitive instructions can create unintended tool associations
  - Specific keywords in system prompts may establish implicit tool associations (such as "call analyze_content when processing content"), overwriting the otherwise clearly written tool description. When troubleshooting misrouting, you should check both the tool description and the system prompts.

#### Skills in:

- Writing tool descriptions that clearly differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives
  - A qualified tool description should also describe: unique responsibilities, input formats and examples, return fields, business scenarios that must be called, and boundary descriptions of which similar tools should be used instead.
- Renaming tools and updating descriptions to eliminate functional overlap (e.g., renaming analyze_content to extract_web_results with a web-specific description)
  - When misrouting persists, renaming the tool is often more effective than adding more instructions to the prompt. Tool names should directly reflect specific business actions (such as `refund_order` instead of `process`) to reduce ambiguity.
- Splitting generic tools into purpose-specific tools with defined input/output contracts (e.g., splitting a generic analyze_document into extract_data_points, summarize_content, and verify_claim_against_source)
  - Splitting a generalization tool with the `analysis_type` parameter into multiple specialized tools (such as `extract_data_points`, `summarize_content`, `verify_claim_against_source`) can make the selection conditions of each tool clearer and reduce the decision-making complexity of the model when selecting tools.
- Reviewing system prompts for keyword-sensitive instructions that might override well-written tool descriptions
  - When the tool description is clear but incorrect selections still occur, you should check whether the system prompt contains keyword-sensitive instructions (such as strongly associating a word with a specific tool). Such instructions may override the routing logic established by the tool description.

### Task Statement 2.2: Implement structured error responses for MCP tools

#### Knowledge of:

- The MCP isError flag pattern for communicating tool failures back to the agent
  - MCP tools should return a structured response with `isError: true` when failing, rather than just returning vague error text. Structured errors enable the model to make specific recovery decisions based on the error category.
- The distinction between transient errors (timeouts, service unavailability), validation errors (invalid input), business errors (policy violations), and permission errors
  - The error category determines the recovery strategy: `transient` (timeout, service unavailable) can usually be retried; `validation` (input format error), `permission` (insufficient permissions), `business` (business rule rejection) should not be retried blindly and require different processing paths.
- Why uniform error responses (generic "Operation failed") prevent the agent from making appropriate recovery decisions
  - Returns generic errors such as "Operation failed" so that the model cannot decide whether to retry, modify parameters, escalate labor, or notify the user. The value of structured errors is in providing a clear recovery path for each failure type.
- The difference between retryable and non-retryable errors, and how returning structured metadata prevents wasted retry attempts
  - `isRetryable: false` can prevent the model from initiating invalid retries for permanent failures (such as permission errors, business rule rejections); `isRetryable: true` can be used with the backoff strategy to recover transient errors locally and reduce unnecessary reporting.

#### Skills in:

- Returning structured error metadata including errorCategory (transient/validation/permission), isRetryable boolean, and human-readable descriptions
  - The structured error response should contain: `errorCategory` (machine-readable classification), `isRetryable` (Boolean value), `description` (user-readable description), `attemptedAction` (the operation being performed at the time of failure), and `partialResults` (partial results in scenarios such as timeout).
- Including retriable: false flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately
  - The error response rejected by the business rule should contain a user-oriented explanation (such as "The refund amount exceeds the automatic approval limit"), so that the model can give a compliant and clear reply to the user instead of just returning a technical error code.
- Implementing local error recovery within subagents for transient failures, propagating to the coordinator only errors that cannot be resolved locally along with partial results and what was attempted
  - Transient errors should be recovered locally by backing off and retrying first within the child agent. Only when retries are exhausted or cannot be resolved locally will the coordinator be reported, along with the attempted action sequence and partial results, to avoid the coordinator repeating invalid delegations.
- Distinguishing between access failures (needing retry decisions) and valid empty results (representing successful queries with no matches)
  - If the query is successful but there is no matching result, `success: true, count: 0, results: []` should be returned instead of swallowing the empty result or returning an error flag. Confusing the two can cause models to misinterpret normal "no data" situations as failures that require recovery.

### Task Statement 2.3: Distribute tools appropriately across agents and configure tool choice

#### Knowledge of:

- The principle that giving an agent access to too many tools (e.g., 18 instead of 4-5) degrades tool selection reliability by increasing decision complexity
  - An increase in the number of tools will increase the decision-making complexity of the model during selection, thereby reducing the reliability of selection. Each subagent should only hold the tools necessary to complete its role (usually 4-5), not all available tools.
- Why agents with tools outside their specialization tend to misuse them (e.g., a synthesis agent attempting web searches)
  - When the synthesis agent holds a search tool, it may bypass the research agent and directly search for information, resulting in inconsistent source screening standards, confusing citation formats, and the coordinator losing observability of the information flow. Restricting the toolset by role prevents this kind of role drift.
- Scoped tool access: giving agents only the tools needed for their role, with limited cross-role tools for specific high-frequency needs
  - Cross-role tools should be narrowed to cover only high-frequency and low-risk requirements (e.g. giving the composition agent a `verify_fact` tool rather than the full `fetch_url`). Complex cross-role data requirements should be handled through coordinator re-deletion.
- tool_choice configuration options: "auto", "any", and forced tool selection ({"type": "tool", "name": "..."})
  - `tool_choice` has three modes: `auto` (the model decides independently whether to call the tool and which one to call), `any` (a certain tool must be called, but which one is determined by the model), `{"type":"tool","name":"..."}` forces a specific tool to be specified. The degree of coercion of the three modes increases in sequence.

#### Skills in:

- Restricting each subagent's tool set to those relevant to its role, preventing cross-specialization misuse
  - Configure the tool whitelist for each type of sub-agent through the `allowed_tools` field to ensure that it can only perform operations within the role. The search agent holds search and crawling tools, and the document agent holds loading and extraction tools to prevent unauthorized calls.
- Replacing generic tools with constrained alternatives (e.g., replacing fetch_url with load_document that validates document URLs)
  - Replacing generalized tools with specialized tools with semantic boundaries (e.g. replacing `fetch_url(url)` with `load_document(document_url)` and restricting to document library domain names) reduces the risk of misuse while making the scope of the tool easier to understand for the model.
- Providing scoped cross-role tools for high-frequency needs (e.g., a verify_fact tool for the synthesis agent) while routing complex cases through the coordinator
  - The synthetic agent can hold narrowed verification tools (such as `verify_fact`) to meet the needs of high-frequency lightweight verification; complex situations that require re-searching or in-depth supplementary evidence should be sent back to the coordinator for re-commissioning to keep the information flow controllable.
- Using tool_choice forced selection to ensure a specific tool is called first (e.g., extract_metadata before forcing enrichment tools), then processing subsequent steps in follow-up turns
  - When the first step of the workflow must be fixed (for example, `extract_metadata` must be executed before the enrich operation can be performed), use forced tool selection to lock the first step; in the second step, switch to `auto` or `any` to let the model make subsequent decisions based on the metadata results.
- Setting tool_choice: "any" to guarantee the model calls a tool rather than returning conversational text
  - `tool_choice: "any"` Guarantees that the model must call the tool (choose one from the available set) and is not allowed to return a plain text response. It is suitable for scenarios where the document type is unknown but must be extracted in a structured manner to prevent the model from answering directly in natural language and bypassing tool calls.

### Task Statement 2.4: Integrate MCP servers into Claude Code and agent workflows

#### Knowledge of:

- MCP server scoping: project-level (.mcp.json) for shared team tooling vs user-level (~/.claude.json) for personal/experimental servers
  - Project-level `.mcp.json` MCP tools suitable for team sharing should be submitted to version control to ensure that all members and CI use the same server configuration. The `mcpServers` field of user-level `~/.claude.json` is suitable for personal experimentation and should not be relied upon as a team.
- Environment variable expansion in .mcp.json (e.g., ${GITHUB_TOKEN}) for credential management without committing secrets
  - Authentication credentials (API Key, Token, database connection string) must be expanded through environment variables (such as `${GITHUB_TOKEN}`) and must not be hard-coded into `.mcp.json`. Hardcoding credentials submitted with a repository is a security incident.
- That tools from all configured MCP servers are discovered at connection time and available simultaneously to the agent
  - All configured MCP server tools will be added to the tool pool at the same time when connecting. When multiple servers coexist, tool naming and descriptions must avoid conflicts, otherwise the model cannot reliably distinguish between tools with the same name or similar names from different servers.
- MCP resources as a mechanism for exposing content catalogs (e.g., issue summaries, documentation hierarchies, database schemas) to reduce exploratory tool calls
  - MCP Resources exposes data (such as issue summary list, document tree, database schema) in the form of a read-only content directory, allowing the model to understand the range of available data without initiating exploratory tool calls, reducing the number of unnecessary API calls.

#### Skills in:

- Configuring shared MCP servers in project-scoped .mcp.json with environment variable expansion for authentication tokens
  - Team-shared MCP configurations should be written to `.mcp.json` in the project root and submitted to version control; authentication values are read from environment variables at runtime via `${ENV_VAR}`, allowing configuration files to be shared securely without exposing credentials.
- Configuring personal/experimental MCP servers in user-scoped ~/.claude.json
  - MCP servers that are still experimental or belong only to individuals should have user-level configuration written to them and should not affect the project configuration. If written to the project level, all team members are required to configure corresponding credentials in the same environment variables, causing unnecessary environmental dependencies.
- Enhancing MCP tool descriptions to explain capabilities and outputs in detail, preventing the agent from preferring built-in tools (like Grep) over more capable MCP tools
  - If the MCP tool description is too brief (for example, just "search docs"), the model may prefer the built-in Grep or Bash. The description should illustrate the tool's unique capabilities, scope of permissions, return fields, and applicable scenarios, allowing the model to accurately determine when an MCP tool should be selected over a built-in tool.
- Choosing existing community MCP servers over custom implementations for standard integrations (e.g., Jira), reserving custom servers for team-specific workflows
  - Prioritize the use of mature community MCP servers for standard integrations such as GitHub, Jira, Slack, and PostgreSQL; it is only worthwhile to develop your own MCP server when it involves internal private systems, team-specific workflows, or specific permission models.
- Exposing content catalogs as MCP resources to give agents visibility into available data without requiring exploratory tool calls
  - Exposing directory information such as issue lists, document hierarchies, database schemas, and API directories as MCP Resources can provide a complete picture of the data before the model calls tools, improve the targeting of subsequent tool calls, and reduce blind exploration.

### Task Statement 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively

#### Knowledge of:

- Grep for content search (searching file contents for patterns like function names, error messages, or import statements)
  - Grep searches for patterns in file content, suitable for content-targeted searches such as finding function call locations, error message sources, import statements, variable references, etc.
- Glob for file path pattern matching (finding files by name or extension patterns)
  - Glob matches files by file path and name pattern, suitable for path-targeted searches such as finding all `*.test.tsx` files, `src/api/**/*.py` modules, all configuration files in a specific directory, etc.
- Read/Write for full file operations; Edit for targeted modifications using unique text matching
  - Edit uses unique text anchors for local modifications, suitable for pinpointing code segments; Read/Write is suitable for reading a complete file or overwriting the entire file. Edit fails when the target text is not unique in the file.
- When Edit fails due to non-unique text matches, using Read + Write as a fallback for reliable file modifications
  - When Edit fails because the anchor point is not unique, you should not repeatedly try to blur the anchor point. Instead, use Read to read the complete file content, confirm the target position, and then use Write or Edit with a more precise context to complete the modification.

#### Skills in:

- Selecting Grep for searching code content across a codebase (e.g., finding all callers of a function, locating error messages)
  - When tracing function callers, error sources, or import paths, Grep is the right tool. It can search directly in the file content to avoid reading a large number of files without purpose and consuming context.
- Selecting Glob for finding files matching naming patterns (e.g., \*_/_.test.tsx)
  - When it is necessary to locate specific files according to file naming conventions (such as all test files, all migration scripts), Glob's path pattern matching is more efficient than directory-by-directory Read.
- Using Read to load full file contents followed by Write when Edit cannot find unique anchor text
  - When the editing target text appears multiple times in the file (non-unique anchor point), Edit cannot determine which part should be modified. It must first read the entire file content and confirm the correct position before performing modifications.
- Building codebase understanding incrementally: starting with Grep to find entry points, then using Read to follow imports and trace flows, rather than reading all files upfront
  - Efficient exploration strategy for large code bases: first use Grep to locate the entry point (main, handler, router), and then Read the entry file to trace along the import chain. Do not read all files in full at the beginning - this will fill the context window with low-value content.
- Tracing function usage across wrapper modules by first identifying all exported names, then searching for each name across the codebase
  - The Wrapper module and barrel export cause functions to be exported as aliases, and searching only for the original function name misses the actual call site. The correct approach is to first read the wrapper file to list all exported names, and then perform Grep on each name separately.

---

## Task 2.1: Tool interface design

### Tool description is the primary mechanism for tool selection

> When the tool selection is wrong, **the first step is to change the tool description**, not to add few-shot or change the routing logic.

**Four elements of a good tool description:**

| Elements | Description | Example |
| -------------------------- | -------------------------- | ------------------------------------- |
| **Input format** | What type/format of input is accepted | "customer_id format CUST-XXXX" |
| **Usage scenarios** | Under what circumstances is this tool called | "Must be called before processing refund" |
| **Edge cases** | Special boundary conditions | "Non-existent ID returns null instead of an error" |
| **Boundary Description** | When to use this tool vs similar tools | "Not used for order lookup, please use lookup_order" |

```python
# Poor description (blurred boundaries between tools, leading to confusing choices)
{"name": "get_customer",  "description": "Get customer information"}
{"name": "lookup_order",  "description": "Get order information"}

# High-quality description (clear distinction to prevent confusion)
{
    "name": "get_customer",
    "description": (
        "Verify and obtain customer account information through customer ID or email."
        "Enter: customer_id (format CUST-XXXX) or email address."
        "Output: Customer name, account status, registration date, refund eligibility."
        "Usage scenario: Verify customer identity. This tool must be called before an order or refund can be processed."
        "Note: It is not used to query order details. For order query, please use lookup_order."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "customer_id": {
                "type": "string",
                "description": "Customer ID, format CUST-XXXX. Choose one from email."
            },
            "email": {
                "type": "string",
                "description": "Customer email address. Choose one from customer_id."
            }
        }
        # Note: neither customer_id nor email are required, but at least one must be provided
    }
}
```

### Tools Input Schema design

```python
# Nested objects
{
    "name": "create_order",
    "input_schema": {
        "type": "object",
        "properties": {
            "customer_id": {"type": "string"},
            "shipping_address": {
                "type": "object", # Nested objects
                "properties": {
                    "street": {"type": "string"},
                    "city":   {"type": "string"},
                    "zip":    {"type": "string"}
                },
                "required": ["street", "city", "zip"]
            },
            "items": {
                "type": "array", # array
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

# enum constraints (limit valid values ​​to prevent invalid input)
{
    "priority": {
        "type": "string",
        "enum": ["low", "medium", "high", "urgent"],
        "description": "processing priority"
    }
}

# nullable fields (information that may not exist)
{
    "discount_code": {
        "type": ["string", "null"],
        "description": "Discount code, null if there is no discount code"
    }
}
```

### Split general tools into specialized tools

```python
# Generalization tool (analysis_type parameter causes unpredictable behavior)
analyze_document(doc_id, analysis_type="summary"|"data"|"verify")

# Specialized tools (single responsibility, precise description)
extract_data_points(doc_id) # Extract only data points
summarize_content(doc_id) # Generate summary only
verify_claim_against_source(claim, doc_id) # Verify claim only
```

### What this tests

#### Troubleshooting order for mis-selected tools

The core of Domain 2.1 is not "the more tools, the better", but to allow the model to make stable choices between similar tools. Exam questions usually give two tools with similar functions, such as `analyze_content` and `analyze_document`, and then describe the model that is often chosen incorrectly. The priority judgment points are: whether the tool name is too abstract, whether the description does not include input/output/boundary, and whether the system prompts using certain keywords to lead the model to the wrong tool.

**Recommended order of investigation:**

1. First check whether the tool description clearly states "when to use it" and "when not to use it".
2. Check whether there is functional overlap between similar tools, and rename or split them if necessary.
3. Then check whether the input schema is sufficient to constrain the input format.
4. Finally, check whether there are keyword-sensitive instructions in the system prompt, such as "call analyze_content whenever content is encountered." Such prompts will overwrite the originally well-written tool description.

```python
# The system prompts that a wrong association was created
system_prompt = "When the user asks about content, use analyze_content."

# System prompts to retain decision boundaries
system_prompt = (
    "Use extract_web_results only for crawled web search results. "
    "Use summarize_document only for uploaded documents or document IDs. "
    "If the source type is unclear, ask for clarification or inspect metadata first."
)
```

Looking at the project, the tool design is not just about writing JSON schema. Name, description, schema, and system prompts together constitute the routing interface. Whenever one of the layers gives ambiguous or conflicting signals, the model may be mis-selecting tools.

---

## Task 2.2: Structured error response

### MCP error field design

| Field | Type | Description |
| ------------------ | ---------- | ----------------------------------------------- |
| `isError` | boolean | Whether it is an error (`true`) vs. a valid empty result (`false`) |
| `errorCategory` | string | Error category (determines whether to retry) |
| `isRetryable` | boolean | Whether it is possible to retry |
| `description` | string | Human-readable error description |
| `attemptedAction` | string | The operation being performed at the time of failure |
| `partialResults` | array/null | Partial results in case of timeout etc. |

### Error types and retryability

| Error type | `errorCategory` | `isRetryable` | Example |
| ------------- | --------------- | ------------- | -------------------------- |
| **Transient error** | `"transient"` | `true` | Timeout, service unavailability, network jitter |
| **Validation error** | `"validation"` | `false` | Invalid input format, missing parameters |
| **Business logic error** | `"business"` | `false` | The refund exceeds the limit and the balance is insufficient |
| **Permission error** | `"permission"` | `false` | No access rights, token expired |
| **The resource does not exist** | `"not_found"` | `false` | The order does not exist and the user does not exist |

```json
// Transient error (can be retried)
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "Database connection timed out, please try again in 5 seconds",
  "attemptedAction": "Query order #ORD-12345",
  "partialResults": null
}

// Permission error (cannot retry)
{
  "isError": true,
  "errorCategory": "permission",
  "isRetryable": false,
  "description": "No access to this order, it belongs to a different account",
  "attemptedAction": "Query order #ORD-99999"
}
```

### Distinguish between "access failure" and "valid empty result"

```python
# Confusing the two (the model can't tell whether it's a failure or "really no data")
def search_orders(customer_id: str):
    try:
        results = db.query(customer_id)
        return results # Empty list looks the same as failure
    except Exception:
        return [] # Disguise failure as empty result

# Clear distinction (model can make correct subsequent decisions)
def search_orders(customer_id: str):
    try:
        results = db.query(customer_id)
        return {
            "success": True,
            "orders": results, # empty list = query successful but no matches
            "count": len(results)
        }
    except TimeoutError:
        return {
            "isError": True,
            "errorCategory": "transient",
            "isRetryable": True,
            "description": "Query timed out, please try again later"
        }
    except PermissionError:
        return {
            "isError": True,
            "errorCategory": "permission",
            "isRetryable": False,
            "description": "Do not have permission to query orders for this account"
        }
```

### What this tests

The key point of Domain 2.2's judgment is that tool errors must give the model enough information to make recovery decisions. Just returning "failed" is useless; qualified answers should describe the type of error, whether it can be retried, a user-readable explanation, the attempted actions, and partial results. In the exam, special distinction should be made between "access failure" and "success but no result". The former requires a recovery strategy, while the latter is a valid business result.

#### Local recovery and error escalation in subagents

The official guidance emphasizes that transient failures should be partially recovered within the subagent first, rather than throwing all failures directly to the coordinator. The reason is that the coordinator is responsible for global planning. If each short timeout is upgraded, it will cause meaningless re-planning and repeated calls.

| Situation | Subagent action | Whether to report to coordinator |
| -------------------------- | --------------------------------------- | -------------------- |
| API timeout, 503, network jitter | Retry according to the backoff policy, retain attemptedAction | Report after exhaustion of retries |
| Input format error | Do not retry, return validation error | Report directly or request correction |
| Business rule rejection | No retry, return business error and user-oriented explanation | Report directly |
| Insufficient permissions | Do not retry blindly, indicating lack of permissions/credentials | Report directly |
| The query is successful but no results | Return success/count=0 | Do not report as an error |

```json
{
  "isError": true,
  "errorCategory": "transient",
  "isRetryable": false,
  "description": "Three consecutive calls to the search service timed out and local retries have been stopped.",
  "attemptedAction": "search_web(query='MCP Jira integration')",
  "attempts": 3,
  "partialResults": [
    { "title": "MCP server overview", "url": "https://example.com/mcp" }
  ]
}
```

**Examination Judgment Method**: If the option only returns `"Operation failed"`, it is usually wrong; if the option distinguishes between `errorCategory`, `isRetryable`, business explanation, partial results and attempted actions, it is usually more in line with official requirements.

---

## Task 2.3: Tool allocation and tool_choice

### Number and reliability of tools

Configuring too many tools for an agent will increase the complexity of decision-making and reduce the reliability of tool selection. The official example emphasizes **18 tools vs 4-5 tools**: 4-5 tools that are highly relevant to the role are usually more reliable than 18 mixed tools.

**Role-wide assignment tools:**

| Subagent role | Tool configuration | Number of tools |
|----------------|---------------------------------------------|--------|
| Network search subagent | `[search_web, fetch_url]` | 2 |
| Document analysis subagent | `[load_document, extract_data, summarize]` | 3 |
| Report generation subagent | `[generate_report, format_citations]` | 2 |
| Coordination agent | `[Task, search_web]` (sub-agent tool) | 2-3 |

### What this tests

The core of Domain 2.3 is "minimum necessary toolset + correct tool_choice". Exam questions usually use "given too many tools leading to misuse", "synthetic agent bypasses search agent" and "must call a tool first" to test whether you can limit the character boundaries and choose `auto`, `any` or forced tool selection.

#### Restricted toolsets and cross-role tools

Each subagent should be given only the tools necessary to complete the role's tasks. If a synthesis agent also obtains `search_web` and `fetch_url`, it may bypass the research agent and check the information on its own, causing evidence sources, citation formats, and task division to get out of control.

| Role | Tools that should be given | Tools that should not be given by default | Reasons |
| -------------------------- | ------------------------------------------------------------- | -------------------------- | ----------------------------------------------------- |
| Research agent | `search_web`, `fetch_url`, `load_document` | `generate_final_report` | Responsible for collecting evidence, not responsible for final synthesis |
| Synthesis agent | `summarize_findings`, `format_citations`, `verify_fact` | `search_web`, `fetch_url` | Responsible for synthesis and can do light fact checking, but complex searches should be reported to coordinator |
| Data extraction agent | `load_document`, `extract_data_points` | `send_email`, `create_ticket` | Only perform structured extraction and do not perform external side effects |
| Coordinator | `delegate_task`, `route_to_agent` | Low-level fine-grained business tools | Responsible for scheduling and should not perform all details personally |

**Restricted Alternative Tool Idea**: Instead of giving the generic `fetch_url(url)`, give `load_document(document_url)` and verify that the URL is from an allowed document domain. The closer the tool is to business semantics, the less likely the model is to be misused.

```python
# Too wide: Any URL can be crawled, but it is easy to crawl the wrong source or bypass the process.
fetch_url(url: str)

# Restricted: Only document library URLs are allowed to be loaded and standardized metadata is returned
load_document(document_url: str)  # validates docs.example.com/*
```

### `tool_choice` Configuration options

```python
# auto (default): The model decides whether and which tool to call.
# Applies to: General conversations, allowing direct text replies
tool_choice = {"type": "auto"}

# any: The model must call at least one tool (plain text responses are not allowed)
# Applicable: The output must be structured and the conversation text cannot be returned.
tool_choice = {"type": "any"}

# tool (mandatory): The model must call this specific tool
# Applies to: It is known that a specific step must be run first
tool_choice = {"type": "tool", "name": "extract_metadata"}
```

Typical usage scenarios of **`tool_choice: "any"`:**

```python
# The document type is unknown but must be extracted in a structured manner (models are not allowed to say "I don't know what type this is")
response = client.messages.create(
    tools=[invoice_extractor, receipt_extractor, contract_extractor],
    tool_choice={"type": "any"}, # Guaranteed to call one of them instead of returning plain text
    messages=messages
)
```

**Typical process of forced tool selection:**

```python
# The first step must be to extract metadata before entering subsequent enrichment.
first_turn = client.messages.create(
    tools=[extract_metadata],
    tool_choice={"type": "tool", "name": "extract_metadata"},
    messages=messages
)

# The second step is to decide to call enrich_customer, enrich_order or summarize_document based on metadata.
```

**Examination Judgment Method**: When the question requires "a tool must be called first", select forced tool selection; when the question requires "a tool must be called, but which one is determined by the model", select `tool_choice: "any"`; when the model is allowed to answer directly, select `auto`.

### tool_use Content block structure

Understanding the `tool_use` content block structure in API responses:

```python
# tool_use block in Claude's response
{
    "type": "tool_use",
    "id": "toulu_01XFDUDYJgAACTvnkyfe3yCN", # Unique ID, must be quoted in tool_result
    "name": "get_customer",
    "input": {
        "customer_id": "CUST-1234"
    }
}

# Access method
for block in response.content:
    if block.type == "tool_use":
        print(block.id)     # "toolu_01XFD..."
        print(block.name)   # "get_customer"
        print(block.input)  # {"customer_id": "CUST-1234"}
```

---

## Task 2.4: MCP Server Integration

### MCP configuration scope

| Scope | Configuration file location | Commit to version control | Purpose |
| ---------- | ---------------------------------------- | --------------- | ------------------ |
| **Project level** | `.mcp.json` (project root directory) | Should be submitted | Team sharing tools |
| **USER-LEVEL** | `mcpServers` field of `~/.claude.json` | Do not commit | Personal/Experimental Server |

### MCP transport types

| Transmission type | Description | Applicable scenarios |
| ---------------------------- | ------------------------------- | ---------------------------- |
| **stdio** | Communication via standard input and output, local process | Local tools (file system, database) |
| **SSE (Server-Sent Events)** | HTTP long connection streaming communication | Remote service, cloud API |

```json
// .mcp.json complete example
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server", // stdio transmission
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}" // Environment variable, no hardcoded key
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
      "url": "https://api.example.com/mcp", // SSE transmission (use url instead of command)
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

### MCP Tools vs MCP Resources

| Type | Purpose | Claude How to use |
| ------------ | ------------------------ | ------------------ |
| **MCP Tools** | Perform operations (with side effects) | Agent is called in a loop |
| **MCP resources** | Expose content directory (read-only data) | Inject context or provide browsing |

**The value of MCP resources**: Let the agent understand the available data without calling exploratory tools and reduce unnecessary API calls.

Example resources:

- `issues://` — Expose summary list of all Issues
- `schema://` — Expose database structure
- `docs://` — Expose document hierarchy

### Community MCP vs Custom MCP

| Situation | Choice |
|--------------------------------------------- | ---------------------------- |
| Standard integrations (GitHub, Jira, Slack, PostgreSQL) | Preferred use of community MCP servers |
| Internal private systems, team-specific workflows | Building custom MCP servers |

### What this tests

The practical focus of Domain 2.4 is on reproducible configuration and tool discoverability of MCP. The team shared server should put project-level `.mcp.json`, and personal experiments should put user-level configuration; credentials must be injected through environment variables; the resource directory should be exposed through MCP resources to reduce blind exploration of the model.

#### Claude Code Integration Checklist

The Domain 2.4 exam focuses on "configuring locations, credential management, tool discovery, and resource exposure." Project-level `.mcp.json` is suitable for team sharing, and the key should be expanded through `${ENV_VAR}`; user-level `~/.claude.json` is suitable for personal or experimental servers and cannot require teammates to also install it.

**Configuration check:**

1. Whether the team shared MCP server is written in the project root directory `.mcp.json`.
2. Whether the token, API key, and database connection string are expanded through environment variables instead of hard-coded.
3. After multiple MCP servers are connected at the same time, will the tool name and description conflict?
4. Whether the MCP tool description explains the capabilities, inputs, outputs and advantages to avoid the model falling back to the built-in Grep.
5. Whether to expose directory information such as issue summary, document hierarchy, and database schema as MCP resources.

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

**Risk of insufficient tool description**: If the MCP tool only writes `"search docs"`, the model may choose the built-in Grep to scan local files; if the description clearly states "Search the remote knowledge base and return a summary after title/URL/update time/permission filtering", the model is more likely to choose the MCP tool.

**Community vs customization**: For standard integrations such as Jira, GitHub, Slack, and PostgreSQL, mature community MCP servers are preferred; only internal approval flows, private data models, and team-specific operations require custom MCP servers.

---

## Task 2.5: Built-in tool selection

### Tool Cheat Sheet

| Tools | Usage | Typical scenarios |
| ---------- | ------------------------------- | ----------------------------------------------- |
| **Grep** | Search for patterns in **file contents** | Find function callers, find TODO comments, find import statements |
| **Glob** | Find files by **file path/name pattern** | Find all `*.test.tsx`, find `src/api/**/*.py` |
| **Read** | Read complete file content | View specific file |
| **Write** | Write/overwrite entire file | Create new file or completely replace |
| **Edit** | Local modification based on **unique text anchor** | Exactly modify a specific code segment (fails when the anchor is not unique) |
| **Bash** | Execute shell commands | Run tests, install dependencies, and perform git operations |

> This group is the easiest to mix: Grep checks the content, and Glob checks the path.

### Grep vs Glob Decision

```
In which files do you need to find the function/variable name? → Grep (search content)
Need to find files that match a specific naming pattern?      → Glob (by path)
Need to find all test files?                → Glob(**/*.test.ts)
Need to find where to call an API?          → Grep (search for calling statements)
Need to find all files under src/components/?  → Glob (directory mode)
Need to find a file with a specific error code?         → Grep (search content keywords)
```

### Edit fallback strategy when failure

```
Edit tool (text anchor not unique in file) → failed
           ↓
    Read reads the complete file content
           ↓
    Write writes the complete modified content
```

### Progressive Code Base Understanding Strategy

```
1. Grep (find entry points: main, app, handler, router and other keywords)
   ↓
2. Read (read the entry file and track the import chain)
   ↓
3. Grep (search for callers of key functions and understand data flow)
   ↓
4. Read related files on demand (do not read the entire code base at the beginning)
```

### What this tests

The Domain 2.5 exam focuses on selecting the correct built-in tools rather than mechanically reading files. `Grep` checks the content, `Glob` checks the path; `Edit` requires a unique anchor point, and uses `Read + Write` to fall back when it fails; when understanding a large code base, you should first find the entry and export names, and then gradually trace the call chain.

#### Call tracking of wrapper module

The official specifically mentioned wrapper modules: real business functions may be exported from a module first and then renamed and referenced by multiple entries. If you only search for the original function name, it is easy to miss the call chain.

```ts
// api/index.ts
export { createCustomer as createUser } from "./customers";
export { refundOrder } from "./orders";

// handlers/signup.ts
import { createUser } from "../api";
```

**Correct tracking method:**

1. First read the wrapper module and identify all exported names.
2. Grep each export name separately, such as `createCustomer`, `createUser`.
3. Then trace the caller and data flow along the import/read path.

**Examination Judgment Method**: When asked "Find all call points of a certain function", don't just read one file; grep the content first, and then read the key files. If `index.ts`, barrel export, wrapper module is involved, first find the export name and then search one by one.

---

## Domain 2 deep dive: making sense of tool design, MCP, and the judgment calls in between

Domain 2 is fundamentally about one thing: turning backend capabilities into an interface the model can use reliably. A tool is not just a function signature — it’s a contract between the model’s decision-making loop and the rest of your system. The description tells the model when to call it. The schema shapes the parameters. The error response tells the model what happened and what to do next. The permission scope tells the agent what it’s allowed to touch. When any of those four contracts is missing or vague, the model fills the gap with guesswork — and in production, guesswork compounds.

### 1. The tool description is the model’s product interface

Engineers often focus on the function signature and assume the model will figure out the rest. It won’t. The model doesn’t read your implementation; it reads the name, description, schema, and system prompt — and makes a routing decision from that alone. If `get_customer` and `lookup_order` both say “get information,” the model can’t reliably know whether to verify the customer first before issuing a refund, or skip straight to the order.

A good tool description answers four questions: what input does it accept and in what format; what does it return; when must it be called; and when should it *not* be called and which alternative applies instead. That last point is the one most often skipped — and it’s exactly what exam questions test, because tool misselection almost always happens at boundary conditions rather than clear-cut cases.

Tool naming matters just as much. `analyze_content` absorbs every “analysis” task because the name is too broad. `extract_web_results`, `summarize_uploaded_document`, and `verify_claim_against_source` each have a specific job the model can recognize. Splitting a multi-mode tool into specialized tools isn’t about adding more tools — it’s about removing ambiguous decision branches within a single one. A tool with an `analysis_type` parameter asks the model to select both the tool and the mode, doubling the opportunity for error.

### 2. Schema constraints define parameter shape, not business logic

JSON Schema handles types, required fields, enumerations, and nested structures — but it cannot enforce business rules. The schema can require `refund_amount` to be a number; it can’t determine whether that amount exceeds policy limits. That validation belongs in the backend and the error response, not the schema alone.

Field design should reflect real-world possibilities. Don’t mark a field as required if it legitimately might not exist — the model will fabricate a value rather than omit it. Use `enum` for fixed-value fields so downstream systems don’t have to parse free text. For extensible categories, `other` plus a `detail` field lets you retain machine-readable structure while handling new cases gracefully. Array fields need a defined `items` schema, or the model may return a list with inconsistent element shapes. Nested objects — addresses, line items, permission ranges — need explicit `required` boundaries to stay structured.

The exam uses specific symptom language to point at specific fix layers. “Model passes invalid status values” → schema problem. “Model picks the wrong tool” → description, naming, or system prompt. “System doesn’t know whether to retry after a tool call fails” → error structure. Match the symptom to the layer, and the answer usually follows.

### 3. Structured errors are what make recovery possible

A tool returning `”Operation failed”` hands all recovery decisions back to the model’s intuition. That’s not recoverable behavior in any meaningful sense. A well-designed MCP tool returns `errorCategory` (`transient`, `validation`, `business`, `permission`), `isRetryable`, a human-readable description, `attemptedAction`, and `partialResults` where applicable. Timeouts and 503s are retryable; permission denials and business-rule rejections are not.

Empty results are not failures — and conflating them is a persistent source of bugs. A successful query with no matches should return `success: true, count: 0, results: []`. Swallowing a timeout and returning an empty list instead looks identical to “no data” from the model’s perspective, causing it to skip recovery. Treating a valid empty result as an error causes pointless retries. These two cases require different responses, and the model can only distinguish them if the tool makes the distinction explicit.

For subagent error propagation, transient failures should be handled locally first: back off, retry, exhaust attempts, then escalate to the coordinator with `attemptedAction`, `attempts`, and `partialResults`. The coordinator can then decide whether to retry with different parameters, switch tools, proceed on partial data, or escalate to a human. If the subagent just reports “service unavailable,” the coordinator has nothing to act on.

### 4. More tools means less reliable selection

Giving an agent 18 tools increases decision complexity and drives down routing accuracy — tool selection is a classification problem, and more overlapping candidates means more errors. Each subagent should hold only the tools its role requires: search agents get search and fetch tools, document agents get load and extract tools, synthesis agents get format and lightweight-verification tools, coordinators get Task and routing tools. Role boundaries also make the system easier to debug, because a misrouted call immediately reveals which boundary was crossed.

Cross-role tools need careful handling. A synthesis agent might legitimately need to verify a fact, but it shouldn’t have full web search capabilities — that bypasses the search agent, breaking citation standards, source filtering, and workflow observability. A narrowed `verify_fact` tool handles the common case; complex search requirements get re-delegated to the coordinator. Similarly, `fetch_url` can retrieve any web page, which is a wide surface; `load_document(document_url)` constrained to your document library is both safer and easier for the model to reason about correctly. When an exam question mentions a model accessing resources it shouldn’t or fetching from the wrong source, the fix is almost always a more semantically restricted tool.

### 5. `tool_choice` is a workflow control mechanism

The three `tool_choice` modes map to three distinct workflow needs. `auto` lets the model decide whether and which tool to call — appropriate for general tasks where a direct text response is sometimes the right answer. `any` guarantees the model calls *some* tool from the available set, preventing it from returning conversational text when structured output is required. Forced tool selection (`{“type”: “tool”, “name”: “...”}`) locks the model to a specific tool — appropriate when a workflow step must always execute in a fixed order, like running `extract_metadata` before any enrichment.

The exam distinguishes these precisely: “must call this specific tool first” → forced selection; “must produce structured output but any matching tool is acceptable” → `any`; “tool call is optional, direct response is fine” → `auto`. A common multi-turn pattern is to use forced selection in the first turn to acquire metadata, then release to `auto` or `any` in subsequent turns so the model can adapt based on what it found.

### 6. MCP is team infrastructure, not a local plugin

MCP standardizes how Claude Code and agent workflows connect to external systems. The project-level `.mcp.json` at the repository root should be committed to version control so every team member and CI job works with the same server configuration. User-level `~/.claude.json` is for personal experimentation — never require teammates to configure their machines to support tools that live there. If a colleague can’t access an MCP tool, the first place to check is whether it was configured at the user level by whoever set it up.

Credentials must be injected via environment variable expansion (`${GITHUB_TOKEN}`, `${JIRA_TOKEN}`, `${DATABASE_URL}`). Hardcoding a token in `.mcp.json` is a security incident, not a configuration shortcut. Tool descriptions for MCP tools also need to be detailed enough that the model prefers them over built-in alternatives. A remote documentation search tool that just says “search docs” will lose to the built-in Grep; one that explains it searches a remote knowledge base with permission filtering and returns title, URL, summary, and last-updated date gives the model the information it needs to make the right call.

MCP Resources are distinct from MCP Tools. Tools perform actions and may have side effects. Resources expose read-only content catalogs — issue summaries, document hierarchies, database schemas, API directories — so the model can understand what data is available before deciding whether to call a tool to retrieve details. In large systems, exposing a manifest first and reading on demand is more token-efficient and more predictable than letting the model probe blindly.

For integrations with standard systems like GitHub, Jira, Slack, or PostgreSQL, use mature community MCP servers. Build custom servers only when you have internal systems, proprietary workflows, or permission models that no existing server supports. Custom MCP involves not just writing code but owning the schema, error structure, authentication, deployment, monitoring, and version compatibility — that overhead is only worth it when the integration is genuinely unique.

### 7. Built-in tool selection is an exploration strategy

Grep and Glob are the two most commonly confused tools. Grep searches *file contents* for patterns — function names, error messages, import paths, variable references. Glob matches *file paths* by pattern — finding all `*.test.tsx` files, all `src/api/**/*.py` modules, every config file in a given directory. Choosing the wrong one wastes context window on irrelevant results. Read loads a complete file; Write creates or overwrites one; Edit makes targeted changes using a unique text anchor; Bash runs shell commands.

Large codebases are explored incrementally, not exhaustively. Start with Grep to locate entry points or key identifiers, then Read the entry file and follow import paths. Avoid loading every file upfront — the context fills with low-value content before you’ve found anything useful. Barrel exports and wrapper modules require special handling: a function may be exported under an alias, so searching only the original name will miss real call sites. The right approach is to read the wrapper first to collect all exported names, then Grep each one separately.

When Edit fails because the anchor text appears more than once, the fix is not to adjust the anchor until something works — that risks modifying the wrong location. Read the full file instead, identify exactly which instance to change, then Write the modified content or construct a unique anchor with more context.

### 8. Diagnosing Domain 2 questions by symptom

Domain 2 exam questions almost always describe a failure symptom and ask for the root cause or best fix. Six symptom categories map reliably to six fix areas:

- “Wrong tool selected” → improve the tool description: add boundaries, rename for specificity, check for conflicting system prompt keywords
- “Model passes malformed or invalid parameters” → tighten the input schema: add `required`, `enum`, `nullable`, and examples
- “System doesn’t know whether to retry after failure” → add structured error response with `errorCategory` and `isRetryable`
- “Subagent used a tool outside its role” → restrict the toolset to role-appropriate tools only
- “MCP tool works locally but not for teammates or in CI” → check whether configuration is at user level instead of project level; check credentials
- “Codebase exploration is slow or misses callers” → use Grep before Read; handle barrel exports; fall back from Edit to Read + Write when anchors aren’t unique

The wrong answers tend to cluster around a few anti-patterns: giving all tools to all agents, writing one-sentence tool descriptions, returning a generic error string, treating empty results as failures, keeping team MCP in user-level config, committing tokens to `.mcp.json`, and using Glob when the question is about content rather than file paths. Recognizing those patterns makes elimination fast.

### 9. The overarching test: does the interface reduce guessing?

Every good tool design decision reduces the amount the model has to infer. A precise name reduces guessing about purpose. A complete description reduces guessing about boundaries. A well-constrained schema reduces guessing about parameter values. A structured error response reduces guessing about recovery. Scoped permissions reduce guessing about what the agent is allowed to do. MCP Resources reduce guessing about what data exists.

When reviewing a Domain 2 question, ask whether the proposed fix reduces a specific kind of guessing or just adds more natural-language persuasion in a prompt. A system prompt instruction telling a synthesis agent not to search bypasses the actual problem — the agent still *has* the search tool. Removing or replacing the tool solves the problem at the interface level, where it belongs. The correct answer for Domain 2 usually looks like “change the structure,” not “add another instruction.”

---

## Pre-exam checklist

1. **Wrong tool selection** → The first step is to change the tool description (not add few-shot or change the routing logic)
2. **Four elements of tool description** → Input format + usage scenarios + edge cases + boundaries with similar tools
3. **Number of Tools** → Give priority to 4-5 highly relevant tools for each role; too many tools will reduce the reliability of selection
4. **`tool_choice: "any"`** → Guaranteed to call the tool (no plain text returned)
5. **`tool_choice: "tool"`** → Force to call specific tools
6. **MCP `.mcp.json`** → project level (shared by team, should be submitted); `~/.claude.json` → user level (individual)
7. **MCP transport** → stdio (local process) vs SSE (remote/HTTP)
8. **Structured error** → `isError` + `errorCategory` + `isRetryable`
9. **Empty result ≠ Error** → `{success: true, orders: []}` is a normal response
10. **tool_use block** → has only `id`, `tool_result` must be referenced through `tool_use_id`
11. **Grep vs Glob** → Grep checks the file content, Glob checks the file path
12. **MCP Resources** → Read-only data directory to reduce exploratory tool calls
13. **System prompts misrouting** → keyword-sensitive directive may overwrite tool description
14. **Sub-agent error recovery** → transient failure. Retry locally first. If unable to recover, report with partial results.
15. **wrapper module tracking** → first find the export name, and then Grep the references one by one