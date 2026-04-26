# Exam Scenario Guide (6 Scenarios)

> The exam randomly selects **4** questions from the following 6 scenarios. Each scenario covers multiple domains.

---

## Scenario 1: Customer Support Resolution Agent

**Background:**  
Build a customer support resolution agent with the Claude Agent SDK. The agent handles highly ambiguous requests (returns, billing disputes, account issues), accesses backend systems through custom MCP tools, and aims for an 80%+ first-contact resolution rate while knowing when to escalate to a human.

**Available MCP tools:**
- `get_customer` - Verify customer identity
- `lookup_order` - Look up order details
- `process_refund` - Process a refund
- `escalate_to_human` - Escalate to a human

**Primary domains tested:** Domain 1, Domain 2, Domain 5

Key points:
- Customer identity must be verified before any refund (programmatic enforcement vs prompt guidance)
- `PostToolUse` hooks normalize data formats returned by different tools
- Structured handoff protocol (summary content when escalating to a human)
- Tool description clarity (distinguishing `get_customer` from `lookup_order`)

---

## Scenario 2: Code Generation with Claude Code

**Background:**  
Use Claude Code to accelerate software development, including code generation, refactoring, debugging, and documentation. Integrate it into the development workflow through custom slash commands and CLAUDE.md configuration, and understand when to use Plan Mode vs direct execution mode.

**Primary domains tested:** Domain 3, Domain 5

Key points:
- Scope and structure of CLAUDE.md configuration
- When Plan Mode is appropriate (preview before high-risk operations)
- How to define custom slash commands
- Hook trigger timing (`PostToolUse`, `PreToolUse`)

---

## Scenario 3: Multi-Agent Research System

**Background:**  
Build a multi-agent research system with the Claude Agent SDK. A coordinator agent delegates to specialized subagents: one searches the web, one analyzes documents, one synthesizes findings, and one generates the final report. The system researches a specific topic and produces a synthesized report with citations.

**Agent architecture:**
```text
Coordinator agent
+-- Web search subagent
+-- Document analysis subagent
+-- Findings synthesis subagent
`-- Report generation subagent
```

**Primary domains tested:** Domain 1, Domain 2, Domain 5

Key points:
- Overly narrow task decomposition causes incomplete coverage (official sample question 7)
- Subagent context isolation (context must be passed explicitly)
- Parallel vs serial subagent calls
- Structured context transfer (preserving source attribution)
- Context window management (long research reports)

---

## Scenario 4: Claude-Powered Developer Productivity Tool

**Background:**  
Build a developer productivity tool with the Claude Agent SDK. The agent helps engineers explore unfamiliar codebases, understand legacy systems, generate boilerplate code, and automate repetitive tasks. It uses built-in tools (Read, Write, Bash, Grep, Glob) and integrates MCP servers.

**Primary domains tested:** Domain 2, Domain 3, Domain 1

Key points:
- Appropriate use cases for built-in tools (Read/Write/Bash/Grep/Glob)
- MCP server configuration (`.mcp.json` vs `~/.claude.json`)
- Dynamic task decomposition (exploring legacy codebases)
- Session state management and recovery

---

## Scenario 5: Claude Code in CI/CD

**Background:**  
Integrate Claude Code into a CI/CD pipeline. The system runs automated code reviews, generates test cases, and provides feedback on PRs. Prompts must be designed to provide actionable feedback while minimizing false positives.

**Primary domains tested:** Domain 3, Domain 4

Key points:
- Prompt chaining design (step-by-step code review)
- Role of few-shot examples (showing the expected feedback format)
- JSON Schema output constraints (structured review results)
- Prompt design techniques for reducing false positives
- Claude Code configuration in non-interactive/headless mode

---

## Scenario 6: Structured Data Extraction System

**Background:**  
Build a structured data extraction system with Claude. The system extracts information from unstructured documents, validates output with JSON Schema, maintains high accuracy, and handles edge cases and downstream integrations gracefully.

**Primary domains tested:** Domain 4, Domain 5

Key points:
- JSON Schema definition and output constraints
- Few-shot example design (showing the extraction format)
- Edge case handling (missing fields, abnormal formats)
- Long-document chunking strategies
- Confidence labeling and human-review triggers

---

## Scenario-Domain Coverage Matrix

| Scenario | D1 | D2 | D3 | D4 | D5 |
|------|----|----|----|----|-----|
| 1. Customer support agent | ** | ** | - | - | * |
| 2. Claude Code code generation | - | - | ** | - | * |
| 3. Multi-agent research system | ** | * | - | - | ** |
| 4. Developer productivity tool | * | ** | * | - | - |
| 5. CI/CD integration | - | - | ** | ** | - |
| 6. Structured data extraction | - | - | - | ** | ** |

** = primary domain tested | * = secondary domain tested
