# Domain 3: Claude Code Configuration & Workflows

> **Weight: 20%**
> Official documentation: [Claude Code Overview](https://docs.anthropic.com/en/docs/claude-code/overview) | [CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/memory) | [Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

---

## Task Statement Coverage

| Task | Topic |
| ---- |------------------------------------------------ |
| 3.1 | Configuring CLAUDE.md with appropriate hierarchy, scope, and modularity |
| 3.2 | Create and configure custom slash commands and skills |
| 3.3 | Apply path-specific rules for conditional contract loading |
| 3.4 | Determining when to use Plan Mode vs direct execution |
| 3.5 | Apply iterative optimization techniques for incremental improvements |
| 3.6 | Integrating Claude Code into the CI/CD pipeline |

---

### Task Statement 3.1: Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization

#### Knowledge of:

- The CLAUDE.md configuration hierarchy: user-level (~/.claude/CLAUDE.md), project-level (.claude/CLAUDE.md or root CLAUDE.md), and directory-level (subdirectory CLAUDE.md files)
  - Be clear about the loading scope and priority of the three-layer memory files, especially how the project level and directory level affect team collaboration.
- That user-level settings apply only to that user—instructions in ~/.claude/CLAUDE.md are not shared with teammates via version control
  - CLAUDE.md is loaded at three levels: user level (`~/.claude/CLAUDE.md`), project level (`.claude/CLAUDE.md` or root directory `CLAUDE.md`), and directory level (subdirectory `CLAUDE.md`). The more specific the level, the higher the priority and can override the upper-level rules.
- The @import syntax for referencing external files to keep CLAUDE.md modular (e.g., importing specific standards files relevant to each package)
  - User-level `~/.claude/CLAUDE.md` is not shared through version control, and new members will not load these rules after cloning the repository. Team sharing specifications must be written into project-level configuration and submitted. This is the most common cause of "rules not taking effect" configuration failures.
- .claude/rules/ directory for organizing topic-specific rule files as an alternative to a monolithic CLAUDE.md
  - The `@import` syntax allows reference to external rule files in the CLAUDE.md main file, which is suitable for splitting rules by package or topic (such as `@import ./standards/testing.md`) to avoid bloated context caused by a single file being too large.

  - The `.claude/rules/` directory is used to store theme-specific rule files (such as `testing.md`, `api-conventions.md`, `deployment.md`). It supports conditional loading by file mode through `paths` frontmatter, so that only relevant rules are loaded for each type of file.

- Diagnosing configuration hierarchy issues (e.g., a new team member not receiving instructions because they're in user-level rather than project-level configuration)
  - When the behavior is inconsistent, first check which level the rule is placed at, whether it is submitted to version control, and whether the current path is hit.
- Using @import to selectively include relevant standards files in each package's CLAUDE.md based on maintainer domain knowledge
  - When diagnosing configuration level problems, you should first confirm which level the rule file is located at, whether it has been submitted to version control, and whether the current working directory matches the corresponding path scope. The `/memory` command can directly view which memory files are actually loaded in the current session.
- Splitting large CLAUDE.md files into focused topic-specific files in .claude/rules/ (e.g., testing.md, api-conventions.md, deployment.md)
  - Selectively import specifications by package through `@import`. Each package only loads rules related to itself, preventing front-end rules from appearing in the back-end development context and infrastructure rules from interfering with application layer code modifications.
- Using the /memory command to verify which files memory are loaded and diagnose inconsistent behavior across sessions
  - Split the large CLAUDE.md into multiple theme files (test specifications, API conventions, security rules) under `.claude/rules/`. Each file has a single responsibility, making it easy to maintain and update independently, and to load on demand through `paths`.

  - The `/memory` command displays all CLAUDE.md and rule files actually loaded in the current session. It is a direct tool for diagnosing the problem of "rules sometimes take effect and sometimes do not take effect". It is more reliable than guessing the configuration level by intuition.

### Task Statement 3.2: Create and configure custom slash commands and skills

#### Knowledge of:

- Project-scoped commands in .claude/commands/ (shared via version control) vs user-scoped commands in ~/.claude/commands/ (personal)
  - Team commands should be placed at the project level, and personal commands should be placed at the user level to avoid imposing private workflows on the team.
- Skills in .claude/skills/ with SKILL.md files that support frontmatter configuration including context: fork, allowed-tools, and argument-hint
  - Project-level commands are stored in `.claude/commands/`, which are shared with version control and can be called by all team members; user-level commands are stored in `~/.claude/commands/` and are only valid for individuals. When both have the same name, the user level takes precedence.
- The context: fork frontmatter option for running skills in an isolated sub-agent context, preventing skill outputs from polluting the main conversation
  - Skill is a reusable task workflow with `SKILL.md` frontmatter. `context: fork` in frontmatter controls context isolation, `allowed-tools` limits tool permissions, and `argument-hint` displays prompt information when called without parameters.
- Personal skill customization: creating personal variants in ~/.claude/skills/ with different names to avoid affecting teammates
  - `context: fork` makes the Skill run in an independent child agent context, and the large amount of exploration output (file reading, intermediate discovery) generated by it will not flow back to the main conversation, preventing the main conversation context from being polluted by irrelevant details.

  - Users' personalized modifications to team skills should create personal versions with different names in `~/.claude/skills/` to avoid overwriting project-level shared skills and affecting the use of other team members.

#### Skills in:

- Creating project-scoped slash commands in .claude/commands/ for team-wide availability via version control
  - Common team processes, such as review, build migration, and release inspection, should be made into project orders and shared with the warehouse.
- Using context: fork to isolate skills that produce verbose output (e.g., codebase analysis) or exploratory context (e.g., brainstorming alternatives) from the main session
  - The team's high-frequency operations (code review, migration generation, release inspection) should be encapsulated into project-level slash commands, stored in `.claude/commands/` and submitted to version control to ensure that team members use unified entrances and standards.
- Configuring allowed-tools in skill frontmatter to restrict tool access during skill execution (e.g., limiting to file write operations to prevent destructive actions)
  - Tasks that generate a large amount of intermediate output (large code base analysis, batch processing, brainstorming) are suitable for use `context: fork`, the main dialogue only receives structured summaries, and the context capacity is reserved for subsequent decisions.
- Using argument-hint frontmatter to prompt developers for required parameters when they invoke the skill without arguments
  - Skill's `allowed-tools` should be configured according to the principle of least privilege. Review Skills only need Read/Grep/Glob; Document Generation Skills may require Write; Skills that do not require write permissions by default should not be configured with Bash or Write.
- Choosing between skills (on-demand invocation for task-specific workflows) and CLAUDE.md (always-loaded universal standards)
  - The `argument-hint` frontmatter field displays a prompt (such as "Please enter the name of the module to be analyzed") when the user calls the Skill without parameters, reducing Skill execution failures or ambiguous results caused by the lack of necessary parameters.

  - CLAUDE.md is suitable for global specifications that always need to be in effect (coding style, test requirements, prohibitions); Skill is suitable for specific task workflows that are performed on demand (security review, migration generation). The two are not interchangeable: stuffing task processes into CLAUDE.md will unnecessarily increase the resident context.

### Task Statement 3.3: Apply path-specific rules for conditional rule loading

#### Knowledge of:

- .claude/rules/ files with YAML frontmatter paths fields containing glob patterns for conditional rule activation
  - Path rules are triggered through frontmatter's `paths` control, suitable for loading conventions by file mode.
- How path-scoped rules load only when editing matching files, reducing irrelevant context and token usage
  - The `.claude/rules/` file controls the loading timing through the `paths` field (glob mode) in the YAML frontmatter: the rule file will be loaded only when the file path being edited matches `paths`.
- The advantage of glob-pattern rules over directory-level CLAUDE.md files for conventions that span multiple directories (e.g., test files spread throughout a codebase)
  - Path-specific rules are only loaded when matching files are edited, preventing Terraform rules from appearing when modifying React components and test specifications from interfering with models when writing API handlers. Reduce irrelevant context and reduce token usage.

  - When similar files are scattered in various directories of the code base (such as `Button.test.tsx` and `Button.tsx` are adjacent instead of concentrated in `tests/`), the directory level CLAUDE.md cannot cover all locations, and a glob pattern such as `**/*.test.tsx` can be hit consistently regardless of the directory location.- Creating .claude/rules/ files with YAML frontmatter path scoping (e.g., paths: ["terraform/**/*"]) so rules load only when editing matching files
  - Create independent rules for infrastructure, testing, frontend, etc. file types and use glob for precise hits.
- Using glob patterns in path-specific rules to apply conventions to files by type regardless of directory location (e.g., \*_/_.test.tsx for all test files)
  - Use `paths` frontmatter to set exclusive rule files for different types of files (such as Terraform rule binding `terraform/**/*`, test rule binding `**/*.test.ts`). Each convention is only activated when the relevant file is edited.
- Choosing path-specific rules over subdirectory CLAUDE.md files when conventions must apply to files spread across the codebase
  - `**/*.test.tsx` This type of glob pattern can match test files everywhere in the code base without maintaining a CLAUDE.md in each directory. This is the right tool choice when the test files coexist in the same directory as the source files.

  - When the rule boundary is consistent with the directory boundary (`src/api/` has an exclusive convention), use directory-level CLAUDE.md; when the rule needs to be applied to specific types of files across directories (test files scattered everywhere, Terraform files), use `.claude/rules/` + `paths` glob.

#### Skills in:

- Creating .claude/rules/ files with YAML frontmatter path scoping (e.g., paths: ["terraform/**/*"]) so rules load only when editing matching files
- Using glob patterns in path-specific rules to apply conventions to files by type regardless of directory location (e.g., \*\*/\*.test.tsx for all test files)
- Choosing path-specific rules over subdirectory CLAUDE.md files when conventions must apply to files spread across the codebase

### Task Statement 3.4: Determine when to use Plan Mode vs direct execution

#### Knowledge of:

- Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, architectural decisions, and multi-file modifications
  - When it comes to architecture, multiple solutions or large-scale changes, plan ahead to avoid blind modifications that lead to rework.
- Direct execution is appropriate for simple, well-scoped changes (e.g., adding a single validation check to one function)
  - Plan Mode is suitable for tasks that affect multiple files, have multiple valid implementation options, involve architectural decisions, or have high rework costs. First explore the code base in Plan Mode, compare solutions, and have users confirm them before starting execution to avoid going in the wrong direction.
- Plan mode enables safe codebase exploration and design before committing to changes, preventing costly rework
  - Direct execution of small changes suitable for well-defined boundaries: single-file bugs with clear stack traces, adding simple checks to single functions, modifying specific logic under known paths. There's no need to explore the entire codebase or compare multiple implementations.
- The Explore subagent for isolating verbose discovery output and returning summaries to preserve main conversation context
  - Plan Mode is a read-only exploration mode: Claude analyzes the code base structure, understands dependencies, and proposes implementation plans, but does not perform any write operations. After the user confirms the plan, he switches to the execution mode to separate the exploration cost and implementation cost.

  - The Explore sub-agent performs a large number of file reading and exploration operations in an independent context, and returns the discovered content to the main conversation in the form of a structured summary, preventing the discovery phase of multi-stage tasks from consuming the context capacity of the main conversation.

- Selecting plan mode for tasks with architectural implications (e.g., microservice restructuring, library migrations affecting 45+ files, choosing between integration approaches with different infrastructure requirements)
  - Once the impact boundary crosses modules, services or infrastructure, you should first let Claude propose a plan before taking action.
- Selecting direct execution for well-understood changes with clear scope (e.g., a single-file bug fix with a clear stack trace, adding a date validation conditional)
  - Once the impact scope of a task crosses module boundaries, involves multiple services or infrastructure components, or there are multiple effective implementation paths, you should first enter Plan Mode to explore and confirm the direction before starting actual modifications.
- Using the Explore subagent for verbose discovery phases to prevent context window exhaustion during multi-phase tasks
  - Bugs with a clear error stack that are located in a single file or a single function, as well as tasks with a clear scope of change such as adding simple verification logic, can be directly executed: locate the target file → modify → run verification.
- Combining plan mode for investigation with direct execution for implementation (e.g., planning a library migration, then executing the planned approach)
  - The exploration phase of a multi-stage task (exploration → planning → implementation) generates a large number of file reads, which are isolated by the Explore sub-agent. The main dialogue only retains a summary of key findings, retaining enough context space for the subsequent implementation phase.

  - Mature workflow: first explore the code base in Plan Mode, confirm the implementation path (user approval), and then switch to direct execution mode to complete specific modifications. Don’t start modifying files before the plan is confirmed, and don’t skip the planning phase for high-risk tasks.

#### Skills in:

- Selecting plan mode for tasks with architectural implications (e.g., microservice restructuring, library migrations affecting 45+ files, choosing between integration approaches with different infrastructure requirements)
- Selecting direct execution for well-understood changes with clear scope (e.g., a single-file bug fix with a clear stack trace, adding a date validation conditional)
- Using the Explore subagent for verbose discovery phases to prevent context window exhaustion during multi-phase tasks
- Combining plan mode for investigation with direct execution for implementation (e.g., planning a library migration, then executing the planned approach)

### Task Statement 3.5: Apply iterative optimization techniques for incremental improvements

#### Knowledge of:

- Concrete input/output examples as the most effective way to communicate expected transformations when prose descriptions are interpreted inconsistently
  - When natural language requirements are repeatedly misunderstood, giving input and expected output is more effective than continuing to explain.
- Test-driven iteration: writing test suites first, then iterating by sharing test failures to guide progressive improvement
  - When natural language requirements descriptions are repeatedly misunderstood, providing 2-3 specific input/output examples is more effective than continuing to pile on text descriptions. Examples directly define conversion boundaries, formatting requirements, and how edge cases are handled.
- The interview pattern: having Claude ask questions to surface considerations the developer may not have anticipated before implementing
  - First write tests that cover expected behavior, edge cases, and performance requirements, and then provide test failure information to Claude as a basis for iteration. Test failure output is precise and reproducible feedback, which can guide effective repairs better than descriptions such as "there is still a problem".
- When to provide all issues in a single message (interacting problems) versus fixing them sequentially (independent problems)
  - Interview Pattern: Ask Claude to ask questions before implementation, exposing design constraints (cache invalidation strategies, concurrent access, failure modes, memory caps) that developers did not anticipate. It is suitable for scenarios with incomplete requirements such as cache strategy design, data migration, and fault tolerance mechanisms.

  - When multiple problems affect each other (fixing A will affect the behavior of B), all related problems should be described in one message; independent problems can be fixed iteratively in sequence, and each time the results are confirmed before processing the next one, reducing the risk of interference between repairs.

- Providing 2-3 concrete input/output examples to clarify transformation requirements when natural language descriptions produce inconsistent results
  - 2-3 high-quality examples are enough to anchor the format and boundaries, which is more stable than a large pile of rules.
- Writing test suites covering expected behavior, edge cases, and performance requirements before implementation, then iterating by sharing test failures
  - 2-3 high-quality examples covering typical scenarios and key boundaries are usually more stable than a large number of abstract rules. Examples directly demonstrate expected output formats, field contents, and boundary handling, allowing the model to generalize patterns from them rather than memorizing rules.
- Using the interview pattern to surface design considerations (e.g., cache invalidation strategies, failure modes) before implementing solutions in unfamiliar domains
  - The test suite should cover the main business paths, edge cases (such as null, null values, extreme values) and performance requirements (such as timeout thresholds). Providing the specific output of the test failure to Claude can lead to more precise repairs than describing "the output is wrong".
- Providing specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)
  - Before implementing in unfamiliar areas, let Claude expose hidden requirements such as cache invalidation strategies, permission boundaries, failure rollback, and data migration compatibility by asking questions, and make implicit design constraints explicit before starting implementation.
- Addressing multiple interacting issues in a single detailed message when fixes interact, versus sequential iteration for independent issues
  - Boundary case bugs should be presented as specific test cases (what is expected to be output when the input is null, processing logic when the date is empty), rather than a general description of "handling edge cases", which will make the model unable to determine the specific behavior to be satisfied.

  - When multiple issues have dependencies (for example, the same data migration script needs to handle null, date format, and duplicate records at the same time), all constraints should be stated at once to prevent each change from destroying other satisfied conditions during step-by-step repair.

#### Skills in:

- Providing 2-3 concrete input/output examples to clarify transformation requirements when natural language descriptions produce inconsistent results
- Writing test suites covering expected behavior, edge cases, and performance requirements before implementation, then iterating by sharing test failures
- Using the interview pattern to surface design considerations (e.g., cache invalidation strategies, failure modes) before implementing solutions in unfamiliar domains
- Providing specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)
- Addressing multiple interacting issues in a single detailed message when fixes interact, versus sequential iteration for independent issues

