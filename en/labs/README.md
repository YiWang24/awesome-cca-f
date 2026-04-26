# Labs Hands-on Practice Environment Setup

> Each notebook corresponds to one exam domain. The current version is split by official Task: every Task has a short explanation and one focused code example.

---

## Environment Setup

### 1. Install dependencies

```bash
pip install jupyter
```

### 2. Set the API key (optional)

The examples use local simulations by default and do not require a real API key. If you extend them into real Claude API calls, create a `.env` file in the project root:

```
ANTHROPIC_API_KEY=your_api_key_here
```

Or set it through an environment variable:

```bash
export ANTHROPIC_API_KEY="your_api_key_here"
```

### 3. Start Jupyter

```bash
jupyter notebook
```

---

## Notebook List

| File | Domain | Core Examples |
|------|------------|---------|
| [01-agentic-architecture.ipynb](01-agentic-architecture.ipynb) | D1 (27%) | 7 Tasks: agent loop, multi-agent orchestration, hooks, session management |
| [02-tool-design-mcp.ipynb](02-tool-design-mcp.ipynb) | D2 (18%) | 5 Tasks: tool interfaces, MCP errors, `tool_choice`, built-in tools |
| [03-claude-code.ipynb](03-claude-code.ipynb) | D3 (20%) | 5 Tasks: `CLAUDE.md`, commands, rules, CLI modes, Skills |
| [04-prompt-engineering.ipynb](04-prompt-engineering.ipynb) | D4 (20%) | 6 Tasks: criteria, few-shot, Schema, validation retry, batch processing, multi-pass review |
| [05-context-management.ipynb](05-context-management.ipynb) | D5 (15%) | 6 Tasks: context preservation, escalation, error propagation, codebase exploration, human review, provenance |

---

## Cost Note

The examples do not call external APIs by default, so running all notebooks does not incur API cost.
