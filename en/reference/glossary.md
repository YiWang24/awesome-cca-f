# Technical Glossary (English)

> Core technical terms covered by the CCA-F exam, arranged alphabetically.

---

## A

| Term | Definition |
|------|------|
| **Agent** | An AI system capable of autonomously performing multi-step tasks |
| **Agent SDK** | Claude's SDK for building agent applications |
| **Agentic Loop** | A loop driven by `stop_reason`: `tool_use` continues the loop, `end_turn` terminates it |
| **AgentDefinition** | Object that configures a subagent's system prompt, tool list, and description |
| **allowedTools** | Field in AgentDefinition that restricts which tools an agent can call; agents that spawn subagents must include `"Task"` |
| **API (Application Programming Interface)** | The programming interface provided by Claude |
| **argument-hint** | SKILL.md frontmatter field that displays guidance when the caller provides no arguments |

---

## B

| Term | Definition |
|------|------|
| **Batch API / Message Batches API** | Asynchronous batch-processing API with roughly 50% cost discount and up to a 24-hour processing window; does not support multi-turn tool use |
| **Bash** | Built-in Claude Code tool for running shell commands; the only tool that can run tests, builds, and installs |

---

## C

| Term | Definition |
|------|------|
| **Case Facts** | Persistent structure for key facts (amounts, dates, order numbers, status), injected every turn to prevent loss in long conversations |
| **CI/CD** | Automated software build, test, and deployment pipeline |
| **Claim-Source Mapping** | Structure that binds each conclusion in multi-source synthesis to a specific source (URL, document name, excerpt, date) |
| **CLAUDE.md** | Claude configuration file read by Claude Code at the project or user level; user-level files are not shared through version control |
| **Claude API** | REST API for interacting with Claude models |
| **Claude Code** | Anthropic's command-line coding agent tool |
| **Confidence Calibration** | Calibrating model confidence scores with a labeled validation set so they reflect actual accuracy |
| **context: fork** | SKILL.md frontmatter option that runs a Skill in an isolated subagent context to avoid polluting the main conversation |
| **Context Window** | Maximum number of tokens a model can process in a single request; varies by model version |
| **Coordinator** | Main agent in a Hub-and-Spoke architecture that manages task decomposition, delegation, aggregation, and error handling |
| **Coverage Gap** | Area in multi-agent research that remains uncovered because sources are unavailable or task decomposition is too narrow |
| **custom_id** | Identifier used by the Batch API to match requests with results |

---

## D

| Term | Definition |
|------|------|
| **Dynamic Task Decomposition** | Adaptive generation of subtasks based on intermediate discoveries; suitable for legacy systems or open-ended research |

---

## E

| Term | Definition |
|------|------|
| **Edit** | Built-in Claude Code tool for localized file modifications using a unique text anchor; fails when the anchor is not unique |
| **end_turn** | `stop_reason` value indicating the model has completed the current task and the loop should terminate |
| **enum** | JSON Schema constraint that restricts a field to one of a predefined set of values |
| **Escalation** | Process of transferring an issue from an agent to a human; should trigger immediately when the user explicitly requests a human |
| **errorCategory** | Field in structured errors that identifies the error type: `transient`/`validation`/`permission`/`business` |
| **Explore Subagent** | Subagent pattern that isolates detailed discovery output while preserving the main agent's context |

---

## F

| Term | Definition |
|------|------|
| **Few-Shot Prompting** | Including a small number of examples (usually 2-4) in the prompt to guide stable output format; preferred over adding more instructions when formatting is unstable |
| **fork_session** | Creates an independent session branch from a shared baseline to explore different implementation options in parallel |

---

## G

| Term | Definition |
|------|------|
| **Glob** | Built-in Claude Code tool for finding files by **path/name pattern** (mnemonic: Glob searches paths) |
| **Grep** | Built-in Claude Code tool for searching patterns in file **contents** (mnemonic: Grep searches contents) |

---

## H

| Term | Definition |
|------|------|
| **Handoff** | Structured transfer when escalating to a human; should include customer ID, root cause, actions attempted, and recommended action |
| **Hooks** | Code executed when specific events occur: `PostToolUse` (normalization), `PreToolUse` (interception/blocking) |
| **Hub-and-Spoke** | Architecture where a coordinator manages all subagent communication; subagents do not communicate directly with each other |
| **Hierarchical Summarization** | Two-stage compression strategy: first summarize each section or document independently, then combine those section-level summaries into a final synthesis. Preserves more structure and source traceability than single-pass compression. |
| **Human-in-the-Loop** | Design pattern that adds human review at critical points in an automated workflow |

