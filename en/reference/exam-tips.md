# Study Advice and Exam Scope

---

## Exam Mindset

CCA-F is not a memorization test. It is an architecture judgment test.

Distractors often "could work," but the correct answer is the one that is **most reliable, deterministic, and officially aligned for production**. The rough priority order is: deterministic > probabilistic, programmatic > prompt-guided, explicit > implicit.

---

## Five Core Answering Principles

**1. Programmatic > prompt-based (when determinism matters)**
"How do you guarantee a step always runs?" The answer is hooks or prerequisite gating, not few-shot prompting or system prompts.
Prompts have a non-zero failure rate. Audit, compliance, and financial operations cannot rely on probabilistic safeguards.

**2. stop_reason > text parsing (for loop control)**
Checking `stop_reason == "end_turn"` is the correct approach. Parsing response text for phrases like "task complete" is brittle and can break when the output wording varies.

**3. Tool descriptions > system prompts (when the wrong tool is selected)**
If an agent frequently routes to the wrong tool, first improve the tool description by adding input format, trigger scenarios, examples, and boundary conditions. Changing the system prompt is the second step.

**4. Explicit passing > implicit inheritance (multi-agent systems)**
Subagents do not automatically receive the coordinator's history. Context must be passed actively each time the Task tool is called.

**5. Dynamic decomposition > fixed pipelines (open-ended tasks)**
For exploring legacy codebases or analyzing unknown data structures, scout first and then plan. Do not hard-code every step in advance.

---

## Exam Signal Words -> Answer Direction

| Signal word / scenario | Correct direction | Eliminate |
|---|---|---|
| "deterministic", "compliance", "must do X before Y" | Hooks / prerequisite gating | few-shot, system prompt |
| "wrong tool selected", "frequent misrouting" | Improve tool description | Change system prompt, add few-shot |
| "infinite loop", "agent does not terminate" | Check `stop_reason`, ensure `tool_result` is appended correctly | Add system prompt limiting turns |
| "subagent lacks information" | Coordinator explicitly passes context | Subagent infers on its own |
| "legacy system", "unknown structure", "open-ended task" | Dynamic decomposition (explore first) | Fixed prompt chain |
| "multiple independent subtasks", "need acceleration" | Multiple Task calls in one response = parallel | Serial calls in sequence |
| "tool call succeeds but data formats differ" | Normalize with `PostToolUse` Hook | `PreToolUse` Hook |
| "block a category of tool calls" | Intercept with `PreToolUse` Hook | `PostToolUse` Hook |
| "run in CI/CD" | `--print` / `-p` flag + `--output-format json` | Interactive mode |
| "structured output fields are occasionally missing" | `required` + `nullable` constraints | Describe the format in the prompt |
| "trigger human review on low confidence" | Add `confidence` field + threshold check | Return the result directly |
| "multi-step, risky operation" | Confirm first in Plan Mode | Execute directly |
| "context is too long, too much history" | Summaries + hierarchical compression, or `/compact` | Truncate directly |
| "multi-agent handoff" | Structured summary rather than full history | Pass the entire message list |
| "resume work across sessions" | `--resume`, `fork_session` | Start a new session from scratch |
| "Batch request, latency acceptable" | Batch API (50% cost, up to 24h processing window) | Real-time API |
| "extract specific information from a long document" | Move relevant snippets forward + locate with XML tags | Put the entire document in directly |
| "MCP service runs as a local process" | `stdio` transport | SSE transport |
| "MCP service runs over remote HTTP" | SSE transport | `stdio` transport |
| "agent step output is needed by the next step" | Append output to message history | Let the agent remember it |
| "temperature 0 scenario" | Structured extraction, deterministic output | Creative writing |

---

## Study Priorities by Domain

### Domain 1 (27%) - Highest weight; master this first

