# Official sample questions (12 questions)

> Source: CCA-F Official Exam Guide v0.1
> **Important Tip**: These 12 questions are official sample questions. The same or similar questions may appear in the exam. Be sure to understand the core test points of each question.

---

## Scenario 1: Customer support solution agent

> **Background**: Use Claude Agent SDK to build customer support agent, tools include `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`

---

### Q1 — Programmed constraints vs prompt guidance

Production data shows that in **12%** cases, the agent completely skips `get_customer` and only calls `lookup_order` based on the name verbally provided by the customer, occasionally leading to account misidentification and incorrect refunds. What modification would most effectively resolve this reliability issue?

A. Add programmatic precondition: block `lookup_order` and `process_refund` before `get_customer` returns the verified customer ID
B. Strengthen the instructions in the system prompt: Customer verification must be completed through `get_customer` before any order-related operations.
C. Add few-shot to show that the agent always calls `get_customer` first
D. Add a routing classifier to enable only a subset of appropriate tools based on request type

> **Answer: A**
> **Core test points**: Programmed constraints (deterministic) vs. prompt guidance (probabilistic)
> **Analysis**: When key business logic requires a fixed sequence, programmatic constraints can provide deterministic guarantees. B and C have only probabilistic compliance, have non-zero failure rates, and are insufficient to handle scenarios with financial consequences.

---

### Q2 — Tool description qualityThe logs show that when users ask questions about orders (such as "Help me check order #12345"), the agent often calls `get_customer` instead of `lookup_order`. Both tools have minimal descriptions and accept similar identifier formats. What is the most effective first step?

A. Include 5 to 8 few-shot examples in the system prompts
B. Expand the tool description and add input format, sample query, boundaries and applicable scenarios
C. Parse user input and pre-select tools before each round
D. Merged into a `lookup_entity` tool

> **Answer: B**
> **Core test point**: When the tool selection is wrong, the first step is to change the tool description
> **Analysis**: Tool description is the primary basis for tool selection, and the root cause is insufficient description quality. C (routing layer) is a heavyweight solution, not the first step.

---

### Q3 — Upgrade Calibration

The agent's first contact resolution rate is only **55%**, well below the 80% goal. The logs show that it escalates some otherwise simple situations while trying to autonomously handle complex situations that require policy exceptions. What is the **most effective way** to improve upgrade calibration?

A. Add clear upgrade criteria to the system prompt, and use a few-shot to show when to upgrade and when to handle it independently.
B. Let the agent first report a `1-10` confidence score. If it is lower than the threshold, it will automatically switch to artificial intelligence.
C. Train a separate classifier to predict which requests need to be upgraded
D. Use sentiment analysis to automatically upgrade when negative emotions exceed the threshold.

> **Answer: A**
> **Core test points**: Upgrade standards must be clear. Confidence scores and sentiment analysis are not reliable upgrade indicators.
> **Analysis**: The root cause is that the upgrade boundary is unclear. The most direct correction is to add clear standards and examples.

---## Scenario 2: Use Claude Code for code generation

---

### Q4 — Slash command position

You'll create a custom `/review` command that runs the team's standard code review checklist, automatically available to all developers who clone the repository. Where should this command file be placed?

A. `.claude/commands/` in the project repository
B. Each developer’s local `~/.claude/commands/`
C. `CLAUDE.md` of the project root directory
D. `.claude/config.json`

> **Answer: A**
> **Core test points**: Project-level commands vs user-level commands
> **Explanation**: `.claude/commands/` is shared to all team members through version control; `~/.claude/commands/` is for personal use only.

---

### Q5 — Plan Mode Judgment

You've been asked to split your team's monolithic application into microservices. This affects dozens of files and involves decisions about service boundaries and module dependencies. What should you do?

A. Enter Plan Mode, first explore the code base, understand dependencies, design an implementation plan, and then start modifying
B. Execute directly, and let the implementation expose its natural boundaries while making changes.
C. Execute directly, but give a long paragraph of super detailed upfront instructions
D. Execute directly first, then switch to Plan Mode when encountering complexity

> **Answer: A**
> **Core test point**: Large-scale architecture changes must first enter Plan Mode
> **Analysis**: Affects dozens of files, involves architectural decisions → Typical Plan Mode scenario.

---

### Q6 — Path-specific rulesThere are different areas of the code base that use different conventions: React components, API handlers, database models. Test files are scattered everywhere, but they should all comply with unified testing specifications. What's the most maintainable way to have Claude **automatically apply the correct rules**?

A. Create a rule file with YAML frontmatter in `.claude/rules/`, and use glob mode to match by path
B. Write all specifications in the root `CLAUDE.md` and let Claude infer it himself
C. Create skills for each code type
D. Put one `CLAUDE.md` in each subdirectory

> **Answer: A**
> **Core test point**: Test files are scattered everywhere → Use glob mode instead of directory-level CLAUDE.md
> **Parsing**: The glob pattern of the `**/*.test.tsx` class can be matched across directories, and the directory-level CLAUDE.md cannot handle this situation.

---

## Scenario 3: Multi-agent research system

---

### Q7 — Task decomposition is too narrow

The research topic is "The Impact of AI on Creative Industries". Each sub-agent ran normally, but the final report only covered visual arts, completely missing music, writing, and film production. The coordinator split the tasks into: AI in digital art creation, AI in graphic design, and AI in photography. What is the **most likely root cause**?