### Task Statement 3.6: Integrate Claude Code into CI/CD pipelines

#### Knowledge of:

- The -p (or --print) flag for running Claude Code in non-interactive mode in automated pipelines
  - CI must be run non-interactively to avoid the pipeline getting stuck waiting for manual input.
- --output-format json and --json-schema CLI flags for enforcing structured output in CI contexts
  - The `-p`/`--print` flag must be used in the CI environment to run Claude Code (Print Mode). This mode will exit immediately after completion and output the results without waiting for interactive input. Lack of this flag causes the pipeline to hang permanently.
- CLAUDE.md as the mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code
  - CI output should use the `--output-format json` plus `--json-schema` constraint structure so that downstream scripts can stably parse the results and automatically generate PR inline comments, inspection status, or quality reports. Markdown output is difficult to machine parse.
- Session context isolation: why the same Claude session that generated code is less effective at reviewing its own changes compared to an independent review instance
  - The Claude Code instance called by CI will also read the project's CLAUDE.md. Test standards, fixture conventions, review dimensions, and content types that do not need to be reported should be written into the project configuration instead of only existing in the developer's local conversation.

  - Generating code in the same Claude session and then reviewing it will be affected by the reasoning context during generation, making it easier to rationalize your design choices. Reviewing using a standalone CI instance (without build history) provides a more objective view of issues.- Running Claude Code in CI with the -p flag to prevent interactive input hangs
  - Use `-p`/`--print` in the automation script to ensure that the results are output after execution rather than entering chat mode.