---

## I

| Term | Definition |
|------|------|
| **Interview Pattern** | Have Claude ask questions first to expose design constraints before implementation; useful for complex designs such as caching strategies and data migrations |
| **isError** | Boolean field in MCP tool responses that marks operation failure (`true` = failed, `false` or missing = successful) |
| **isRetryable** | Error-response field indicating whether the error is worth retrying (`true` for transient errors, `false` for permission/business errors) |

---

## J

| Term | Definition |
|------|------|
| **JSON Schema** | Specification for defining and validating JSON data structures; `tool_use` + Schema can guarantee structured output |

---

## L

| Term | Definition |
|------|------|
| **Lost in the Middle** | Effect where models handle the beginning and end of long inputs more reliably while missing information in the middle; important information should be placed at the front or back |

---

## M

| Term | Definition |
|------|------|
| **Manifest** | State file for long-running tasks that records completed/pending modules and discovery files for crash recovery |
| **max_tokens** | API parameter for maximum generated tokens; `stop_reason` of `"max_tokens"` means the response was truncated, not that the task is complete |
| **MCP (Model Context Protocol)** | Standardized protocol created by Anthropic for tool and resource interfaces |
| **MCP Resources** | Read-only content catalogs (issue lists, documentation trees, database schemas) that reduce exploratory tool calls |
| **MCP Server** | Service that implements the MCP protocol and provides tools/resources; supports `stdio` (local) and SSE (remote) transport types |
| **MCP Tools** | Callable operations exposed through the MCP protocol, listed in the tool pool alongside built-in tools |
| **messages** | Conversation history array sent to the API, containing `user`, `assistant`, and `tool_result` messages |
| **multi-agent** | System architecture in which multiple Claude instances collaborate to complete a task |

---

## N

| Term | Definition |
|------|------|
| **nullable** | JSON Schema pattern where `"type": ["string", "null"]` allows a field to be null, preventing the model from inventing missing information |

---

## O

| Term | Definition |
|------|------|
| **Orchestrator** | Synonym for Coordinator; responsible for task decomposition, subagent scheduling, and result aggregation |
| **Overlapping Chunks** | Document chunking strategy that keeps partial overlap between adjacent chunks to prevent information from being cut at boundaries |

---

## P

| Term | Definition |
|------|------|
| **Partial Results** | Incomplete data obtained before a subagent failure; should be reported to the coordinator together with the structured error |
| **paths** | `.claude/rules/` frontmatter field that uses glob patterns to apply rules only to matching files |
| **Plan Mode** | Claude Code work mode that explores and plans before making changes; suitable for large changes and multi-option decisions |
| **PostToolUse** | Hook triggered after tool execution and before model processing; commonly used for data-format normalization (timestamp conversion, status-code mapping) |
| **Prefill** | Pre-populating the beginning of an assistant turn to guide the model into a specific format, such as `[` to start a JSON array |
| **PreToolUse** | Hook triggered before a tool call; can intercept/block by returning an error or validate prerequisites |
| **Programmatic Enforcement** | Implementing deterministic constraints with code/hooks; unlike prompt guidance, it has no non-zero failure rate |
| **Prompt Chaining** | Splitting a complex task into multiple sequential steps where each step's output becomes the next step's input |
| **pause_turn** | `stop_reason` value indicating a tool call was paused by an external system; call the resume interface after resolving the pause reason |

---

## R

| Term | Definition |
|------|------|
| **Read** | Built-in Claude Code tool for reading full file contents |
| **Retry-with-Error-Feedback** | Retry strategy that includes the specific validation error in the prompt, rather than a vague instruction like "try again", so the model can correct the issue |
| **--resume** | Session recovery parameter; `claude --resume <session-name>` resumes a named session when prior context is still valid |

---

## S