- [ ] **Complete agent loop shape**: the five `stop_reason` values (`end_turn`, `tool_use`, `max_tokens`, `stop_sequence`, `pause_turn`) and their handling logic
- [ ] **`tool_result` message format**: must include `tool_use_id`, supports `is_error: true`, and must be appended to messages before continuing the request
- [ ] **Hub-and-Spoke**: the coordinator does not directly execute tasks; subagents have isolated permissions, and each subagent can access only the tools it needs
- [ ] **Task tool**: `allowedTools` must explicitly include `"Task"` to launch subagents; the `context` parameter passes context
- [ ] **Parallel vs serial**: Claude calling multiple Task tools in one response = parallel; sequential calls through the loop = serial
- [ ] **Hooks**: PreToolUse (intercept/block tool calls), PostToolUse (normalize output); configured in `.claude/hooks/`
- [ ] **Programmatic enforcement vs prompt guidance**: compliance/audit scenarios require programmatic enforcement because prompts can fail
- [ ] **Session recovery**: `--resume` (continue a specific session), `fork_session` (create a branch), new session + summary (isolation scenarios)
- [ ] **Stopping conditions**: `max_turns` limits, error threshold detection, `pause_turn` while waiting for human input

### Domain 2 (18%)

- [ ] **Tool description quality**: must clearly explain "when to use it", "input format", "trigger conditions", and "limitations"
- [ ] **Tool count**: reliability drops beyond roughly 10 tools; consider splitting into multiple specialized agents
- [ ] **`tool_choice`**: `auto` (default) / `any` (must use some tool) / `{"type":"tool","name":"..."}` to force a specific tool
- [ ] **`tool_use` block structure**: `id`, `name`, and `input`; accessed as `block.id`, `block.name`, `block.input`
- [ ] **Error classification**: four `errorCategory` values (`transient`, `validation`, `business`, `permission`) + `isRetryable`
- [ ] **MCP transport types**: `stdio` (local subprocess) vs SSE (remote HTTP); `stdio` does not cross the network
- [ ] **MCP configuration locations**: `.mcp.json` (project-level) vs `~/.claude.json` (user-level)
- [ ] **Partial Results pattern**: long-running tool calls return intermediate state to prevent timeout
- [ ] **JSON Schema constraints**: `required` vs optional fields, `nullable`, `enum`, and the `other+detail` pattern

### Domain 3 (20%) - Detail-heavy and easy to miss

- [ ] **Three CLAUDE.md scopes**: user-level (`~/.claude/CLAUDE.md`) > project-level (project root) > directory-level (`.claude/CLAUDE.md`)
- [ ] **CLAUDE.md content structure**: project overview, tech stack, directory structure, naming conventions, and prohibitions (`NEVER do X`)
- [ ] **`.claude/rules/`**: supports the `paths` field (glob matching) for fine-grained rules scoped to specific files/directories
- [ ] **Hook trigger timing**: `PreToolUse` (before tool calls), `PostToolUse` (after tool calls), `Stop` (session end)
- [ ] **Plan Mode**: entered with `/plan`, or by planning in the response followed by manual user confirmation; use before high-risk operations
- [ ] **Slash commands**: defined in `.claude/commands/`; the filename is the command name; Markdown format
- [ ] **CI/CD integration**: `-p`/`--print` (non-interactive mode), `--output-format json` (machine-readable), `--allowedTools` (permission allowlist), `--model` (model selection)
- [ ] **`/compact` command**: compresses context, preserves key content, and supports continuation of large tasks
- [ ] **`/memory` command**: views and diagnoses currently loaded CLAUDE.md content
- [ ] **`context: fork` in SKILL.md**: runs the subagent in isolated context and avoids polluting the main conversation
- [ ] **`--resume`**: resumes a specific session ID; `--headless`: no terminal interaction mode

### Domain 4 (20%) - Equal priority