- Using --output-format json with --json-schema to produce machine-parseable structured findings for automated posting as inline PR comments
  - Use `-p` (Print Mode) in CI scripts: execute specified tasks, output results, and exit immediately. When this flag is not used, Claude Code enters interactive mode and waits for terminal input, causing the automated pipeline to be permanently blocked.
- Including prior review findings in context when re-running reviews after new commits, instructing Claude to report only new or still-unaddressed issues to avoid duplicate comments
  - `--output-format json` cooperates with `--json-schema` to generate structured JSON that conforms to the specified schema, which can be stably parsed by the script and mapped to the file path, line number, severity level and description field of PR inline comments.
- Providing existing test files in context so test generation avoids suggesting duplicate scenarios already covered by the test suite
  - When running a code review repeatedly, the last findings should be passed into the context and explicitly required to report only new or unresolved issues. Without this constraint, the same issues will be reported repeatedly in each review, causing PR review noise.
- Documenting testing standards, valuable test criteria, and available fixtures in CLAUDE.md to improve test generation quality and reduce low-value test output
  - Provide existing test files as context before generating tests, allowing Claude to identify covered scenarios and avoid recommending repeated tests. Also describe what criteria a valuable test should meet (such as coverage requirements, types of scenarios that must be included).

  - Test criteria (coverage requirements, types of scenarios that must be covered), valuable test criteria and paths to available test fixtures should be recorded in CLAUDE.md. Claude used by CI reads these configurations to generate tests that meet the project's quality requirements.

#### Skills in:

- Running Claude Code in CI with the -p flag to prevent interactive input hangs
- Using --output-format json with --json-schema to produce machine-parseable structured findings for automated posting as inline PR comments
- Including prior review findings in context when re-running reviews after new commits, instructing Claude to report only new or still-unaddressed issues to avoid duplicate comments
- Providing existing test files in context so test generation avoids suggesting duplicate scenarios already covered by the test suite
- Documenting testing standards, valuable test criteria, and available fixtures in CLAUDE.md to improve test generation quality and reduce low-value test output

## Task 3.1: CLAUDE.md Configuration Hierarchy

### What this tests

This question tests "Why the rules are not effective" and "Where should the rules be placed?" The exam will not just ask about the file name, but will usually give a team collaboration scenario: a developer's local Claude Code behaves normally, but a new colleague does not have the same agreement after cloning the warehouse; or the rules in a certain subdirectory are inconsistent with the project root rules. When making a judgment, first look at the scope: personal preferences should be placed at the user level, team standards should be placed at the project level, and local agreements should be placed at the directory level or path rules. In practice, incorrectly writing team standards into `~/.claude/CLAUDE.md` is the most common problem, because it will not enter version control and will not be loaded by other members.