| Term | Definition |
|------|------|
| **Scaled Score** | Standardized exam score on a 100-1000 scale, with 720 as the passing score |
| **Scratchpad** | File where an agent persists key findings across context boundaries, preventing context degradation in long sessions |
| **SKILL.md** | Markdown file defining a Claude Code Skill; frontmatter controls `context: fork`, `allowed-tools`, and `argument-hint` |
| **Skills** | On-demand workflow templates (contrast with CLAUDE.md: rules that are always loaded) |
| **Slash Commands** | Custom commands beginning with `/`; project-level commands live in `.claude/commands/`, personal commands live in `~/.claude/commands/` |
| **SSE (Server-Sent Events)** | Remote MCP transport type for HTTP long-connection communication, contrasted with `stdio` for local processes |
| **stdio** | Local MCP transport type that communicates through a process's standard input/output |
| **stop_reason** | API response field: `"tool_use"` (continue with tool calls), `"end_turn"` (complete), `"max_tokens"` (truncated) |
| **stop_sequence** | `stop_reason` value indicating the response encountered a predefined stop string; handle follow-up according to business logic |
| **Stratified Sampling** | Evaluating accuracy by document type or field dimension instead of relying on an overall average that can hide local problems |
| **Structured Error Propagation** | Reporting `isError`, `errorCategory`, `isRetryable`, and Partial Results together to the coordinator when a subagent fails |
| **Subagent** | Specialized agent spawned by a coordinator; context is not inherited automatically and must be passed explicitly |
| **system prompt** | Special prompt set before the conversation that defines model role and constraints; stable rules go in system, variable tasks go in user |

---

## T

| Term | Definition |
|------|------|
| **Task tool** | Agent SDK tool for spawning subagents; `allowedTools` must include `"Task"` to use it |
| **TDD (Test-Driven Development)** | Development method where tests are written before implementation; failing tests provide precise feedback that is more accurate than natural-language descriptions |
| **temperature** | Parameter controlling output randomness; use `0` for structured extraction (most deterministic), and higher values for creative generation |
| **token** | Basic text-processing unit for models, roughly 4 English characters or 2 Chinese characters |
| **tool_choice** | `"auto"` (may choose not to call a tool) / `"any"` (must call some tool) / `{"type":"tool","name":"..."}` (force a specific tool) |
| **tool_result** | Tool execution result message type; must include `tool_use_id` and be appended to conversation history |
| **tool_use** | `stop_reason` value meaning the model is requesting a tool call; also refers to the tool-call block in response content |
| **tool_use_id** | Unique identifier for a tool call; `tool_result` uses `tool_use_id` to match the call and must match exactly |

---

## W

| Term | Definition |
|------|------|
| **WebFetch** | Built-in Claude Code tool that fetches content from a specified URL; used for external docs and API references |
| **WebSearch** | Built-in Claude Code tool that searches the web; suitable when information needs to be current or sources are unknown |
| **Write** | Built-in Claude Code tool that creates or overwrites files; fallback when Edit anchors are not unique |

---

## X

| Term | Definition |
|------|------|
| **XML Tags** | Prompt-engineering technique for clearly separating sections such as constraints, examples, and formats, e.g. `<report_categories>` |

---

## Configuration File Location Quick Reference

| File/Directory | Location | Version control | Purpose |
|----------|------|---------|------|
| `CLAUDE.md` (project-level) | `.claude/CLAUDE.md` or project root | Yes, commit | Team-shared rules |
| `CLAUDE.md` (user-level) | `~/.claude/CLAUDE.md` | No, do not share | Personal preferences |
| Project commands | `.claude/commands/` | Yes, commit | Team-shared commands |
| User commands | `~/.claude/commands/` | No, do not share | Personal commands |
| Path rules | `.claude/rules/` | Yes, commit | Rules by topic/path |
| Skills | `.claude/skills/` | Yes, commit | Reusable workflows |
| MCP (project-level) | `.mcp.json` | Yes, commit (use environment variables for secrets) | Team MCP servers |
| MCP (user-level) | `mcpServers` field in `~/.claude.json` | No, do not share | Personal MCP servers |

---

## stop_reason Quick Reference

| Value | Meaning | Loop action |
|----|------|---------|
| `"tool_use"` | Model requests a tool call | Execute tool -> append `tool_result` -> continue loop |
| `"end_turn"` | Task completed normally | Terminate loop and return final response |
| `"max_tokens"` | Response was truncated | Not complete; increase `max_tokens` or process in batches |
| `"stop_sequence"` | Predefined stop sequence encountered | Handle according to business logic |
| `"pause_turn"` | Tool call was externally paused | Resume after resolving the pause reason |

---

## Tool Selection Quick Reference (Grep vs Glob vs Others)

| Need | Tool |
|------|------|
| Find which files call a function | Grep (search contents) |
| Find all `*.test.tsx` files | Glob (path pattern) |
| Find TODO comments | Grep (search content keywords) |
| Find all files under `src/api/` | Glob (directory pattern) |
| Find files that import a module | Grep (search import statements) |
| Modify a specific code block in a file | Edit (unique text anchor) |
| When the Edit anchor is not unique | Read + Write (full rewrite) |
| Run tests/install dependencies | Bash |