A. The comprehensive sub-agent will not identify the coverage gap.
B. The coordinator’s tasks are too narrowly broken down, resulting in subtasks that do not cover all relevant areas of the topic.
C. The web search sub-agent query is not comprehensive enough
D. The document analysis sub-agent filters non-visual art sources

> **Answer: B**
> **Core test point**: Incomplete coverage of multi-agent systems — the coordinator’s task decomposition is too narrow.
> **Analysis**: The problem lies with the coordinator, not the sub-agents. The root cause is that the task decomposition at the coordination layer did not cover all required content types.

---

### Q8 — Structured Error Propagation

The web search subagent times out when researching complex topics. You need to design how this failure information is returned to the coordinator. Which method of error propagation is most conducive to intelligent recovery?

A. Return structured error context: failure type, attempted query, partial results, potential alternatives
B. The sub-agent internally performs automatic exponential backoff retries, and only returns to a general state after exhaustion.
C. Swallow the timeout and return an empty result and mark success
D. Throw the exception directly to the top level and terminate the entire process

> **Answer: A**
> **Core test point**: Error propagation must contain enough information for the coordinator to make recovery decisions
> **Parsing**: Structured error context (failure type + attempted query + partial results + alternatives) allows the coordinator to choose to retry, modify the strategy, or continue with partial results.

---

### Q9 — Tool Scope Optimization

The synthesis sub-agent often needs to check simple facts when integrating results. Now control must be returned to the coordinator every time, and then transferred to the search sub-agent, resulting in 2-3 round trips and a 40% increase in latency. The assessment found that **85%** were simple fact checks and **15%** required in-depth research. What is the best optimization solution?

A. Give the comprehensive sub-agent a restricted `verify_fact` tool to handle simple verification, and complex verification is still delegated by the coordinator
B. The comprehensive sub-agent collects all verification requirements and sends them back to the coordinator together at the end.
C. Give the comprehensive sub-agent all search tools
D. Let the search sub-agent cache more context from the beginning> **Answer: A**
> **Core test points**: Restricted tools + routing based on complexity, not all tools or no tools
> **Analysis**: Give the comprehensive sub-agent a restricted `verify_fact` tool (85% simple processing), and complex verification (15%) still takes the coordinator delegation path.

---

## Scenario 4: Claude Code in CI/CD

---

### Q10 — Non-interactive mode

Your pipeline runs:
```bash
claude "Analyze this pull request for security issues"
```
As a result, the job hangs indefinitely, and the log shows that Claude Code is waiting for interactive input. What is the correct approach?

A. Add `-p`: `claude -p "Analyze this pull request for security issues"`
B. Set environment variable `CLAUDE_HEADLESS=true`
C. Redirect stdin to `/dev/null`
D. Add `--batch`

> **Answer: A**
> **Core test point**: Claude Code in CI must use the `-p` mark> **Analysis**: `-p` (`--print`) allows Claude Code to run in non-interactive mode and exit after outputting the results without waiting for user input.

---

### Q11 — Batch API Applicability

The team wants to save on API costs. There are currently two workflows:
1. **Pre-merge blocking check**: must be completed before the developer merges
2. **Nightly Technical Debt Report**: for review the next morning

The manager recommends switching entirely to the Message Batches API (50% cost savings). How do you evaluate that?

A. Only change the technical debt report to batch, and continue to use real-time calls for pre-merge checks.
B. Change the batch of both and poll the status
C. Both retain real-time calls
D. Change the batch for both, and fallback when it is too slow.

> **Answer: A**
> **Core test point**: Batch API is not suitable for blocking workflows
> **Analysis**: The Batch API takes up to 24 hours to process and is not suitable for blocking the merge process. Technical debt reporting has high latency tolerance and is suitable for batch processing.

---

### Q12 — Large PR review structure

One PR modified 14 files in the inventory module. Appearances in a single full review: Some files have detailed feedback, while others are very shallow; obvious bugs are missed; the same model gets contradictory judgments in different files. How should the review process be restructured?

A. Split into multiple rounds: analyze local issues file by file, and then do a cross-file integrated analysis
B. Ask the developer to split the large PR into small PRs of 3-4 files
C. Change to a model with a larger context and continue the review at once
D. Run the entire PR 3 times and only retain problems that appear at least 2 times

> **Answer: A**
> **Core test point**: Large code reviews should be conducted file by file first and then across files to avoid dilution of attention.
> **Parse**: Split large PR reviews into two tiers: (1) file-by-file partial analysis, (2) separate cross-file integrated analysis, which is more reliable than a full review.

---

## Summary of core rules of official sample questions

| Question number | Core test points | Key memory points |
|------|---------|-----------|
| Q1 | Programmatic vs Prompt | Has financial consequences → Must use programmatic constraints |
| Q2 | Quality of tool description | Wrong tool selection → Step 1: Change description |
| Q3 | Upgrade calibration | Confidence score/sentiment ≠ Reliable upgrade signal |
| Q4 | Command location | Team sharing → `.claude/commands/` |
| Q5 | Plan Mode | Large-scale architecture → Plan Mode takes precedence |
| Q6 | Path rules | Scattered files → glob pattern |
| Q7 | Task decomposition | Incomplete coverage → Let’s look at coordinator decomposition first |
| Q8 | Error propagation | Structured errors + partial results + alternatives |
| Q9 | Tool scope | Limited tools handle simple situations and escalate to complex situations |
| Q10 | CI mode | CI must use `-p` flag |
| Q11 | Batch API | Blocking tasks are not suitable for Batch |
| Q12 | Code review | Big PR → file by file + two layers across files |