Modularity is the second focus. `CLAUDE.md` Longer is not always better. An excessively large resident context will dilute key rules. Project-level files are suitable for putting global standards, and then use `@import` or `.claude/rules/` to break out topics such as testing, API, security, and deployment. When encountering inconsistent behavior, don't just check whether the file exists, use `/memory` to verify which memory files are actually loaded in the current session.

### Three-level hierarchy

```
~/.claude/CLAUDE.md ← User level (only effective for this user, not shared through version control)
        ↓ Inherited/overridden
.claude/CLAUDE.md
or root CLAUDE.md ← project level (committed to version control, shared by all team members)
        ↓ Inherited/overridden
src/api/CLAUDE.md ← Directory level (only valid in this directory and its subdirectories)
```

**Scope inheritance rules:** The more specific the level, the higher the priority and can override the upper-level rules.

> When a new colleague does not receive the project specification after cloning the warehouse, first check whether the specification is written in **user-level** `~/.claude/CLAUDE.md` instead of **project-level** `.claude/CLAUDE.md`. Only the project level can be shared through git.

### CLAUDE.md Content structure (recommended template)

```markdown
# Project name

## Project overview

Briefly describe the project purpose and technology stack.

## key agreement

- Coding style: use ESLint + Prettier
- Naming convention: PascalCase for components, camelCase for tool functions
- Test: Each component must have a corresponding .test.tsx file

## Common commands

- `npm test` — run the test suite
- `npm run lint` — code inspection
- `npm run build` — build the production version

## important documents

- `src/api/client.ts` — API client configuration
- `src/components/` — reusable components
- `tests/fixtures/` — test data

## Prohibited operation

- Do not modify the dist/ directory directly
- Don't commit .env files
- console.log is not allowed in production code
```

### `@import` Syntax (modular splitting)

```markdown
# .claude/CLAUDE.md main file

@import ./standards/api-conventions.md
@import ./standards/testing-standards.md
@import ./standards/security.md
```

Applicable scenarios: If the rule file is too large, split it by topic to keep the main file concise.

### `.claude/rules/` Directory (themed organization)

```
.claude/
├── CLAUDE.md ← Project-level main configuration (or use @import)
├── rules/
│ ├── testing.md ← Test specification (can contain paths field)
│ ├── api-conventions.md ← API Conventions
│ ├── security.md ← Security Rules
│ └── deployment.md ← Deployment rules
└── commands/
    ├── review.md ← /review command
    └── test-gen.md ← /test-gen command
```

### Diagnostic commands

```bash
/memory # Check which memory files are loaded in the current session
           # Used to diagnose: Why is the rule not taking effect? Is the expected CLAUDE.md loaded?
```

---

## Task 3.2: Custom slash commands and skills

### What this tests

This question tests "When to use commands, when to use Skills, and when to write CLAUDE.md". The slash command is suitable for solidifying prompt templates that are frequently executed by the team, such as `/review`, `/test-gen`, `/release-check`. Project-level commands should be put in `.claude/commands/` and submitted to the warehouse, user-level commands should be put in `~/.claude/commands/` and only serve individuals. If the question emphasizes "everyone on the team must be able to use it", the answer usually points to the project level; if it emphasizes "personal preference or experimental process", the answer usually points to the user level.

The focus of Skill is `SKILL.md` frontmatter. `context: fork` is used to isolate long analysis or exploratory output to sub-agent context, and the main dialogue only retains the conclusion; `allowed-tools` is used to limit tool permissions, especially suitable for processes such as review, reporting, and document generation that should not change files; `argument-hint` is used to tell the caller which parameters are missing. In practice, Skill is more like a "process package called on demand", and CLAUDE.md is more like "a long-term standard that should be adhered to every time".

### Command scope

| Type | Storage location | Shared scope | Calling method |
| -------------- | --------------------------------- | ---------------- | --------- |
| **Project-level commands** | `.claude/commands/review.md` | Sharing via version control | `/review` |
| **User-Level Command** | `~/.claude/commands/review.md` | Personal use only | `/review` |

### Slash command file format

```markdown
# .claude/commands/security-review.md

---

description: Conduct a security review of current code changes
argument-hint: "Optional: Specify review focus (e.g.'auth', 'injection'）"

---

Please conduct a security review on the following code, focusing on:

1. Injection vulnerabilities (SQL, command injection)
2. Authentication/authorization defects
3. Exposure of sensitive data
4. Hardcoded keys or passwords

$ARGUMENTS If additional parameters are provided, focus on: $ARGUMENTS
```

### SKILL.md Frontmatter configuration

```markdown
---
context: fork #Run in an isolated sub-agent context to prevent large amounts of output from contaminating the main conversation.
allowed-tools: # Limit the tools available during Skill execution (principle of least privilege)
  - Read
  - Grep
  - Glob
argument-hint: "Please enter the module name to be analyzed" # Prompt shown when no argument is provided
description: "Analyze the code quality of the specified module" # Skill description
---

# Skill content

Analyzing the code quality of the $ARGUMENTS module...
```

### `context: fork` Usage Principles

```
Good to use context: fork (which produces a lot of exploratory output):
- Code base exploration (reading large amounts of files)
- Brainstorming and scenario analysis
- Batch file processing
- Generate large reports

No need for context: fork (the output is short and does not pollute the context):
- Short formatting command (/format)
- Quick query of single file
- Simple code completion
```

### CLAUDE.md vs Skills selection

| Applicability | Selection |
|------------------------------------------------ | ---------------------------------- |
| Common project standards (coding style, naming conventions) that always need to be loaded | **CLAUDE.md** |
| Task-specific workflows invoked on demand (security review, test generation) | **Skills (slash command)** |
| The workflow has many steps and will produce a lot of intermediate output | **Skills + `context: fork`** |

---

## Task 3.3: Path-Specific Rules

### What this tests

This question tests the difference between "loading rules by directory" and "loading rules by file mode". Directory-level `CLAUDE.md` is suitable for situations where the rule boundaries are consistent with the directory boundaries. For example, `src/api/` has a special API convention. Path-specific rules are suitable for situations where the rules span multiple directories, such as test files, Terraform files, Storybook files, and migration scripts scattered in different locations in the warehouse. When the question "files spread throughout the codebase" or "regardless of directory location" appears, you should first think of `.claude/rules/` plus glob.The benefit of path rules is not just to organize files, but to reduce irrelevant context. Loading the corresponding rules only when editing files matching `paths` can reduce token usage and reduce Claude being misled by irrelevant conventions. In actual combat, the glob must be written accurately enough, for example, use `**/*.test.tsx` for test rules and `terraform/**/*` or `infra/**/*` for infrastructure rules to avoid mistaken triggering of rules in irrelevant files.

### `.claude/rules/` `paths` field of the file

Make the rule only take effect on matching files through the `paths` field of YAML frontmatter:

```yaml
# .claude/rules/terraform.md
---
paths:
  - "terraform/**/*"
  - "infrastructure/**/*"
---
# Terraform specification
- All resources must add `Environment` and `Owner` tags
- Variable definition must contain description and type
- Hardcoding AWS Account IDs is not allowed
- Use `data` source instead of hardcoding AMI ID
```