- [ ] **XML tags**: separate documents, instructions, and examples (`<document>`, `<examples>`, `<output_format>`)
- [ ] **Prefill technique**: append `{"role":"assistant","content":"["}` as the final message to force Claude to start output with a specific symbol
- [ ] **Few-shot best practices**: 2-4 positive examples covering edge cases; positive/negative contrast examples work better
- [ ] **`temperature=0`**: structured extraction/deterministic scenarios; higher temperatures for creative tasks
- [ ] **JSON Schema structured output**: `properties`, `required`, `type`, `enum`, `nullable`, and `description` fields
- [ ] **Confidence labeling**: add a `confidence` field (0.0-1.0) to the schema; below-threshold values trigger human review
- [ ] **Batch API**: 50% cost, up to 24h processing window, no multi-turn tool calls, `custom_id` for tracking
- [ ] **Case Facts pattern**: move key facts to the front of the context to avoid Lost in the Middle
- [ ] **Prompt chaining design**: each step has a single responsibility; previous output becomes next input
- [ ] **System prompt vs user prompt**: persistent instructions (role, format, constraints) go in the system prompt; specific tasks go in the user message
- [ ] **Claim-Source Mapping**: attach a source to every claim in multi-source synthesis to track citation confidence

### Domain 5 (15%)

- [ ] **Context window size**: different Claude models have different windows (Claude 3.5 Sonnet is 200K, but not all Claude 3 models are 200K)
- [ ] **"Lost in the Middle" effect**: put key information at the beginning or end; information in the middle is most likely to be missed
- [ ] **Context priority order**: System prompt > early conversation > recent messages (put important content early)
- [ ] **Multi-agent handoff strategy**: pass structured summaries, not full message history; independent subagent contexts avoid cross-contamination
- [ ] **Long-document chunking**: use Overlapping Chunks to prevent information from being cut at chunk boundaries
- [ ] **Hierarchical summarization**: summarize sections first, then summarize the summaries; high compression while preserving structure
- [ ] **Manifest pattern**: when exploring large codebases, first build a file/module index, then read on demand
- [ ] **Scratchpad pattern**: the agent maintains a working notebook to accumulate findings across steps
- [ ] **Confidence stratification**: high-confidence results are returned directly; low-confidence results pause for human confirmation to prevent hallucination propagation
- [ ] **Stratified Sampling validation**: sample from different subsets to validate output quality rather than spot-checking only the full aggregate

---

## Officially Tested Scope

- Claude Agent SDK (multi-agent orchestration, subagent lifecycle)
- Claude Code (CLAUDE.md, Hooks, MCP integration, Plan Mode, slash commands)
- Claude API (tool use, structured output, message history management, Batch API)
- Model Context Protocol (tool and resource interface design, transport types)
- Prompt engineering (JSON Schema, few-shot prompting, structured extraction, temperature control)
- Context window management and long-document handling
- CI/CD integration patterns
- Error handling and human-review workflows

## Officially Out of Scope

- Claude model internals (neural network architecture, etc.)
- Model training, fine-tuning, and RLHF processes
- API details for other LLM providers
- Internal details of non-Anthropic LLM frameworks (LangChain, LlamaIndex, etc.)
- Low-level Python/JavaScript language features

---

## Exam Strategy

**Answer every question** - there is no penalty for wrong answers; blank answers are treated as incorrect.

**When unsure**: first eliminate clearly wrong options (usually 1-2 choices), then choose the most deterministic remaining option. If two options are both correct, choose the more direct and more officially recommended one.

**Pay attention to whether the question asks for the "first step" or the "best solution"** - these have different answers. "First step" usually means diagnosis/analysis; "best solution" means the complete fix.

**Time allocation**: the typical CCA-F format is 60 questions in 90 minutes, averaging 90 seconds per question. Answer the ones you know first, mark uncertain questions, and return to them later.

---

## Recommended Study Resources

| Resource | Description |
|------|------|
| [Anthropic Docs](https://docs.anthropic.com) | Official authoritative documentation |
| [Tool Use Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | Complete tool-use guide |
| [Multi-agent Systems](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/multi-agent) | Multi-agent systems |
| [Claude Code SDK](https://docs.anthropic.com/en/docs/claude-code/sdk) | Agent SDK documentation |
| [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) | Hooks configuration |
| [MCP Docs](https://docs.anthropic.com/en/docs/mcp) | Model Context Protocol |
| [Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) | Prompt engineering guide |
| [Batch API](https://docs.anthropic.com/en/docs/build-with-claude/batch) | Message Batches API |
