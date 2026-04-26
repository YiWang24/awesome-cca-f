[中文版本](README.zh.md)

# Awesome CCA-F

> A curated resource list and study kit for **Claude Certified Architect -
> Foundations (CCA-F)**.

This repository collects the best public resources for CCA-F preparation:
official Anthropic materials, domain guides, hands-on labs, practice questions,
open-source study projects, third-party references, and community exam reports.
It also includes a local curriculum for learners and instructors who want a
structured path instead of a loose bookmark list.

[![Claude Certified Architect access instructions](assets/cca-f-access-video-thumbnail.jpg)](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request)

[Request access to Claude Certified Architect - Foundations](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request)

[![Anthropic Academy courses](assets/anthropic-courses-og.png)](https://anthropic.skilljar.com/)

---

## Contents

- [What This Is](#what-this-is)
- [Start Here](#start-here)
- [Exam At A Glance](#exam-at-a-glance)
- [Repository Study Kit](#repository-study-kit)
- [Official Anthropic Resources](#official-anthropic-resources)
- [Domain Resources](#domain-resources)
- [Open-Source Study Projects](#open-source-study-projects)
- [Practice Questions And Mock Exams](#practice-questions-and-mock-exams)
- [Claude Code And Agent Engineering](#claude-code-and-agent-engineering)
- [Third-Party Guides And Study Plans](#third-party-guides-and-study-plans)
- [Community Reports](#community-reports)
- [Study Paths](#study-paths)
- [How To Use External Resources](#how-to-use-external-resources)
- [Maintenance Principles](#maintenance-principles)

---

## What This Is

CCA-F is not mainly a memorization exam. It tests architecture judgment around
Claude systems: agent orchestration, tool design, Claude Code workflows,
structured output, context management, reliability, and human escalation.

This project is organized like an awesome list, with one difference: it also
contains original local study material under `domains/`, `labs/`, `practice/`,
and `reference/`.

| Audience | Use This Repository To |
| --- | --- |
| Individual candidates | Build a study plan, practice scenario questions, and review weak domains |
| Instructors | Run workshops using domain notes, labs, and practice questions |
| Engineering teams | Convert exam ideas into team rules, hooks, tool schemas, and CI workflows |

### Resource Labels

| Label | Meaning | How To Treat It |
| --- | --- | --- |
| Official | Published by Anthropic | Source of truth for scope and product behavior |
| Local | Included in this repository | Course-ready notes, labs, and practice material |
| Open source | Public GitHub or Gist material | Useful, but verify answers and claims |
| Third-party | Commercial or independent material | Good for extra practice, not authoritative |
| Community | Reddit or public experience post | Useful for exam feel and blind spots |

---

## Start Here

If you only have limited time, use these first.

| Priority | Resource | Why It Matters |
| --- | --- | --- |
| 1 | [Official CCA-F exam guide PDF](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F8lsy243ftffjjy1cx9lm3o2bw%2Fpublic%2F1773274827%2FClaude+Certified+Architect+%E2%80%93+Foundations+Certification+Exam+Guide.pdf) | Confirms domains, task statements, scenarios, and official samples |
| 2 | [CCA-F access request](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request) | Entry point for certification access |
| 3 | [Building effective agents](https://www.anthropic.com/research/building-effective-agents) | Core background for agentic architecture and tool use |
| 4 | [Best practices for Claude Code](https://www.anthropic.com/engineering/claude-code-best-practices) | Practical foundation for Claude Code workflow questions |
| 5 | [Local domain guides](en/domains/) | Structured notes by exam domain |
| 6 | [Local practice questions](en/practice/) | Domain-based practice with explanations |
| 7 | [Official sample analysis](en/practice/official-sample-questions.md) | Calibrate against official sample style |

---

## Exam At A Glance

| Item | Description |
| --- | --- |
| Certification | Claude Certified Architect - Foundations |
| Question format | Single-choice scenario questions |
| Scoring | No penalty for incorrect answers |
| Score range | 100-1000 scaled score |
| Passing score | **720** |
| Source priority | Official exam guide > Anthropic Docs > local notes > third-party resources |

> If Anthropic updates the exam guide, access policy, scoring, or domain
> weights, follow the current official guidance.

### Domain Weights

```text
Domain 1: Agentic Architecture & Orchestration    ██████████████ 27%
Domain 2: Tool Design & MCP Integration           █████████      18%
Domain 3: Claude Code Configuration & Workflows   ██████████     20%
Domain 4: Prompt Engineering & Structured Output  ██████████     20%
Domain 5: Context Management & Reliability        ███████        15%
```

Recommended study priority: **D1 > D3 = D4 > D2 > D5**.

### Exam Scenarios

| # | Scenario | Main Domains | Typical Decision Points |
| --- | --- | --- | --- |
| 1 | Customer support resolution agent | D1, D2, D5 | Tool prerequisites, refund gates, escalation |
| 2 | Code generation with Claude Code | D3, D5 | CLAUDE.md, Plan Mode, path-specific rules |
| 3 | Multi-agent research system | D1, D2, D5 | Coordinator, subagent context, attribution |
| 4 | Claude-powered developer tool | D1, D2, D3 | Built-in tools, MCP, codebase exploration |
| 5 | Claude Code in CI/CD | D3, D4 | Non-interactive execution, JSON output, batch review |
| 6 | Structured data extraction | D4, D5 | Schema, validation, retry, low-confidence handling |

See [exam scenario notes](en/reference/exam-scenarios.md) for the local breakdown.

---

## Repository Study Kit

This repository is more than a link list. It contains a complete local study
kit that mirrors the five official domains.

```text
CCA-F/
├── README.md                         # Awesome CCA-F default README
├── README.zh.md                      # Chinese version
├── assets/                           # README images
├── domains/                          # Five domain deep dives
├── labs/                             # Jupyter notebooks
├── practice/                         # Practice questions and samples
└── reference/                        # Scenarios, tips, and glossary
```

### Local Curriculum

| Resource | Type | Use |
| --- | --- | --- |
| [Domain 1 guide](en/domains/01-agentic-architecture.md) | Local | Agent loops, subagents, hooks, handoffs, sessions |
| [Domain 2 guide](en/domains/02-tool-design-mcp.md) | Local | Tool design, MCP errors, tool allocation, built-in tools |
| [Domain 3 guide](en/domains/03-claude-code.md) | Local | CLAUDE.md, commands, skills, rules, CI/CD |
| [Domain 4 guide](en/domains/04-prompt-engineering.md) | Local | Prompt criteria, few-shot, schemas, validation, batches |
| [Domain 5 guide](en/domains/05-context-management.md) | Local | Long context, escalation, reliability, provenance |
| [Labs setup](en/labs/README.md) | Local | API key setup and notebook instructions |
| [Practice questions](en/practice/) | Local | Domain-based scenario practice |
| [Official sample analysis](en/practice/official-sample-questions.md) | Local | 12 official sample questions with explanations |
| [Exam tips](en/reference/exam-tips.md) | Local | Signal words, traps, and test-taking principles |
| [Glossary](en/reference/glossary.md) | Local | Bilingual terms and quick reference tables |

### Hands-On Labs

```bash
pip install anthropic python-dotenv jupyter
export ANTHROPIC_API_KEY="your_api_key_here"
jupyter notebook
```

| Notebook | Domain | Core Example |
| --- | --- | --- |
| [01-agentic-architecture.ipynb](en/labs/01-agentic-architecture.ipynb) | D1 | Agent loop, multi-agent orchestration, hooks |
| [02-tool-design-mcp.ipynb](en/labs/02-tool-design-mcp.ipynb) | D2 | Tool definitions, MCP integration, error handling |
| [03-claude-code.ipynb](en/labs/03-claude-code.ipynb) | D3 | CLAUDE.md structure and CI/CD integration |
| [04-prompt-engineering.ipynb](en/labs/04-prompt-engineering.ipynb) | D4 | JSON Schema output, few-shot, batch processing |
| [05-context-management.ipynb](en/labs/05-context-management.ipynb) | D5 | Long-document handling, context compression, error propagation |

---

## Official Anthropic Resources

### Certification And Academy

| Resource | Type | Use |
| --- | --- | --- |
| [CCA-F access request](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request) | Official | Request certification course and exam access |
| [Official CCA-F exam guide PDF](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2F8lsy243ftffjjy1cx9lm3o2bw%2Fpublic%2F1773274827%2FClaude+Certified+Architect+%E2%80%93+Foundations+Certification+Exam+Guide.pdf) | Official | Check official scope and sample questions |
| [Anthropic Academy](https://anthropic.skilljar.com/) | Official | Course catalog and certification learning paths |
| [Introduction to subagents](https://anthropic.skilljar.com/introduction-to-subagents) | Official course | Subagents and delegation |
| [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) | Official course | Claude Code basics |
| [Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action) | Official course | Claude Code workflow examples |
| [Building with the Claude API](https://anthropic.skilljar.com/claude-with-the-anthropic-api) | Official course | Messages API and tool use |
| [Introduction to MCP](https://anthropic.skilljar.com/introduction-to-model-context-protocol) | Official course | Model Context Protocol basics |
| [MCP Advanced Topics](https://anthropic.skilljar.com/model-context-protocol-advanced-topics) | Official course | Advanced MCP topics |
| [Introduction to agent skills](https://anthropic.skilljar.com/introduction-to-agent-skills) | Official course | Skill organization patterns |

### Official Docs

| Resource | Type | Use |
| --- | --- | --- |
| [Anthropic Docs](https://docs.anthropic.com/) | Official | Main documentation entry |
| [Get started with Claude](https://docs.anthropic.com/en/docs/quickstart) | Official | First API call |
| [Models overview](https://docs.anthropic.com/en/docs/models-overview) | Official | Model names, capabilities, and context |
| [Messages examples](https://docs.anthropic.com/en/api/messages-examples) | Official | Messages API request structure |
| [Tool use implementation](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use) | Official | `tool_use`, `tool_result`, and tool schemas |
| [Batch processing](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing) | Official | Offline batch jobs and large reviews |
| [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering) | Official | Prompting methods |
| [Long context prompting tips](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips) | Official | Multi-document and long-context prompting |
| [Context windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) | Official | Context limits and model capabilities |
| [Prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) | Official | Reduce repeated long-prompt cost and latency |
| [Token counting](https://docs.anthropic.com/en/docs/build-with-claude/token-counting) | Official | Estimate tokens before requests |
| [Citations](https://docs.anthropic.com/en/docs/build-with-claude/citations) | Official | Preserve traceability in multi-source answers |
| [Evaluation tool](https://docs.anthropic.com/en/docs/test-and-evaluate/eval-tool) | Official | Test prompts in the Console |
| [Create empirical evaluations](https://docs.anthropic.com/en/docs/test-and-evaluate/develop-tests) | Official | Build reproducible eval sets |

### Claude Code Docs

| Resource | Type | Use |
| --- | --- | --- |
| [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code/overview) | Official | Claude Code capabilities |
| [Claude Code quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart) | Official | Install, authenticate, and run basics |
| [CLI reference](https://docs.anthropic.com/en/docs/claude-code/cli-reference) | Official | `-p`, JSON output, resume, and other flags |
| [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings) | Official | Settings, permissions, tools, hierarchy |
| [Manage Claude's memory](https://docs.anthropic.com/en/docs/claude-code/memory) | Official | CLAUDE.md hierarchy and `/memory` |
| [Slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands) | Official | Built-in and custom commands |
| [Hooks guide](https://docs.anthropic.com/en/docs/claude-code/hooks-guide) | Official | Enforce workflow rules with hooks |
| [Hooks reference](https://docs.anthropic.com/en/docs/claude-code/hooks) | Official | PreToolUse, PostToolUse, and events |
| [Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents) | Official | Subagent config, tools, and context isolation |
| [Claude Code MCP](https://docs.anthropic.com/en/docs/claude-code/mcp) | Official | Connect MCP servers to Claude Code |

### Official Engineering And Research

| Resource | Type | Use |
| --- | --- | --- |
| [Claude Partner Network announcement](https://www.anthropic.com/news/claude-partner-network) | Official | CCA-F and Partner Network context |
| [Claude Partner Network](https://claude.com/partners) | Official | Partner, certification, and portal entry points |
| [Building effective agents](https://www.anthropic.com/research/building-effective-agents) | Official | Agent and workflow design principles |
| [Best practices for Claude Code](https://www.anthropic.com/engineering/claude-code-best-practices) | Official | Practical Claude Code usage |
| [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk/) | Official | Agent SDK, subagents, and tool loops |
| [Claude Code Advanced Patterns](https://www.anthropic.com/webinars/claude-code-advanced-patterns) | Official | Subagents, MCP, and large-codebase workflows |
| [Claude Code plugins](https://www.anthropic.com/news/claude-code-plugins) | Official | Skills, hooks, subagents, and MCP packaging |

---

## Domain Resources

### D1: Agentic Architecture And Orchestration

| Resource | Type | Use |
| --- | --- | --- |
| [Local D1 guide](en/domains/01-agentic-architecture.md) | Local | Full local notes for the highest-weight domain |
| [Building effective agents](https://www.anthropic.com/research/building-effective-agents) | Official | Workflow and agent design patterns |
| [Introduction to subagents](https://anthropic.skilljar.com/introduction-to-subagents) | Official course | Coordinator and subagent patterns |
| [Building agents with the Claude Agent SDK](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk/) | Official | Agent SDK concepts and examples |
| [D1 practice questions](en/practice/domain1-questions.md) | Local | Domain-specific practice |

### D2: Tool Design And MCP Integration

| Resource | Type | Use |
| --- | --- | --- |
| [Local D2 guide](en/domains/02-tool-design-mcp.md) | Local | Tool descriptions, errors, scope, and built-in tools |
| [Tool use implementation](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use) | Official | Tool schema and tool-result mechanics |
| [Model Context Protocol](https://docs.anthropic.com/en/docs/mcp) | Official | MCP concepts and protocol entry |
| [MCP connector](https://docs.anthropic.com/en/docs/agents-and-tools/mcp-connector) | Official | Remote MCP servers through Messages API |
| [D2 practice questions](en/practice/domain2-questions.md) | Local | Domain-specific practice |

### D3: Claude Code Configuration And Workflows

| Resource | Type | Use |
| --- | --- | --- |
| [Local D3 guide](en/domains/03-claude-code.md) | Local | CLAUDE.md, commands, skills, hooks, CI/CD |
| [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code/overview) | Official | Product capabilities and mental model |
| [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings) | Official | Settings and permission hierarchy |
| [Hooks reference](https://docs.anthropic.com/en/docs/claude-code/hooks) | Official | Hook events and enforcement patterns |
| [D3 practice questions](en/practice/domain3-questions.md) | Local | Domain-specific practice |

### D4: Prompt Engineering And Structured Output

| Resource | Type | Use |
| --- | --- | --- |
| [Local D4 guide](en/domains/04-prompt-engineering.md) | Local | Criteria, few-shot, schemas, validation, batch |
| [Prompt engineering overview](https://docs.anthropic.com/en/docs/prompt-engineering) | Official | Prompting methods |
| [Prompt templates and variables](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-templates-and-variables) | Official | Separate fixed instructions from variables |
| [Prompt improver](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/prompt-improver) | Official | Improve complex prompts |
| [D4 practice questions](en/practice/domain4-questions.md) | Local | Domain-specific practice |

### D5: Context Management And Reliability

| Resource | Type | Use |
| --- | --- | --- |
| [Local D5 guide](en/domains/05-context-management.md) | Local | Context preservation, escalation, reliability |
| [Long context prompting tips](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/long-context-tips) | Official | Long-document organization |
| [Context windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) | Official | Context limits and model capabilities |
| [Citations](https://docs.anthropic.com/en/docs/build-with-claude/citations) | Official | Source attribution and traceability |
| [D5 practice questions](en/practice/domain5-questions.md) | Local | Domain-specific practice |

---

## Open-Source Study Projects

| Resource | Type | Why It Is Useful |
| --- | --- | --- |
| [paullarionov/claude-certified-architect](https://github.com/paullarionov/claude-certified-architect) | Open source | Multilingual CCA-F guide and PDFs |
| [English guide](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_en.MD) | Open source | Long-form English study guide |
| [Chinese guide](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_zh.md) | Open source | Chinese study guide and terminology |
| [aderegil/claude-certified-architect](https://github.com/aderegil/claude-certified-architect) | Open source | Guided labs for 6 scenarios, 5 domains, and 30 tasks |
| [CCA Foundations Mock Exam](https://github.com/neerajkr7/cca-foundations-exam-practice) | Open source | 100-question static mock exam project |
| [Mock Exam live site](https://neerajkr7.github.io/cca-foundations-exam-practice/) | Open source | Free online practice without login or API key |
| [Architect Cert MCP](https://github.com/Connectry-io/connectrylab-architect-cert-mcp) | Open source | MCP study tool with questions and spaced repetition |
| [Architect Cert overview](https://mcpmarket.com/server/architect-cert) | Open source | Summary and installation information for the MCP tool |
| [Agent Engineering Skill](https://gist.github.com/antoniopresto/3e0b9849339170075c8f82bb61b75255) | Open source | CCA-F domains translated into a Claude Code skill |

---

## Practice Questions And Mock Exams

### Local Practice

| Resource | Type | Use |
| --- | --- | --- |
| [Official sample analysis](en/practice/official-sample-questions.md) | Local | 12 official-style sample explanations |
| [Domain 1 questions](en/practice/domain1-questions.md) | Local | Agentic architecture practice |
| [Domain 2 questions](en/practice/domain2-questions.md) | Local | Tool design and MCP practice |
| [Domain 3 questions](en/practice/domain3-questions.md) | Local | Claude Code workflow practice |
| [Domain 4 questions](en/practice/domain4-questions.md) | Local | Prompting and structured-output practice |
| [Domain 5 questions](en/practice/domain5-questions.md) | Local | Context and reliability practice |

### External Practice

| Resource | Type | Use |
| --- | --- | --- |
| [Scrolly CCA-F Study Guide](https://www.scrolly.to/s/ccaf-study-guide) | Third-party | Interactive guide with practice questions |
| [Claude Certification Guide mock exam](https://claudecertificationguide.com/mock-exam) | Third-party | Free full mock exam |
| [ReadRoost practice questions](https://readroo.st/blog/cca-foundations-practice-questions) | Third-party | Extra scenario-question practice |
| [CCA sample questions PDF](https://claudecertified.com/downloads/cca-sample-5q.pdf) | Third-party | Small third-party sample PDF |
| [Panaversity CCA-F page](https://panaversity.org/certifications/exams/CCA-F) | Third-party | Domain and scenario overview |
| [Claude Certified Architect Practice](https://www.claudecertifiedarchitect.dev/) | Third-party | Scenario questions and domain diagnostics |
| [Claude Certified Architects Prep](https://www.claudecertifiedarchitects.com/) | Third-party | Course-based prep and full-length practice exam |
| [ClaudeCertified practice questions](https://claudecertified.com/cca-practice-questions) | Third-party | Practice-question PDFs |
| [Claude Architect Lab](https://www.anthropiccertifications.com/) | Third-party | Adaptive practice, mock exam, and tutor |
| [Udemy practice exams](https://www.udemy.com/course/claude-certified-architect-foundations-practice-tests-2026/) | Third-party | Paid practice exams |
| [Udemy 360 questions](https://www.udemy.com/course/claude-certified-architect-foundations-practice-tests-u/) | Third-party | Paid multi-test practice set |
| [Udemy mock exam with explanation](https://www.udemy.com/course/claude-certified-architect-mock-exam-with-answer-explanation/) | Third-party | Paid mock exam with explanations |

---

## Claude Code And Agent Engineering

These are not CCA-F-specific in every case, but they are useful for the
engineering concepts tested by the exam.

| Resource | Type | Use |
| --- | --- | --- |
| [Claude Code Ultimate Guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/core/architecture.md) | Open source | Claude Code loop, tools, context, and sessions |
| [Claude Code Architecture Deep Dive](https://gist.github.com/yanchuk/0c47dd351c2805236e44ec3935e9095d) | Open source | Notes on Claude Code internals |
| [Claude Code Infrastructure Showcase](https://github.com/diet103/claude-code-infrastructure-showcase) | Open source | Project-level skills, hooks, and agents |
| [Awesome Claude Code Agents](https://github.com/supatest-ai/awesome-claude-code-sub-agents) | Open source | Subagent role and description examples |
| [Claude Code Unified Agents](https://github.com/stretchcloud/claude-code-unified-agents) | Open source | Multi-agent organization patterns |
| [AI Software Architect](https://github.com/codenamev/ai-software-architect) | Open source | ADRs, architecture reviews, assistant workflows |
| [ccsetup](https://github.com/MrMarciaOng/ccsetup) | Open source | Claude Code project setup and agent orchestration |
| [cc-skills](https://github.com/terrylica/cc-skills) | Open source | Skills, plugins, and hooks organization examples |
| [Prompt Architect](https://github.com/ckelsoe/prompt-architect) | Open source | Prompt framework and D4-style refinement practice |

---

## Third-Party Guides And Study Plans

| Resource | Type | Use |
| --- | --- | --- |
| [Claude Certified Architect Exam Guide 2026](https://www.claudecertifiedarchitects.com/blog/cca-foundations-exam-guide-2026/) | Third-party | Exam format, domains, timeline, sample explanations |
| [Preporato Complete Guide](https://preporato.com/blog/claude-certified-architect-complete-guide-2026) | Third-party | Domain checklist and anti-pattern review |
| [AI.cc CCA-F guide](https://www.ai.cc/blogs/claude-certified-architect-foundations-cca-f-exam-guide-2026/) | Third-party | Certification background, structure, access notes |
| [ThinkSmart complete guide](https://thinksmart.life/research/posts/claude-certified-architect-study-guide/) | Third-party | Five domains, six scenarios, sample overview |
| [AI Productivity prep guide](https://aiproductivity.ai/blog/claude-certified-architect-guide/) | Third-party | Study advice and resource roundup |
| [Vorantis CCA guide](https://www.vorantis.co/blog/claude-certified-architect-cca-guide) | Third-party | Certification positioning and study path |
| [DataStudios access analysis](https://www.datastudios.org/post/claude-certified-architect-foundations-what-it-is-who-it-is-for-how-it-fits-anthropic-s-partner) | Third-party | Partner access and availability boundaries |
| [Sundeep Teki career angle](https://www.sundeepteki.org/advice) | Third-party | Career framing for FDE and enterprise AI delivery |
| [Finstor 14-day study plan](https://www.finstor.in/wp-content/uploads/2026/03/CCA-F_14Day_Study_Plan.pdf) | Third-party | Example 14-day study schedule |
| [CertStud study notes](https://certstud.com/pdfs/anthropic-claude-architect-study-notes.pdf) | Third-party | Supplemental final-review notes |
| [Unofficial handbook](https://rominur.com/Claude-Code-Architect-Handbook-2026.pdf) | Third-party | Unofficial long-form handbook |

---

## Community Reports

Community posts are useful for exam feel, but they are not authoritative. Use
them to identify blind spots and then verify against official material.

| Resource | Type | Use |
| --- | --- | --- |
| [Notes from studying the exam guide](https://www.reddit.com/r/ClaudeCode/comments/1s3hbv3/notes_from_studying_the_claude_certified/) | Community | How another learner broke down the guide |
| [Real exam has more scenarios](https://www.reddit.com/r/ClaudeAI/comments/1s34iyl/ccaf_the_real_exam_has_more_scenarios_than_the/) | Community | Possible differences between guide and live exam |
| [Passed with 893/1000](https://www.reddit.com/r/ClaudeAI/comments/1sgn0cf/passed_anthropics_claude_certified_architect/) | Community | Passing candidate retrospective |
| [Passed with 985/1000](https://www.reddit.com/r/ClaudeAI/comments/1ruf70b/just_passed_the_new_claude_certified_architect/) | Community | High-score feedback and mock-exam links |
| [Free 100-question mock exam](https://www.reddit.com/r/ClaudeAI/comments/1skbf1i/i_built_a_free_100question_cca_foundations_mock/) | Community | Discussion around the open-source mock exam |
| [Labs for CCA-F](https://www.reddit.com/r/ClaudeAI/comments/1sip8jd/labs_for_claude_certified_architect_foundations/) | Community | Discussion around guided labs |
| [Q&A prep guide](https://www.reddit.com/r/ClaudeAI/comments/1sb37sf/i_made_a_qa_prep_guide_for_the_anthropic_claude/) | Community | Q&A-style review material |
| [Anki deck thread](https://www.reddit.com/r/ClaudeAI/comments/1sp9nz0/claude_certified_architect_foundations/) | Community | Spaced-repetition card resource lead |
| [Anthropic subreddit Anki thread](https://www.reddit.com/r/Anthropic/comments/1sq0fut/claude_certified_architect_foundations/) | Community | Backup discussion for Anki materials |
| [Any review on CCA?](https://www.reddit.com/r/ClaudeAI/comments/1rv5t3r/any_review_on_claude_certified_architect/) | Community | Difficulty, controversy, and exam feel |

---

## Study Paths

### 14-Day Standard Plan

| Days | Goal | Local Material | Output |
| --- | --- | --- | --- |
| Day 1-3 | D1 | D1 guide, Lab 01, D1 questions | Agent loop vs subagent notes |
| Day 4-6 | D3 | D3 guide, Lab 03, D3 questions | CLAUDE.md and CI/CD checklist |
| Day 7-9 | D4 | D4 guide, Lab 04, D4 questions | Structured output template |
| Day 10-11 | D2 | D2 guide, Lab 02, D2 questions | Tool and MCP error template |
| Day 12 | D5 | D5 guide, Lab 05, D5 questions | Context reliability checklist |
| Day 13 | Samples | Official sample analysis | Mistake distribution table |
| Day 14 | Review | Glossary, exam tips, mistake log | Personal cheat sheet |

### 7-Day Sprint

1. Day 1: Map D1 and D3 core mechanisms.
2. Day 2: Review D4, especially schema, few-shot, temperature, and batch.
3. Day 3: Review high-frequency traps in D2 and D5.
4. Day 4-5: Practice by domain and tag mistakes by task statement.
5. Day 6: Rework official samples and all six scenarios.
6. Day 7: Review only mistakes, glossary, and cheat sheets.

### Practice Review Template

```md
## Question / Source

- Domain:
- Signal words:
- My answer:
- Correct answer:
- Mistake tag:
- Why the correct answer is more engineering-oriented:
- What signal should I notice next time:
```

Common mistake tags:

| Tag | Meaning |
| --- | --- |
| Concept | You did not know the mechanism |
| Boundary | You knew the concept but used it in the wrong scenario |
| Reading | You missed "first step", "best", or "root cause" |
| Engineering | You chose something workable but unreliable |
| Terminology | English and local terms were misaligned |

---

## How To Use External Resources

1. Confirm scope with the official exam guide and Anthropic Docs.
2. Use this repository's domain guides and labs to build a working structure.
3. Use open-source guides to compare explanations and fill gaps.
4. Use mock exams to train scenario judgment, not to memorize answers.
5. Treat third-party answers as hypotheses until verified against official docs.
6. Use community reports to discover blind spots, not as factual authority.

Core answer-selection heuristics:

```text
High-risk rule      -> programmatic enforcement before prompt guidance
Wrong tool choice   -> improve tool name, description, schema, and boundaries
Agent loop issue    -> inspect stop_reason and tool_result handling
Subagent gap        -> explicitly pass context and provenance
CI workflow         -> use non-interactive mode and structured output
Extraction failure  -> schema + validation + retry
Long context        -> preserve facts, state, source, and uncertainty
```

---

## Maintenance Principles

- Keep this project as a curated **Awesome CCA-F** resource list and local
  study kit.
- Separate official, local, open-source, third-party, and community resources.
- Prefer resources that improve architecture judgment, not just answer volume.
- When official materials change, update exam facts and domain mappings first.
- Do not present third-party questions, blogs, or Reddit posts as official
  guidance.