```yaml
# .claude/rules/testing.md
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
---
# Test file specifications
- Every test must have describe block
- Mock function must be reset in beforeEach
- use of test.only is not allowed (will prevent other tests from running)
```

### Glob mode vs directory level CLAUDE.md

| Scenario | Recommended solution | Reason |
| -------------------------------------------------- | ---------------------------------- | ---------------------------------- |
| Rules are divided by **directory location** (`src/api/` has special rules) | Directory level `src/api/CLAUDE.md` | Clear directory boundaries |
| Rules divided by **file type** (test files scattered everywhere) | `.claude/rules/` + glob mode | Test files are not concentrated in one directory |
| Rule spans multiple unrelated directories | `.claude/rules/` + multiple paths | glob can match multiple locations |

**Classic exam scene:**
Test files are scattered throughout the code base (`Button.test.tsx` is next to `Button.tsx`, rather than concentrated in the `tests/` directory) → Using `**/*.test.tsx` glob mode, directory level CLAUDE.md cannot cover this scenario.

---

## Task 3.4: Plan Mode vs direct execution

### What this tests

This question tests the judgment of risk and uncertainty. Plan Mode is suitable for large-scale, multi-file, multi-scheme, architecturally-impacted, or high-risk tasks because it allows you to explore the code base, compare options, and confirm the design before starting to modify it. Directly execute small tasks that are suitable for well-bounded tasks, such as fixing a single file bug based on a clear stack, or adding simple validation to a function. The judgment criterion is not "whether the task sounds big", but whether there are multiple reasonable solutions, whether dependencies need to be understood first, and whether there is a high cost of rework.

The Explore subagent is meant to isolate the noise from the discovery phase. Large-scale migration, legacy system combing, and cross-service dependency analysis will produce a large amount of intermediate output. Directly inserting it into the main conversation will consume context and affect subsequent decisions. A better process is: Plan Mode does investigations and plans, Explore subagent summarizes the findings, and then uses direct execution to complete the implementation after the user confirms.

### Judgment Matrix

| Use **Plan Mode** | Use **Direct Execution** |
|------------------------------------------ |----------------------------- |
| Large scale changes (affecting 10+ files) | Single file bug fix |
| There are multiple valid implementation solutions, which need to be discussed first | The correct method is known and the scope is clear |
| Architectural decisions (microservice splitting, database migration, library migration) | Add single field validation |
| Need to explore the code base before deciding how to change | Simple fix with clear stack trace |
| Preview before high-risk operations (deleting data, irreversible changes) | New features (low destructive) |

### How to turn on Plan Mode

```bash
# Method 1: Use the /plan command to enter in the conversation
/plan

# Method 2: Add "Think about..." before the prompt (implicit trigger plan mode)
# Will make Claude think before he acts
```

### Combination strategy

```
Plan Mode exploration phase:
→ Claude analyzes the code base structure (read only, no writing)
→ Understand dependencies
→ Propose an implementation plan (the user can modify/reject before confirming)

After the user confirms the plan:
→ Direct execution phase (implementation of specific changes as planned)
```

### Explore subagent

Used to isolate detailed discovery output to prevent the main conversation context from being exhausted:

- **Suitable**: legacy code base exploration, large file reading, unknown code base structure
- **RETURN**: Structured summary instead of full exploration output
- **Effect**: Only the summary is visible in the main dialogue, and the context capacity is retained.

---

## Task 3.5: Iterative optimization technology

### What this tests

This question tests how to move Claude from "fuzzy execution" to "verifiable improvement". When natural language descriptions are repeatedly misunderstood, the most effective feedback is not to continue writing longer descriptions, but to give 2-3 concrete input/output examples. Examples can directly define conversion boundaries, formatting requirements, and exception handling, making them less likely to be misinterpreted than abstract rules.

Test-driven iteration is another thread. By first writing tests that cover expected behavior, edge cases, and performance requirements, and then providing failure information to Claude, you can turn "I don't think something is right" into a reproducible fix goal. The Interview pattern is suitable for use when the requirements are incomplete or the field is unfamiliar. Let Claude first ask about hidden constraints such as cache invalidation, failure modes, permission boundaries, and data consistency. Whether multiple questions are given at one time depends on whether they affect each other: interactive questions are explained at one time, and independent questions are iterated sequentially.

### Four core technologies

**1. Specific input/output examples**

When written descriptions are repeatedly misunderstood, providing 2-3 specific examples is more effective than a long description:

```
Input: "2024-01-15T10:30:00Z"
Output: "January 15, 2024 10:30"

Input: "1705311000" (Unix timestamp)
Output: "January 15, 2024 10:30"

Input: null or empty string
Output: "Unknown date"
```

**2. Test-driven iteration (TDD)**

Write the test first → Share test failure information → Guide Claude to fix it step by step:

- Test failures are high-quality, precise feedback that is more accurate than natural language descriptions
- Explicitly include edge cases and performance requirements
- Only fix one failing test per iteration (avoiding large-scale changes)

**3. Interview Pattern**

Let Claude ask first, exposing design considerations that the developers didn't anticipate:

```
Developer: "Implement a caching system"
Claude should first ask:
- What is the cache invalidation strategy? (TTL? LRU? Manual invalidation?)
- Is locking required for concurrent access?
- Does the cache hit rate need to be monitored?
- What is the memory limit?
```

Suitable scenarios: cache strategy design, data migration plan, fault tolerance mechanism

**4. Batch vs Sequential Repair**

| Situation | Strategy | Reason |
| -------------------------------- | -------------------- | --------------------- |
| Multiple problems affect each other (fixing A will affect B) | A single message provides all problems | Avoid fixing A and breaking B |
| Multiple problems are independent of each other | Sequential iterative repair | Confirm the results each time and reduce risks |

---

## Task 3.6: CI/CD integration

### What this tests

This question is about turning Claude Code from an interactive tool into an automated pipeline component. `-p` or `--print` non-interactive mode must be used in CI, otherwise the task may hang waiting for input. It is best to use the `--output-format json` and `--json-schema` constraint structures for automated results, so that subsequent scripts can convert findings into PR inline comments, check status, or reports.

CI scenarios also emphasize context and isolation. Claude called by the pipeline still requires project background. Test standards, fixture conventions, and review standards should be written in CLAUDE.md, rather than existing only in the developer's local conversation. Reviews are best done in separate sessions rather than the same session that generated the code, since it is easier for the generator to inherit assumptions from the original review. When repeating the review, you should bring existing findings and make it clear that only new or unresolved issues will be reported to avoid creating duplicate comments in the PR.

### Non-interactive mode configuration

```bash
# -p/--print (Print Mode) must be used in CI
# Otherwise Claude Code will wait for interactive input, causing the CI pipeline to hang permanently
claude -p "Review this PR for security vulnerabilities" \
       --output-format json \
       --json-schema ./schemas/review-result.json

# Using a heredoc to pass multiple lines of content
claude -p "$(cat <<'EOF'
Review the following code changes:
$(git diff HEAD~1)
Key checks: SQL injection, XSS, CSRF vulnerabilities
EOF
)"
```

### Key CLI parameters

| Parameters | Function | CI usage scenarios |
| ------------------------ | ----------------------------------------------- | -------------------------- |
| `-p` / `--print` | **Print Mode**: Exit immediately after outputting the results (non-interactive) | **Must be used in all CI scenarios** |
| `--output-format json` | Structured JSON output | For machine parsing results |
| `--json-schema <file>` | Constrain JSON output structure | Force a specific output format |
| `--headless` | Disable UI interactive elements | Automated pipeline |
| `--model <model>` | Specify the Claude model to use | Control the cost/speed trade-off |
| `--allowedTools <tools>` | Limit available tools (comma separated) | Principle of least privilege |
| `--resume <session>` | Restore named session | Preserve context across steps |

### CI Code Review Best Practices

```bash
# Recommended: Independent review of examples (rather than self-examination)
# In .github/workflows/review.yml:

- name: Claude Code Review
  run: |
    claude -p "Review changes to this PR" \
           --output-format json \
           --allowedTools "Read,Grep,Glob" \ # Read-only tools, writing is not allowed
           > review-result.json

# Put existing test files into context to avoid suggesting repeated tests
- name: Claude Code Review with Context
  run: |
    claude -p "Review new code and suggest missing test cases based on existing test files (tests/ directory).
               Don't repeat existing tests. " \
           --output-format json > suggestions.json
```

**CI Review Notes:**

```
Recommended practices:
- Use a standalone instance (standalone session) for review, not self-check in the same session where you write the code
- When re-examining, pass existing findings into the context (only new or unrepaired issues are reported)
- Document test criteria and available fixtures in CLAUDE.md (also read by CI)
- Structured output + JSON Schema (convenient for CI system parsing)

What to avoid:
- Without the -p flag (the CI pipeline will hang permanently waiting for input)
- Write code and review in the same session (reducing objectivity, model has generative bias)
- No restrictions on tool permissions (CI review instances should not have write permissions)
```

### The role of CLAUDE.md in CI

The Claude Code instance in CI also reads `.claude/CLAUDE.md`, which can be used to inject project context:

```markdown
## CI review configuration

### Test standards

- Test coverage requirement: >= 80%
- All asynchronous functions must have error handling
- Disable console.log in production code

### Available test fixtures

- `tests/fixtures/mock-user.ts` — user data mock
- `tests/fixtures/mock-api.ts` — API response mock

### No suggested content needed

- Performance optimization (not within the scope of this PR)
- Style/formatting issues (automatically handled by Prettier)
```

---

## Domain 3 deep dive: Claude Code configuration, workflows, and team engineering

Domain 3 tests how Claude Code can enter a real development team, rather than a single developer temporarily chatting with a model. It covers configuration hierarchy, team sharing rules, slash commands, Skills, path rules, Plan Mode, iteration optimization and CI/CD. Putting these contents together, the core issue is: how to make Claude Code consistent, controllable, and reproducible among different people, different directories, different tasks, and different automation environments. Exam questions usually present a team collaboration issue, such as new colleagues not loading rules, test file rules are invalid, CI is stuck, model duplication suggests existing tests, and then ask you to choose the fix that best meets engineering practices.

### 1. CLAUDE.md is team memory, not a personal notepad

The CLAUDE.md hierarchy is the basis of Domain 3. User level `~/.claude/CLAUDE.md` is only valid for individuals and is suitable for personal preferences, such as commonly used languages, personal explanation styles, and private command habits. Project-level `.claude/CLAUDE.md` or root `CLAUDE.md` should be committed to version control, suitable for team-shared standards such as coding style, testing standards, architectural conventions, and no-nos. Directory-level `src/api/CLAUDE.md` is suitable for local conventions, such as API layer error handling, database access modes, and front-end component naming conventions.

In the exam, "Claude did not comply with the regulations after the new colleague cloned the repository" is almost certainly a test of scope. If the specification is written in the user-level file of an old employee, the new colleague will not load it; the correct fix is ​​to move it to the project level or directory level and submit it to version control. Another common scenario is that there are too many or too few rules for a certain subdirectory. In this case, it is necessary to determine whether the rule boundary is a directory boundary or a file type boundary. Use directory-level CLAUDE.md when directory boundaries are clear, and use `.claude/rules/` and glob when file types are scattered across directories.

CLAUDE.md also needs to be modular. Large warehouses cram all content into one huge file, which will make the context bloated and increase maintenance costs. `@import` You can split testing, security, api, deployment and other rules into independent files, and then selectively import them by CLAUDE.md of the relevant package. This reflects the principle of "loading only relevant rules". During the exam, if the question says that monolithic CLAUDE.md is too long and the rules interfere with each other, the correct direction is usually to split topic-specific files or use `@import`.

### 2. /memory is a diagnostic tool, not a decoration command

When Claude Code behaves inconsistently, don't rely on your gut to guess whether the rules are loading. `/memory` can check which memory files are actually loaded in the current session. For example, one developer sees the API rules taking effect in the `src/api` directory, but another developer does not see it in the project root directory. This may be due to the path or hierarchy. `/memory` can turn "model disobedience" problems into diagnosable configuration problems.

This idea is very useful for exams. If the question describes "sometimes the rules are followed, sometimes not", "different directories behave differently", "new session loading content is inconsistent", don't rush to change the prompt, first check the memory hierarchy, path scope, whether the rules are submitted, and whether the current working directory is hit. Claude Code has a configuration system, and engineering problems must be solved using configuration diagnostics.

### 3. The difference between slash commands and skills

Slash commands are suitable for solidifying commonly used team operations, such as `/review-module`, `/generate-tests`, `/write-migration`. Project-level commands are placed in `.claude/commands/` for team sharing; user-level commands are placed in `~/.claude/commands/` for personal use. The value of commands is to reduce repeated prompts and allow teams to use the same entry points for the same type of tasks.

Skills are more like reusable task workflows, usually with `SKILL.md` and frontmatter. `context: fork` is a key option, suitable for tasks such as code base exploration, batch analysis, and brainstorming that generate a large amount of context. After a fork, the main conversation only receives a summary, which is not contaminated by intermediate discoveries of subtasks. `allowed-tools` is used to limit the tools that can be used during Skill execution to prevent an analysis Skill that should be read-only from performing write operations. `argument-hint` is used to prompt the caller to provide necessary parameters to reduce misuse.

Choosing CLAUDE.md or Skill depends on whether the content is "always required". Team naming conventions, testing standards, and architectural principles should be resident in CLAUDE.md; security review, migration generation, and complex analysis processes should be made into Skills that can be called on demand. Writing all task processes into CLAUDE.md will waste context; making all basic team specifications into Skills will not take effect by default. Exam questions often use this boundary to differentiate between good design and over-configuration.

### 4. Path-specific rules solve the "scattered files" problem

The `paths` frontmatter of `.claude/rules/` is designed for conditional loading. Test files may be spread across `src/components/Button.test.tsx`, `packages/api/user.spec.ts`, and `apps/web/*.test.ts` — directory-level CLAUDE.md can't cover all of these. A glob like `**/*.test.tsx` or `**/*.spec.ts` loads test rules by file type regardless of location. Terraform, SQL migrations, React components, and GitHub Actions all fit this pattern.

The value of path rules is to reduce irrelevant context. Infrastructure rules are loaded when editing Terraform files, test rules are loaded when editing test files, and design system rules are loaded when editing UI components. The more relevant the rules the model sees, the easier they are to follow. If the exam question says "test rules are scattered and cannot be covered", the correct answer is usually path-specific rules instead of copying CLAUDE.md in each directory.

### 5. Plan Mode is for risk control, not for slowing things down

Plan Mode is suitable for complex, multi-file, multi-scheme, and high-risk tasks. For example, library migration affects 45 files, microservice splitting, database migration, authentication architecture adjustment, and infrastructure transformation. Its value is to first read the code, understand the dependencies, compare solutions, and then let the user confirm the direction. Directly execute small changes suitable for clear scope, such as single function verification, clear stack bugs, and processing logic of a field.

Plan Mode pairs naturally with the Explore subagent. Large-scale exploration generates a lot of intermediate discoveries. If all that output lands in the main conversation, context fills up before the implementation phase even begins. The Explore subagent handles the file reading and returns a structured summary; the main conversation holds decisions and planning. This connects directly to Domain 5 context management. If an exam question describes "exploration of a large legacy codebase leading to context exhaustion," the answer is usually the Explore subagent or staged summarization rather than reading everything into the main context.

A mature process is: Plan Mode for investigation, direct execution for implementation. First explore and confirm the plan in Plan Mode, and then directly implement specific changes after the user approves it. Don’t understand Plan Mode as only planning but not implementation, and don’t use direct execution for high-risk unknown tasks.

### 6. Iterative optimization requires verifiable feedback to the model

The iteration of Claude Code is not about saying "better" over and over again. When the natural language description is unstable, give 2-3 input/output examples; when the implementation has clear behavioral requirements, write the test first, and then feedback the failure information to Claude; when the field is unfamiliar, use an interview pattern to let Claude ask questions first. Verifiable feedback is more useful than subjective evaluations.

Test-driven iteration is especially important. First write tests to cover the main path, boundaries, exceptions, and performance, and then let Claude implement them. When it fails, output the specific test to it, instead of just saying "there is still a bug". If multiple issues affect each other, such as data migration involving nulls, date formats, and duplicate records, they should be explained at once, because repairing them separately may destroy each other. If the issues are independent of each other, such as a copywriting error and a single bounds check, they can be dealt with sequentially.

The interview pattern is suitable for scenarios with incomplete requirements. Let Claude ask you questions about cache invalidation, permission models, rollback on failure, compatibility, and data migration strategies before implementation. This is not a waste of time but makes hidden needs explicit. If the question in the exam says that the developer is not familiar with the field and may have missed design considerations, the correct direction is usually to ask Claude to ask the question first, rather than implement it directly.

### 7. Claude Code in CI/CD: non-interactive, structured, and minimal permissions

In the CI environment, Claude Code must be run non-interactively with `-p` or `--print`, otherwise the pipeline will wait for input and hang. Output should use `--output-format json` and `--json-schema` to facilitate automatic posting of PR comments, generation of inspection results, and filtering severity levels. CI review instances should be as read-only as possible, such as allowing `Read,Grep,Glob` to avoid review tasks writing code or executing dangerous commands.

CLAUDE.md is equally important in CI. Test standards, fixture locations, review standards, and content that does not need to be reported should all be written into the project configuration. Otherwise, Claude in CI can only recommend tests based on general experience, and it is easy to repeat existing tests or make low-value suggestions. If you want Claude to generate missing tests, you should provide an existing test file context so that it avoids overwriting existing scenarios.

Examples of independent reviews are also readily available. If the same Claude session has just generated code and then asks it to review itself, it will be affected by the reasoning context during generation, making it easier to rationalize your choices. CI review should use a standalone instance, without a build process, and only look at diff, context, and criteria. When re-examining, old findings must also be passed in, and only new or unresolved issues must be reported to avoid repeated PR comments.

### 8. Diagnosing Domain 3 questions

When encountering Claude Code workflow problems, you can first determine which layer the problem occurs at. Configuration is not shared, which is a CLAUDE.md scope problem; certain types of file rules are not loaded, which is a path-specific rules problem; repeated task prompts are inconsistent, which is a slash command or Skill problem; long exploration pollution context is a `context: fork` or Explore subagent problem; large-scale modification risks are high, which is a Plan Mode problem; CI is stuck, which is a lack of `-p`; CI output is difficult to parse, which is a lack of JSON/schema; the review is not objective and is a matter of self-examination within the same session.

The most common wrong answers include: putting team rules at the user level; overwriting personal skills with team skills; using directory-level CLAUDE.md for scattered test files; executing complex migrations directly; CI not using print mode; CI outputting natural language and letting scripts parse it; generating and reviewing the same session; not providing existing tests resulting in duplicate suggestions. As long as you keep the four words "scope, context, permissions, and structured output" in your mind, most of the questions in Domain 3 can be located.

### 9. Implementing Claude Code as a team

When individuals use Claude Code, the most important thing is whether the prompts are clear; when a team uses Claude Code, the most important thing is consistency. If everyone in a team has their own rules, their own orders, and their own review standards, the output of Claude Code will be as fragmented as the personal habits of different developers. Domain 3 focuses on how to upgrade individual prompts into team workflows: rules go into version control, commands go into the project directory, Skills have explicit tool permissions, CI uses structured output, and review criteria are written into CLAUDE.md.The first step for the team to implement is to organize the "default rules" and "on-demand processes". Default rules include coding style, test directory, error handling conventions, safety bottom line, and submission information format. These should be written into project-level CLAUDE.md or directory-level CLAUDE.md. On-demand processes include generating migrations, security reviews, refactoring plans, release notes, and performance analysis, which are suitable for writing slash commands or skills. In this way, developers do not need to copy long prompts every time, and do not insert one-time tasks into the permanent context.

The second step is to define permission boundaries. Claude Code can read files, write files, run commands, and connect to MCP. The stronger the ability, the more it needs to be restricted according to the scene. Code review commands typically require only Read/Grep/Glob; build tests may require Write/Edit; deployment checks may require Bash but should not have production write access. `allowed-tools` Not formalism, but aligning task risks with tool capabilities. If there are exam questions like "The file was accidentally modified during the review process" or "The analysis skill executed a dangerous command", the correct direction is to tighten the tool permissions.

The third step is to incorporate feedback into the process. Claude Code should not only be run locally by the developer, but can also be used for PR, CI, code review, test generation, and document updates. Once in automation, output must be structured, failures must be parsable, and duplicate comments must be avoided. Natural language is suitable for human reading, not suitable for machine processing. In CI, JSON/schema is used to output findings, and then the script is mapped into inline comments, which is a maintainable method.

### 10. Setting team norms around Plan Mode vs direct execution

It is best for teams to be clear about which tasks must be planned first. For example, when modifying authentication, payment, permissions, database schema, public API, and cross-package dependencies, you should enter Plan Mode first; copywriting, single point bugs, and simple verification can be executed directly. This norm avoids two extremes: planning everything, which leads to low efficiency, and directly changing everything, which leads to high risk. Plan Mode products should also have quality requirements: list the scope of impact, candidate solutions, risks, verification steps, and rollback strategies.

Direct execution is not blind execution either. Even if it is a small change, you must first read the relevant documents, confirm and test, and verify after modification. The advantage of Claude Code is that it can execute commands and edit files, but this also means that it should form a closed loop like an engineer: understand, modify, run tests, and interpret the results. Exam questions that only emphasize "let Claude fix it directly" without verification steps are usually not the best engineering answers.

### 11. Design quality of Skills

A good skill should be as clear as the internal tool documentation. The beginning explains what problems it solves, what scenarios it is suitable for, and what scenarios it is not suitable for; frontmatter limits tool permissions; if the parameters are necessary, use `argument-hint` to prompt; if a large amount of exploration is required, use `context: fork`; the text gives the steps, output format, quality standards, and failure handling. Don't write a huge universal Skill, and don't let the Skill depend on the implicit context.

Personal skills and project skills should be named to distinguish them. Individuals can have `/my-review-style`, but the team `/review` command should not be overwritten by anyone's personal version. For a team skill to be stable, changes to it should be reviewed like any other code change, because it affects Claude Code behavior for all members. The exam may not ask about review processes directly, but will test scope and naming isolation through scenarios like "a developer's personal changes are affecting teammates."

### 12. Claude Code and MCP working together

The intersection of Domain 3 and Domain 2 is MCP integration. Claude Code obtains external system capabilities through MCP, such as Jira, GitHub, databases, and document libraries. Project-level MCP configuration lets teams share these capabilities, but CLAUDE.md still tells Claude when to use them. For example, "When checking requirements, use Jira MCP first, and do not use Grep to search the local cache." "Check the database schema through MCP resources, and do not run production queries." Without these rules, although the tool is connected, the model may not be selected correctly.

MCP also affects Plan Mode. In the planning stage, you may need to read issues, read schemas, and check documents; in the execution stage, you may need to modify code and run tests. Tool permissions can be different at different stages. A mature workflow will separate exploration, planning, execution, and verification, and only provide necessary tools for each step. This not only reduces risks but also makes audits clearer.

### 13. Scenario-based judgment on the exam

If the question is "new members don't see project rules after cloning", the answer depends on the configuration level; if it is "Test rules only take effect in a certain directory", the answer depends on path-specific rules; if it is "Too much Skill output causes confusion in the main dialogue", the answer looks on `context: fork`; if it is "CI waiting for input", the answer looks on `-p`; if it is "PR comments cannot be automatically located", the answer looks on JSON/schema and inline comments; if it is "Same Claude "Post-generation review missed the issue", the answer depends on the independent instance.The essence of Domain 3 is transforming Claude Code from a chat assistant into an engineering system component. Engineering system components require configuration, scoping, permissions, version control, automation, validation, and auditing. An answer that reflects these elements will always be closer to the official intent than one that simply says "change the prompt."

### 14. Deducing the correct configuration from failure cases

A typical failure case is: the team wrote "all tests must use fixtures" in the personal `~/.claude/CLAUDE.md`. The old members run normally locally, but the new members and CI do not comply. The correct fix is ​​not to have new members copy personal files, but to move the rules to the project-level CLAUDE.md and make the fixture list clear. Another failure case is: the team stuffed all front-end, back-end, Terraform, and test rules into the root directory CLAUDE.md, causing Claude to see a large number of irrelevant rules when editing any file. The correct fix is ​​to split it into `.claude/rules/` and directory-level CLAUDE.md, and let the rules be loaded according to the path.

Another failure is that Skill has no restricted tools. For example, the security review skill originally only required code reading, but allowed Bash and Write. As a result, the model directly changed the file in order to "fix" the problem. The correct design is to make the review skill read-only, and the fix skill to be set separately and ask the user for confirmation. Splitting "review" and "modification" into different workflows will create clearer authority boundaries in the project.CI failures are also common: not having `-p` causes the pipeline to hang; outputting Markdown causes script parsing to fail; not bringing old findings leads to duplicate comments; not providing existing tests leads to suggestions for repeated testing; the same session is generated and reviewed causing missed issues. These are not model capability issues, but workflow design issues. Domain 3 questions often use these phenomena to test whether you can put Claude Code into an automated system.

### 15. Building a mental map for Domain 3

When reviewing Domain 3, six concepts organize everything: configuration, reuse, conditional loading, execution mode, iteration, and automation. Configuration maps to the CLAUDE.md hierarchy. Reuse maps to commands and Skills. Conditional loading maps to path-specific rules. Execution mode maps to Plan Mode vs direct execution. Iteration maps to examples, tests, and the interview pattern. Automation maps to CI/CD.

For scenario questions: "inconsistency between team members" → check scope and version control. "File-type rules not loading" → check glob paths. "Context pollution from verbose output" → check `context: fork` and Explore subagent. "High-risk changes" → check Plan Mode. "Machine needs to parse output" → check JSON/schema. "Review quality is poor" → check independent instances. Mapping symptoms to mechanisms is more reliable than memorizing file paths.

The goal isn't maximum configuration — it's configuration that matches the workflow. Too few rules and the model doesn't know the project's habits. Too many rules and the model gets confused by irrelevant ones. Good teams periodically clean CLAUDE.md, retire stale commands, promote repeated prompts into slash commands, and split high-risk processes into plan, execute, and verify stages.

Think of each Claude Code feature in terms of four questions: who shares it, when is it loaded, what can it do, and how is it verified. CLAUDE.md handles shared rules. Path rules handle conditional loading. Commands and Skills handle reusable processes. `allowed-tools` handles capability scope. Plan Mode and CI verification handle safe implementation. That framework turns Domain 3 from a path-memorization exercise into an understanding of what each mechanism is actually for.

---

## Pre-exam checklist

1. **No specification for new colleagues** → Check whether the specification is written in **user level** `~/.claude/CLAUDE.md` (not project level `.claude/CLAUDE.md`)
2. **Project level vs user level** → `.claude/CLAUDE.md` is submitted to git and shared by the team; `~/.claude/CLAUDE.md` is not shared
3. **Team shared command** → `.claude/commands/` (version control); personal command → `~/.claude/commands/`
4. **`context: fork`** → Prevent the large output of Skill from polluting the main conversation context
5. **Test files are scattered everywhere** → `.claude/rules/` + glob mode (`**/*.test.tsx`), directory-level CLAUDE.md cannot be used
6. **Plan Mode** → Used before large-scale changes, multiple file impacts, multiple plans, and high-risk operations
7. **CI hang** → Add `-p`/`--print` flag (Print Mode = non-interactive)
8. **CI structured output** → `--output-format json` + `--json-schema`
9. **CI minimum permission** → `--allowedTools "Read,Grep,Glob"` (only read permission is required for review)
10. **Independent Review Example** → More objective than same-session self-examination, reducing bias
11. **/memory command** → Diagnose which configuration files are loaded in the current session
