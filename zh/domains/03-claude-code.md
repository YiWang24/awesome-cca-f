# Domain 3: Claude Code Configuration & Workflows（Claude Code配置与工作流）

> **权重：20%**  
> 官方文档：[Claude Code Overview](https://docs.anthropic.com/en/docs/claude-code/overview) | [CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/memory) | [Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

---

## Task Statement 覆盖范围

| Task | 主题                                             |
| ---- | ------------------------------------------------ |
| 3.1  | 配置具有适当层次结构、作用域和模块化的 CLAUDE.md |
| 3.2  | 创建和配置自定义斜杠命令与技能                   |
| 3.3  | 应用路径特定规则进行条件约定加载                 |
| 3.4  | 判断何时使用 Plan Mode vs 直接执行               |
| 3.5  | 应用迭代优化技术进行渐进式改进                   |
| 3.6  | 将 Claude Code 集成到 CI/CD 流水线               |

---

### Task Statement 3.1: Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization

#### Knowledge of:

- The CLAUDE.md configuration hierarchy: user-level (~/.claude/CLAUDE.md), project-level (.claude/CLAUDE.md or root CLAUDE.md), and directory-level (subdirectory CLAUDE.md files)
  - 要清楚三层记忆文件的加载范围和优先级，尤其是项目级与目录级如何影响团队协作。
- That user-level settings apply only to that user—instructions in ~/.claude/CLAUDE.md are not shared with teammates via version control
  - CLAUDE.md 按三个层级加载：用户级（`~/.claude/CLAUDE.md`）、项目级（`.claude/CLAUDE.md` 或根目录 `CLAUDE.md`）、目录级（子目录 `CLAUDE.md`）。越具体的层级优先级越高，可覆盖上级规则。
- The @import syntax for referencing external files to keep CLAUDE.md modular (e.g., importing specific standards files relevant to each package)
  - 用户级 `~/.claude/CLAUDE.md` 不通过版本控制共享，新成员 clone 仓库后不会加载这些规则。团队共享规范必须写入项目级配置并提交。这是最常见的"规则不生效"配置故障原因。
- .claude/rules/ directory for organizing topic-specific rule files as an alternative to a monolithic CLAUDE.md
  - `@import` 语法允许在 CLAUDE.md 主文件中引用外部规则文件，适合按包或主题拆分规则（如 `@import ./standards/testing.md`），避免单一文件过大导致上下文臃肿。

  - `.claude/rules/` 目录用于存放主题特定的规则文件（如 `testing.md`、`api-conventions.md`、`deployment.md`），支持通过 `paths` frontmatter 按文件模式条件加载，使每类文件只加载相关规则。

- Diagnosing configuration hierarchy issues (e.g., a new team member not receiving instructions because they're in user-level rather than project-level configuration)
  - 行为不一致时先检查规则放在哪一级、是否提交版本控制、当前路径是否命中。
- Using @import to selectively include relevant standards files in each package's CLAUDE.md based on maintainer domain knowledge
  - 诊断配置层级问题时，应先确认规则文件位于哪个层级、是否提交到版本控制、当前工作目录是否命中对应的 path scope。`/memory` 命令可直接查看当前会话实际加载了哪些记忆文件。
- Splitting large CLAUDE.md files into focused topic-specific files in .claude/rules/ (e.g., testing.md, api-conventions.md, deployment.md)
  - 通过 `@import` 按包选择性导入规范，每个 package 只加载与自身相关的规则，避免前端规则出现在后端开发上下文、基础设施规则干扰应用层代码修改。
- Using the /memory command to verify which memory files are loaded and diagnose inconsistent behavior across sessions
  - 将大型 CLAUDE.md 拆分为 `.claude/rules/` 下的多个主题文件（测试规范、API 约定、安全规则），每个文件职责单一，便于单独维护、更新，也便于通过 `paths` 按需加载。

  - `/memory` 命令显示当前会话实际加载的所有 CLAUDE.md 及规则文件，是诊断"规则有时生效有时不生效"问题的直接工具，比凭感觉猜配置层级更可靠。

#### Knowledge of:

- Project-scoped commands in .claude/commands/ (shared via version control) vs user-scoped commands in ~/.claude/commands/ (personal)
  - 团队命令应放项目级，个人命令放用户级，避免把私人工作流强加给团队。
- Skills in .claude/skills/ with SKILL.md files that support frontmatter configuration including context: fork, allowed-tools, and argument-hint
  - 项目级命令存放在 `.claude/commands/`，随版本控制共享，团队所有成员可以调用；用户级命令存放在 `~/.claude/commands/`，仅对个人有效。两者命名相同时用户级优先。
- The context: fork frontmatter option for running skills in an isolated sub-agent context, preventing skill outputs from polluting the main conversation
  - Skill 是带 `SKILL.md` frontmatter 的可复用任务工作流。frontmatter 中的 `context: fork` 控制上下文隔离，`allowed-tools` 限制工具权限，`argument-hint` 在无参数调用时显示提示信息。
- Personal skill customization: creating personal variants in ~/.claude/skills/ with different names to avoid affecting teammates
  - `context: fork` 使 Skill 在独立的子智能体上下文中运行，其产生的大量探索输出（文件读取、中间发现）不会回流到主对话，防止主对话上下文被无关细节污染。

  - 用户对团队 Skill 的个人化修改应在 `~/.claude/skills/` 中创建不同名称的个人版本，避免覆盖项目级共享 Skill，影响其他团队成员的使用。

- Creating project-scoped slash commands in .claude/commands/ for team-wide availability via version control
  - 常用团队流程如审查、生成迁移、发布检查，应做成项目命令并随仓库共享。
- Using context: fork to isolate skills that produce verbose output (e.g., codebase analysis) or exploratory context (e.g., brainstorming alternatives) from the main session
  - 团队高频操作（代码审查、迁移生成、发布检查）应封装为项目级斜杠命令，存入 `.claude/commands/` 并提交到版本控制，确保团队成员使用统一的入口和标准。
- Configuring allowed-tools in skill frontmatter to restrict tool access during skill execution (e.g., limiting to file write operations to prevent destructive actions)
  - 产生大量中间输出的任务（大型代码库分析、批量处理、头脑风暴）适合使用 `context: fork`，主对话只接收结构化摘要，保留上下文容量用于后续决策。
- Using argument-hint frontmatter to prompt developers for required parameters when they invoke the skill without arguments
  - Skill 的 `allowed-tools` 应按最小权限原则配置。审查类 Skill 只需 Read/Grep/Glob；文档生成类 Skill 可能需要 Write；不应给默认情况下不需要写权限的 Skill 配置 Bash 或 Write。
- Choosing between skills (on-demand invocation for task-specific workflows) and CLAUDE.md (always-loaded universal standards)
  - `argument-hint` frontmatter 字段在用户不带参数调用 Skill 时显示提示（如 "请输入要分析的模块名称"），减少因缺少必要参数导致的 Skill 执行失败或产生模糊结果。

  - CLAUDE.md 适合始终需要生效的全局规范（代码风格、测试要求、禁止事项）；Skill 适合按需执行的特定任务工作流（安全审查、迁移生成）。两者不互换：将任务流程塞入 CLAUDE.md 会无谓增大常驻上下文。

#### Knowledge of:

- .claude/rules/ files with YAML frontmatter paths fields containing glob patterns for conditional rule activation
  - 路径规则通过 frontmatter 的 `paths` 控制触发，适合按文件模式加载约定。
- How path-scoped rules load only when editing matching files, reducing irrelevant context and token usage
  - `.claude/rules/` 文件通过 YAML frontmatter 中的 `paths` 字段（glob 模式）控制加载时机：只有正在编辑的文件路径匹配 `paths` 时，该规则文件才会被加载。
- The advantage of glob-pattern rules over directory-level CLAUDE.md files for conventions that span multiple directories (e.g., test files spread throughout a codebase)
  - 路径特定规则只在匹配文件被编辑时加载，避免 Terraform 规则在修改 React 组件时出现、测试规范在编写 API handler 时干扰模型。减少不相关上下文，同时降低 token 使用量。

  - 当同类文件分散在代码库各目录（如 `Button.test.tsx` 与 `Button.tsx` 相邻而非集中在 `tests/`），目录级 CLAUDE.md 无法覆盖所有位置，`**/*.test.tsx` 这样的 glob 模式可以无论目录位置一致命中。

- Creating .claude/rules/ files with YAML frontmatter path scoping (e.g., paths: ["terraform/**/*"]) so rules load only when editing matching files
  - 为基础设施、测试、前端等文件类型建立独立规则，并用 glob 精确命中。
- Using glob patterns in path-specific rules to apply conventions to files by type regardless of directory location (e.g., \*_/_.test.tsx for all test files)
  - 使用 `paths` frontmatter 为不同类型文件设置专属规则文件（如 Terraform 规则绑定 `terraform/**/*`，测试规则绑定 `**/*.test.ts`），每种约定只在相关文件被编辑时激活。
- Choosing path-specific rules over subdirectory CLAUDE.md files when conventions must apply to files spread across the codebase
  - `**/*.test.tsx` 这类 glob 模式可以匹配代码库中所有位置的测试文件，而无需在每个目录下维护一个 CLAUDE.md。当测试文件与源文件同目录共存时，这是正确的工具选择。

  - 规则边界与目录边界一致时（`src/api/` 有专属约定），使用目录级 CLAUDE.md；规则需跨目录应用于特定类型文件时（散落各处的测试文件、Terraform 文件），使用 `.claude/rules/` + `paths` glob。

#### Knowledge of:

- Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, architectural decisions, and multi-file modifications
  - 涉及架构、多方案或大范围变更时先计划，避免盲改造成返工。
- Direct execution is appropriate for simple, well-scoped changes (e.g., adding a single validation check to one function)
  - Plan Mode 适合会影响多文件、存在多种有效实现方案、涉及架构决策，或有较高返工成本的任务。先在 Plan Mode 中探索代码库、对比方案、由用户确认，再开始执行，避免走错方向。
- Plan mode enables safe codebase exploration and design before committing to changes, preventing costly rework
  - 直接执行适合边界明确的小改动：有清晰堆栈跟踪的单文件 bug、给单个函数添加简单校验、修改已知路径下的特定逻辑。不需要探索代码库全貌或比较多种实现方案。
- The Explore subagent for isolating verbose discovery output and returning summaries to preserve main conversation context
  - Plan Mode 是只读探索模式：Claude 分析代码库结构、理解依赖关系、提出实施方案，但不执行任何写操作。用户确认方案后再切换到执行模式，将探索成本和实施成本分离。

  - Explore 子智能体在独立上下文中执行大量文件读取和探索操作，将发现内容以结构化摘要的形式返回主对话，防止多阶段任务的发现阶段消耗主对话的上下文容量。

- Selecting plan mode for tasks with architectural implications (e.g., microservice restructuring, library migrations affecting 45+ files, choosing between integration approaches with different infrastructure requirements)
  - 一旦影响边界跨模块、跨服务或基础设施，应先让 Claude 提方案再动手。
- Selecting direct execution for well-understood changes with clear scope (e.g., a single-file bug fix with a clear stack trace, adding a date validation conditional)
  - 一旦任务影响范围跨越模块边界、涉及多个服务或基础设施组件，或存在多种有效实现路径，应先进入 Plan Mode 探索并确认方向，再开始实际修改。
- Using the Explore subagent for verbose discovery phases to prevent context window exhaustion during multi-phase tasks
  - 存在明确错误堆栈且定位到单个文件、单个函数的 bug，以及添加简单校验逻辑这类改动范围清晰的任务，可以直接执行：定位目标文件 → 修改 → 运行验证。
- Combining plan mode for investigation with direct execution for implementation (e.g., planning a library migration, then executing the planned approach)
  - 多阶段任务（探索 → 规划 → 实施）的探索阶段产生大量文件读取，用 Explore 子智能体隔离，主对话只保留关键发现摘要，为后续实施阶段保留足够的上下文空间。

  - 成熟的工作流：先在 Plan Mode 中探索代码库、确认实施路径（用户批准），再切换到直接执行模式完成具体修改。不要在计划确认前开始修改文件，也不要对高风险任务跳过计划阶段。

#### Knowledge of:

- Concrete input/output examples as the most effective way to communicate expected transformations when prose descriptions are interpreted inconsistently
  - 当自然语言需求被反复误解时，给输入和期望输出比继续解释更有效。
- Test-driven iteration: writing test suites first, then iterating by sharing test failures to guide progressive improvement
  - 当自然语言需求描述被反复误解时，提供 2-3 个具体输入/输出示例比继续叠加文字说明更有效。示例直接定义了转换边界、格式要求和边界情况处理方式。
- The interview pattern: having Claude ask questions to surface considerations the developer may not have anticipated before implementing
  - 先编写覆盖期望行为、边界情况和性能要求的测试，再将测试失败信息提供给 Claude 作为迭代依据。测试失败输出是精确的、可复现的反馈，比"还有问题"这类描述更能引导有效修复。
- When to provide all issues in a single message (interacting problems) versus fixing them sequentially (independent problems)
  - Interview Pattern：在实施前让 Claude 先提问，暴露开发者未预料到的设计约束（缓存失效策略、并发访问、失败模式、内存上限）。适合缓存策略设计、数据迁移、容错机制等需求不完整的场景。

  - 多个问题相互影响时（修复 A 会影响 B 的行为），应在一条消息中说明所有相关问题；相互独立的问题可以顺序迭代修复，每次确认结果后再处理下一个，降低修复间的干扰风险。

- Providing 2-3 concrete input/output examples to clarify transformation requirements when natural language descriptions produce inconsistent results
  - 2-3 个高质量例子足以锚定格式和边界，比堆大量规则更稳定。
- Writing test suites covering expected behavior, edge cases, and performance requirements before implementation, then iterating by sharing test failures
  - 2-3 个覆盖典型场景和关键边界的高质量示例，通常比大量抽象规则更稳定。示例直接展示期望的输出格式、字段内容和边界处理，使模型能够从中概括规律而非死记规则。
- Using the interview pattern to surface design considerations (e.g., cache invalidation strategies, failure modes) before implementing solutions in unfamiliar domains
  - 测试套件应覆盖主要业务路径、边界情况（如 null、空值、极值）和性能要求（如超时阈值）。将测试失败的具体输出提供给 Claude，比描述"输出不对"更能引导精准修复。
- Providing specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)
  - 在不熟悉的领域实施前，让 Claude 通过提问暴露缓存失效策略、权限边界、失败回滚、数据迁移兼容性等隐藏需求，将隐式设计约束显式化，再开始实现。
- Addressing multiple interacting issues in a single detailed message when fixes interact, versus sequential iteration for independent issues
  - 边界情况 bug 应以具体的测试用例呈现（输入 null 时期望输出什么、日期为空时的处理逻辑），而非笼统描述"处理边界情况"，后者会让模型无法确定具体要满足的行为。

  - 当多个问题存在依赖关系（如同一数据迁移脚本需要同时处理 null、日期格式和重复记录），应一次性说明所有约束，防止分步修复时每次改动破坏其他已满足的条件。

#### Knowledge of:

- The -p (or --print) flag for running Claude Code in non-interactive mode in automated pipelines
  - CI 中必须非交互运行，避免 pipeline 等待人工输入而卡住。
- --output-format json and --json-schema CLI flags for enforcing structured output in CI contexts
  - CI 环境中必须使用 `-p`/`--print` 标志运行 Claude Code（Print Mode），该模式执行完成后立即退出并输出结果，不等待交互输入。缺少该标志会导致 pipeline 永久挂起。
- CLAUDE.md as the mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code
  - CI 输出应使用 `--output-format json` 加 `--json-schema` 约束结构，使下游脚本可以稳定解析结果，自动生成 PR inline comments、检查状态或质量报告。Markdown 输出难以机器解析。
- Session context isolation: why the same Claude session that generated code is less effective at reviewing its own changes compared to an independent review instance
  - CI 调用的 Claude Code 实例同样会读取项目的 CLAUDE.md。测试标准、fixture 约定、审查维度、不需要报告的内容类型，应写入项目配置，而不是只存在于开发者本地对话中。

  - 由同一 Claude 会话先生成代码再审查，会受生成时推理上下文的影响，更容易合理化自己的设计选择。使用独立 CI 实例（不带生成历史）进行审查，能更客观地发现问题。

- Running Claude Code in CI with the -p flag to prevent interactive input hangs
  - 自动化脚本里用 `-p`/`--print`，确保执行完输出结果而不是进入聊天模式。
- Using --output-format json with --json-schema to produce machine-parseable structured findings for automated posting as inline PR comments
  - CI 脚本中使用 `-p`（Print Mode）：执行指定任务、输出结果、立即退出。不使用该标志时 Claude Code 进入交互模式，等待终端输入，导致自动化流水线永久阻塞。
- Including prior review findings in context when re-running reviews after new commits, instructing Claude to report only new or still-unaddressed issues to avoid duplicate comments
  - `--output-format json` 配合 `--json-schema` 产生符合指定 schema 的结构化 JSON，可以被脚本稳定解析，映射为 PR inline comments 的文件路径、行号、严重级别和描述字段。
- Providing existing test files in context so test generation avoids suggesting duplicate scenarios already covered by the test suite
  - 重复运行代码审查时，应将上次的 findings 传入上下文并明确要求只报告新增或仍未解决的问题。不带此约束时，每次审查都会重复报告相同问题，造成 PR 评论噪音。
- Documenting testing standards, valuable test criteria, and available fixtures in CLAUDE.md to improve test generation quality and reduce low-value test output
  - 生成测试前提供现有测试文件作为上下文，使 Claude 能够识别已覆盖的场景，避免建议重复测试。同时说明有价值的测试应满足哪些标准（如覆盖率要求、必须包含的场景类型）。

  - CLAUDE.md 中应记录测试标准（覆盖率要求、必须覆盖的场景类型）、有价值的测试判断标准和可用的 test fixtures 路径。CI 使用的 Claude 会读取这些配置，从而生成符合项目质量要求的测试。

## Task 3.1：CLAUDE.md 配置层次结构

### 这题怎么考

这道题考的是“规则为什么没有生效”和“规则应该放在哪里”。考试不会只问文件名，通常会给一个团队协作场景：某个开发者本地 Claude Code 行为正常，但新同事 clone 仓库后没有相同约定；或者某个子目录下规则与项目根规则不一致。判断时先看作用域：个人偏好放用户级，团队标准放项目级，局部约定放目录级或路径规则。实战中，错误地把团队标准写进 `~/.claude/CLAUDE.md` 是最常见的问题，因为它不会进入版本控制，也不会被其他成员加载。

模块化是第二个重点。`CLAUDE.md` 不是越长越好，过大的常驻上下文会稀释关键规则。项目级文件适合放全局标准，再用 `@import` 或 `.claude/rules/` 拆出测试、API、安全、部署等主题。遇到行为不一致时，不要只看文件是否存在，要用 `/memory` 验证当前会话实际加载了哪些 memory 文件。

### 三级层次结构

```
~/.claude/CLAUDE.md          ← 用户级（仅对该用户生效，不通过版本控制共享）
        ↓ 被继承/覆盖
.claude/CLAUDE.md
或 root CLAUDE.md             ← 项目级（提交到版本控制，所有团队成员共享）
        ↓ 被继承/覆盖
src/api/CLAUDE.md            ← 目录级（仅在该目录及其子目录生效）
```

**作用域继承规则：** 越具体的级别优先级越高，可覆盖上级规则。

> 新同事 clone 仓库后没有收到项目规范时，先检查规范是不是写在了**用户级** `~/.claude/CLAUDE.md`，而不是**项目级** `.claude/CLAUDE.md`。项目级才能通过 git 共享。

### CLAUDE.md 内容结构（推荐模板）

```markdown
# 项目名称

## 项目概述

简要描述项目用途和技术栈。

## 关键约定

- 代码风格：使用 ESLint + Prettier
- 命名规范：组件用 PascalCase，工具函数用 camelCase
- 测试：每个组件必须有对应的 .test.tsx 文件

## 常用命令

- `npm test` — 运行测试套件
- `npm run lint` — 代码检查
- `npm run build` — 构建生产版本

## 重要文件

- `src/api/client.ts` — API 客户端配置
- `src/components/` — 可复用组件
- `tests/fixtures/` — 测试数据

## 禁止操作

- 不要直接修改 dist/ 目录
- 不要提交 .env 文件
- 生产代码中不允许 console.log
```

### `@import` 语法（模块化拆分）

```markdown
# .claude/CLAUDE.md 主文件

@import ./standards/api-conventions.md
@import ./standards/testing-standards.md
@import ./standards/security.md
```

适用场景：规则文件过大，按主题拆分，保持主文件简洁。

### `.claude/rules/` 目录（主题化组织）

```
.claude/
├── CLAUDE.md               ← 项目级主配置（或使用 @import）
├── rules/
│   ├── testing.md          ← 测试规范（可含 paths 字段）
│   ├── api-conventions.md  ← API 约定
│   ├── security.md         ← 安全规则
│   └── deployment.md       ← 部署规则
└── commands/
    ├── review.md           ← /review 命令
    └── test-gen.md         ← /test-gen 命令
```

### 诊断命令

```bash
/memory    # 查看当前会话加载了哪些 memory 文件
           # 用于诊断：规则为何没有生效？是否加载了预期的 CLAUDE.md？
```

---

## Task 3.2：自定义斜杠命令与技能

### 这题怎么考

这道题考的是“什么时候用命令、什么时候用 Skill、什么时候写进 CLAUDE.md”。斜杠命令适合把团队经常执行的提示模板固化下来，例如 `/review`、`/test-gen`、`/release-check`。项目级命令放 `.claude/commands/` 并提交仓库，用户级命令放 `~/.claude/commands/` 只服务个人。题目如果强调“团队所有人都要能用”，答案通常指向项目级；如果强调“个人偏好或实验性流程”，答案通常指向用户级。

Skill 的重点是 `SKILL.md` frontmatter。`context: fork` 用来把长分析或探索性输出隔离到子智能体上下文，主对话只保留结论；`allowed-tools` 用来限制工具权限，尤其适合审查、报告、生成文档等不应改文件的流程；`argument-hint` 用来告诉调用者缺少哪些参数。实战中，Skill 更像“按需调用的流程包”，CLAUDE.md 更像“每次都应该遵守的长期标准”。

### 命令作用域

| 类型           | 存放位置                       | 共享范围         | 调用方式  |
| -------------- | ------------------------------ | ---------------- | --------- |
| **项目级命令** | `.claude/commands/review.md`   | 通过版本控制共享 | `/review` |
| **用户级命令** | `~/.claude/commands/review.md` | 仅个人使用       | `/review` |

### 斜杠命令文件格式

```markdown
# .claude/commands/security-review.md

---

description: 对当前代码变更进行安全审查
argument-hint: "可选：指定审查重点（如 'auth', 'injection'）"

---

请对以下代码进行安全审查，重点检查：

1. 注入漏洞（SQL、命令注入）
2. 认证/授权缺陷
3. 敏感数据暴露
4. 硬编码密钥或密码

$ARGUMENTS 如果提供了额外参数，重点关注：$ARGUMENTS
```

### SKILL.md Frontmatter 配置

```markdown
---
context: fork # 在隔离子智能体上下文中运行，防止大量输出污染主对话
allowed-tools: # 限制 Skill 执行期间可用工具（最小权限原则）
  - Read
  - Grep
  - Glob
argument-hint: "请输入要分析的模块名称" # 无参数时的提示
description: "分析指定模块的代码质量" # Skill 描述
---

# Skill 内容

分析 $ARGUMENTS 模块的代码质量...
```

### `context: fork` 使用原则

```
适合使用 context: fork（会产生大量探索性输出）：
- 代码库探索（读取大量文件）
- 头脑风暴和方案分析
- 批量文件处理
- 生成大型报告

不需要 context: fork（输出简短，不污染上下文）：
- 简短的格式化命令（/format）
- 单文件快速查询
- 简单的代码补全
```

### CLAUDE.md vs Skills 选择

| 适用情况                                         | 选择                         |
| ------------------------------------------------ | ---------------------------- |
| 始终需要加载的通用项目标准（代码风格、命名规范） | **CLAUDE.md**                |
| 按需调用的特定任务工作流（安全审查、测试生成）   | **Skills（斜杠命令）**       |
| 工作流步骤多、会产生大量中间输出                 | **Skills + `context: fork`** |

---

## Task 3.3：路径特定规则（Path-Specific Rules）

### 这题怎么考

这道题考的是“按目录加载规则”与“按文件模式加载规则”的区别。目录级 `CLAUDE.md` 适合规则边界与目录边界一致的情况，例如 `src/api/` 有专门 API 约定。路径特定规则适合规则横跨多个目录的情况，例如测试文件、Terraform 文件、Storybook 文件、迁移脚本散落在仓库不同位置。题目出现“files spread throughout the codebase”或“regardless of directory location”时，应优先想到 `.claude/rules/` 加 glob。

路径规则的收益不是只为了整理文件，而是减少不相关上下文。只有编辑匹配 `paths` 的文件时才加载对应规则，可以降低 token 使用量，也减少 Claude 被无关约定误导。实战中要把 glob 写得足够精确，例如测试规则用 `**/*.test.tsx`，基础设施规则用 `terraform/**/*` 或 `infra/**/*`，避免让规则在不相关文件中误触发。

### `.claude/rules/` 文件的 `paths` 字段

通过 YAML frontmatter 的 `paths` 字段，使规则仅在匹配的文件上生效：

```yaml
# .claude/rules/terraform.md
---
paths:
  - "terraform/**/*"
  - "infrastructure/**/*"
---
# Terraform 规范
- 所有资源必须添加 `Environment` 和 `Owner` 标签
- 变量定义必须包含 description 和 type
- 不允许硬编码 AWS Account ID
- 使用 `data` 源代替硬编码 AMI ID
```

```yaml
# .claude/rules/testing.md
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
  - "**/*.spec.ts"
---
# 测试文件规范
- 每个测试必须有 describe 块
- Mock 函数必须在 beforeEach 中重置
- 不允许使用 test.only（会阻止其他测试运行）
```

### Glob 模式 vs 目录级 CLAUDE.md

| 场景                                            | 推荐方案                      | 原因                     |
| ----------------------------------------------- | ----------------------------- | ------------------------ |
| 规则按**目录位置**划分（`src/api/` 有特殊规则） | 目录级 `src/api/CLAUDE.md`    | 目录边界清晰             |
| 规则按**文件类型**划分（测试文件散落各处）      | `.claude/rules/` + glob 模式  | 测试文件不集中在一个目录 |
| 规则跨越多个不相关目录                          | `.claude/rules/` + 多个 paths | glob 可以匹配多个位置    |

**经典考题场景：**  
测试文件散落在整个代码库（`Button.test.tsx` 紧邻 `Button.tsx`，而非集中在 `tests/` 目录下）→ 使用 `**/*.test.tsx` glob 模式，目录级 CLAUDE.md 无法覆盖这种场景。

---

## Task 3.4：Plan Mode vs 直接执行

### 这题怎么考

这道题考的是风险和不确定性的判断。Plan Mode 适合大范围、多文件、多方案、有架构影响或高风险的任务，因为它允许先探索代码库、比较方案、确认设计，再开始修改。直接执行适合边界明确的小任务，例如根据清晰堆栈修复单文件 bug，或给一个函数增加简单校验。判断标准不是“任务听起来大不大”，而是是否存在多个合理方案、是否需要先理解依赖关系、是否有较高返工成本。

Explore subagent 是为了隔离发现阶段的噪音。大型迁移、遗留系统梳理、跨服务依赖分析会产生大量中间输出，直接塞进主对话会消耗上下文并影响后续决策。更好的流程是：Plan Mode 做调查和方案，Explore subagent 汇总发现，用户确认后再用直接执行完成实施。

### 判断矩阵

| 使用 **Plan Mode**                         | 使用**直接执行**         |
| ------------------------------------------ | ------------------------ |
| 大规模变更（影响 10+ 文件）                | 单文件 bug 修复          |
| 存在多个有效实现方案，需要先讨论           | 已知正确方法，范围清晰   |
| 架构决策（微服务拆分、数据库迁移、库迁移） | 添加单个字段验证         |
| 需要先探索代码库再决定怎么改               | 有清晰堆栈跟踪的简单修复 |
| 高风险操作（删除数据、不可逆变更）前预览   | 新增功能（低破坏性）     |

### 如何开启 Plan Mode

```bash
# 方式 1：在对话中使用 /plan 命令进入
/plan

# 方式 2：在提示前加 "Think about..."（隐式触发计划模式）
# 会让 Claude 先思考再行动
```

### 组合使用策略

```
Plan Mode 探索阶段：
→ Claude 分析代码库结构（只读，不写入）
→ 理解依赖关系
→ 提出实施方案（用户可在确认前修改/拒绝）

用户确认计划后：
→ 直接执行阶段（按计划实施具体变更）
```

### Explore 子智能体

用于隔离详细发现输出，防止主对话上下文耗尽：

- **适合**：遗留代码库探索、大量文件读取、不知道代码库结构
- **返回**：结构化摘要而非完整探索输出
- **效果**：主对话只看到摘要，保留上下文容量

---

## Task 3.5：迭代优化技术

### 这题怎么考

这道题考的是如何让 Claude 从“模糊执行”进入“可验证改进”。当自然语言描述被反复误解时，最有效的反馈不是继续写更长的说明，而是给 2-3 个具体输入/输出示例。示例能直接定义转换边界、格式要求和异常处理，比抽象规则更不容易被误读。

测试驱动迭代是另一条主线。先写覆盖期望行为、边界情况和性能要求的测试，再把失败信息提供给 Claude，可以把“我觉得不对”变成可复现的修复目标。Interview pattern 适合需求不完整或领域不熟时使用，让 Claude 先提问缓存失效、失败模式、权限边界、数据一致性等隐藏约束。多个问题是否一次给出，取决于它们是否相互影响：交互问题一次讲清，独立问题顺序迭代。

### 四种核心技术

**1. 具体输入/输出示例**

当文字描述被反复误解时，提供 2-3 个具体示例比长篇说明更有效：

```
输入: "2024-01-15T10:30:00Z"
输出: "2024年1月15日 10:30"

输入: "1705311000"（Unix时间戳）
输出: "2024年1月15日 10:30"

输入: null 或空字符串
输出: "日期未知"
```

**2. 测试驱动迭代（TDD）**

先写测试 → 分享测试失败信息 → 引导 Claude 逐步修复：

- 测试失败是高质量的精确反馈，比自然语言描述更准确
- 明确包含边缘情况和性能要求
- 每次迭代只修复一个失败的测试（避免大范围变更）

**3. 访谈模式（Interview Pattern）**

让 Claude 先提问，暴露开发者没预料到的设计考量：

```
开发者："实现一个缓存系统"
Claude 应该先问：
- 缓存失效策略是什么？（TTL？LRU？手动失效？）
- 并发访问时需要加锁吗？
- 缓存命中率需要监控吗？
- 内存上限是多少？
```

适合场景：缓存策略设计、数据迁移方案、容错机制

**4. 批量 vs 顺序修复**

| 情况                             | 策略                 | 原因                   |
| -------------------------------- | -------------------- | ---------------------- |
| 多个问题相互影响（修复A会影响B） | 单条消息提供所有问题 | 避免修复A又破坏B       |
| 多个问题相互独立                 | 顺序迭代修复         | 每次确认结果，减少风险 |

---

## Task 3.6：CI/CD 集成

### 这题怎么考

这道题考的是把 Claude Code 从交互式工具变成自动化流水线组件。CI 中必须使用 `-p` 或 `--print` 非交互模式，否则任务可能等待输入而挂起。自动化结果最好使用 `--output-format json` 和 `--json-schema` 约束结构，方便后续脚本把 findings 转成 PR inline comments、检查状态或报告。

CI 场景还强调上下文和隔离。流水线调用的 Claude 仍然需要项目背景，测试标准、fixture 约定、审查标准应写入 CLAUDE.md，而不是只存在于开发者本地对话里。审查最好用独立会话，而不是同一个生成代码的会话，因为生成者自审更容易继承原来的假设。重复审查时要带上已有 findings，并明确只报告新增或仍未解决的问题，避免在 PR 中制造重复评论。

### 非交互模式配置

```bash
# CI 中必须使用 -p/--print（Print Mode）
# 否则 Claude Code 会等待交互输入，导致 CI 流水线永久挂起
claude -p "审查这个 PR 的安全漏洞" \
       --output-format json \
       --json-schema ./schemas/review-result.json

# 使用 heredoc 传递多行内容
claude -p "$(cat <<'EOF'
审查以下代码变更：
$(git diff HEAD~1)
重点检查：SQL注入、XSS、CSRF漏洞
EOF
)"
```

### 关键 CLI 参数

| 参数                     | 作用                                         | CI 使用场景          |
| ------------------------ | -------------------------------------------- | -------------------- |
| `-p` / `--print`         | **Print Mode**：输出结果后立即退出（非交互） | **所有 CI 场景必用** |
| `--output-format json`   | 结构化 JSON 输出                             | 供机器解析结果       |
| `--json-schema <file>`   | 约束 JSON 输出结构                           | 强制特定输出格式     |
| `--headless`             | 禁用 UI 交互元素                             | 自动化流水线         |
| `--model <model>`        | 指定使用的 Claude 模型                       | 控制成本/速度权衡    |
| `--allowedTools <tools>` | 限制可用工具（逗号分隔）                     | 最小权限原则         |
| `--resume <session>`     | 恢复命名会话                                 | 跨步骤保留上下文     |

### CI 代码审查最佳实践

```bash
# 推荐：独立的审查实例（而非自查）
# 在 .github/workflows/review.yml 中：

- name: Claude Code Review
  run: |
    claude -p "审查这次 PR 的变更" \
           --output-format json \
           --allowedTools "Read,Grep,Glob" \  # 只读工具，不允许写入
           > review-result.json

# 将现有测试文件放入上下文，避免建议重复测试
- name: Claude Code Review with Context
  run: |
    claude -p "根据现有测试文件（tests/目录），审查新代码并建议缺失的测试用例。
               不要重复已有的测试。" \
           --output-format json > suggestions.json
```

**CI 审查注意事项：**

```
推荐做法：
- 使用独立实例（独立会话）进行审查，不是在编写代码的同一会话中自查
- 重新审查时，把已有发现传入上下文（只报新增或未修复的问题）
- 在 CLAUDE.md 中记录测试标准和可用 fixture（CI 也会读取）
- 结构化输出 + JSON Schema（方便 CI 系统解析）

避免做法：
- 不加 -p 标志（CI 流水线会永久挂起等待输入）
- 同一会话既写代码又审查（降低客观性，模型有生成偏见）
- 不限制工具权限（CI 审查实例不应有写权限）
```

### CLAUDE.md 在 CI 中的作用

CI 中的 Claude Code 实例也读取 `.claude/CLAUDE.md`，可通过它注入项目上下文：

```markdown
## CI 审查配置

### 测试标准

- 测试覆盖率要求：>= 80%
- 所有异步函数必须有错误处理
- 禁止 console.log 在生产代码中

### 可用测试 Fixture

- `tests/fixtures/mock-user.ts` — 用户数据 mock
- `tests/fixtures/mock-api.ts` — API 响应 mock

### 不需要建议的内容

- 性能优化（非本次 PR 范围）
- 样式/格式问题（由 Prettier 自动处理）
```

---

## 讲透这一域：Claude Code 配置、工作流与团队工程化

Domain 3 考的是 Claude Code 如何进入真实开发团队，而不是单个开发者临时和模型聊天。它覆盖配置层级、团队共享规则、斜杠命令、Skills、路径规则、Plan Mode、迭代优化和 CI/CD。把这些内容串起来看，核心问题是：怎样让 Claude Code 在不同人、不同目录、不同任务、不同自动化环境里保持一致、可控、可复现。考试题通常会给出一个团队协作问题，例如新同事没有加载规则、测试文件规则失效、CI 卡住、模型重复建议已有测试，然后让你选择最符合工程实践的修复。

### 一、CLAUDE.md 是团队记忆，不是个人笔记

CLAUDE.md 的层级是 Domain 3 的基础。用户级 `~/.claude/CLAUDE.md` 只对个人有效，适合个人偏好，例如常用语言、个人解释风格、私人命令习惯。项目级 `.claude/CLAUDE.md` 或 root `CLAUDE.md` 应提交到版本控制，适合团队共享标准，例如代码风格、测试标准、架构约定、禁止事项。目录级 `src/api/CLAUDE.md` 适合局部约定，例如 API 层错误处理、数据库访问模式、前端组件命名规范。

考试里“新同事 clone 仓库后 Claude 不遵守规范”几乎一定是在考作用域。规范如果写在某个老员工的用户级文件里，新同事不会加载；正确修复是移动到项目级或目录级，并提交到版本控制。另一个常见场景是某个子目录规则过多或过少，这时要判断规则边界是目录边界还是文件类型边界。目录边界清楚时用目录级 CLAUDE.md，文件类型跨目录散落时用 `.claude/rules/` 和 glob。

CLAUDE.md 也需要模块化。大型仓库把所有内容塞进一个巨大文件，会让上下文臃肿，也让维护成本升高。`@import` 可以把 testing、security、api、deployment 等规则拆成独立文件，再由相关 package 的 CLAUDE.md 选择性导入。这样做体现“只加载相关规则”的原则。考试中如果题目说 monolithic CLAUDE.md 太长、规则互相干扰，正确方向通常是拆分 topic-specific files 或使用 `@import`。

### 二、/memory 是诊断工具，不是装饰命令

当 Claude Code 行为不一致时，不要凭感觉猜规则是否加载。`/memory` 可以查看当前会话实际加载了哪些 memory 文件。比如一个开发者在 `src/api` 目录里看到 API 规则生效，另一个在项目根目录里没有看到，可能是路径或层级导致。`/memory` 能把“模型不听话”的问题转成可诊断的配置问题。

这个思路对考试很有用。题目如果描述“有时遵守规则，有时不遵守”“不同目录表现不同”“新会话加载内容不一致”，不要急着改 prompt，先检查 memory hierarchy、路径作用域、规则是否提交、当前工作目录是否命中。Claude Code 是有配置系统的，工程化问题要用配置诊断解决。

### 三、Slash Commands 和 Skills 的区别

斜杠命令适合把团队常用操作固定下来，例如 `/review-module`、`/generate-tests`、`/write-migration`。项目级命令放 `.claude/commands/`，团队共享；用户级命令放 `~/.claude/commands/`，个人使用。命令的价值是减少重复提示，让团队对同一类任务使用同一套入口。

Skills 更像可复用的任务工作流，通常带 `SKILL.md` 和 frontmatter。`context: fork` 是关键选项，适合代码库探索、批量分析、头脑风暴这类会产生大量上下文的任务。fork 后主对话只收到摘要，不会被子任务的中间发现污染。`allowed-tools` 用来限制 Skill 执行时能用的工具，避免一个本应只读的分析 Skill 执行写操作。`argument-hint` 用来提示调用者提供必要参数，减少误用。

选择 CLAUDE.md 还是 Skill，要看内容是否“总是需要”。团队命名规范、测试标准、架构原则应常驻 CLAUDE.md；安全审查、迁移生成、复杂分析流程应做成 Skill 按需调用。把所有任务流程都写进 CLAUDE.md 会浪费上下文；把团队基本规范都做成 Skill 又会导致默认不生效。考试题常用这个边界来区分好设计和过度配置。

### 四、路径特定规则解决“跨目录同类文件”问题

`.claude/rules/` 的 `paths` frontmatter 是为条件加载设计的。测试文件可能分布在 `src/components/Button.test.tsx`、`packages/api/user.spec.ts`、`apps/web/*.test.ts`，这时目录级 CLAUDE.md 很难覆盖。用 glob `**/*.test.tsx` 或 `**/*.spec.ts` 能按文件类型加载测试规则。Terraform、SQL migration、React components、GitHub Actions 也常适合这种模式。

路径规则的价值是减少无关上下文。编辑 Terraform 文件时加载基础设施规则，编辑测试文件时加载测试规则，编辑 UI 组件时加载设计系统规则。模型看到的规则越相关，越容易遵守。考试题如果说“测试规则散落无法覆盖”，正确答案通常是 path-specific rules，而不是在每个目录复制 CLAUDE.md。

### 五、Plan Mode 是风险控制，不是拖慢流程

Plan Mode 适合复杂、多文件、多方案、高风险任务。比如库迁移影响 45 个文件、微服务拆分、数据库迁移、认证架构调整、基础设施改造。它的价值是先读代码、理解依赖、比较方案，再让用户确认方向。直接执行适合范围清晰的小改动，例如单函数校验、明确堆栈的 bug、一个字段的处理逻辑。

Plan Mode 与 Explore subagent 也有关。大型探索会产生大量发现，如果全部留在主对话，后续实现时上下文会拥挤。Explore subagent 负责读很多文件并返回摘要，主对话保留决策和计划。这与 Domain 5 的上下文管理呼应。考试里看到“大型遗留代码库探索导致上下文耗尽”，通常要选择 Explore subagent 或阶段性总结，而不是继续把所有文件读进主上下文。

一个成熟流程是“Plan for investigation, direct execute for implementation”。先在 Plan Mode 里探索并确认方案，用户批准后再直接执行具体改动。不要把 Plan Mode 理解成只能计划不能实施，也不要把 direct execution 用在高风险未知任务上。

### 六、迭代优化要给模型可验证反馈

Claude Code 的迭代不是反复说“再好一点”。当自然语言描述不稳定时，给 2-3 个输入/输出示例；当实现有明确行为要求时，先写测试，再把失败信息反馈给 Claude；当领域不熟时，用 interview pattern 让 Claude 先问问题。可验证反馈比主观评价更有用。

测试驱动迭代尤其重要。先写测试覆盖主路径、边界、异常、性能，再让 Claude 实现。失败时把具体测试输出给它，而不是只说“还有 bug”。如果多个问题相互影响，例如数据迁移同时涉及 null、日期格式、重复记录，要一次性说明，因为分开修可能互相破坏。如果问题相互独立，例如文案错误和单个边界校验，则可以顺序处理。

interview pattern 适合需求不完整的场景。让 Claude 在实现前问你关于缓存失效、权限模型、失败回滚、兼容性、数据迁移策略的问题。这样做不是浪费时间，而是把隐藏需求显式化。考试中如果题目说开发者对领域不熟、可能遗漏设计考虑，正确方向通常是让 Claude 先提问，而不是直接实现。

### 七、CI/CD 中的 Claude Code 是非交互、结构化、最小权限

CI 环境里 Claude Code 必须用 `-p` 或 `--print` 非交互运行，否则 pipeline 会等待输入而挂起。输出应使用 `--output-format json` 和 `--json-schema`，方便自动发布 PR 评论、生成检查结果、过滤严重级别。CI 审查实例应尽量只读，例如允许 `Read,Grep,Glob`，避免审查任务写代码或执行危险命令。

CLAUDE.md 在 CI 中同样重要。测试标准、fixture 位置、审查标准、不需要报告的内容，都应写进项目配置。否则 CI 中的 Claude 只能根据通用经验建议测试，容易重复已有测试或提出低价值建议。如果要让 Claude 生成缺失测试，应提供现有测试文件上下文，让它避免重复覆盖已存在场景。

独立审查实例也很容易被考到。同一 Claude 会话刚生成代码，再要求它审查自己，会受生成时的推理上下文影响，更容易合理化自己的选择。CI 审查应使用独立实例，不带生成过程，只看 diff、上下文和标准。重新审查时还要传入旧 findings，并要求只报告新增或仍未解决的问题，避免 PR 评论重复刷屏。

### 八、Domain 3 的答题框架

遇到 Claude Code 工作流题，可以先判断问题发生在哪一层。配置不共享，是 CLAUDE.md 作用域问题；某类文件规则不加载，是 path-specific rules 问题；重复任务提示不一致，是 slash command 或 Skill 问题；长探索污染上下文，是 `context: fork` 或 Explore subagent 问题；大规模修改风险高，是 Plan Mode 问题；CI 卡住，是缺少 `-p`；CI 输出难解析，是缺少 JSON/schema；审查不客观，是同会话自审问题。

最常见错误答案包括：把团队规则放用户级；把个人 Skill 覆盖团队 Skill；对散落测试文件使用目录级 CLAUDE.md；复杂迁移直接执行；CI 不使用 print mode；CI 输出自然语言再让脚本解析；同一会话生成又审查；不提供现有测试导致重复建议。只要把“作用域、上下文、权限、结构化输出”四个词放在脑中，Domain 3 的大多数题目都能定位。

### 九、从团队落地角度看 Claude Code

个人使用 Claude Code 时，最重要的是提示是否清楚；团队使用 Claude Code 时，最重要的是一致性。一个团队如果每个人都有自己的规则、自己的命令、自己的审查标准，Claude Code 的输出就会像不同开发者的个人习惯一样分裂。Domain 3 关注的正是如何把个人提示升级为团队工作流：规则进入版本控制，命令进入项目目录，Skills 有明确工具权限，CI 使用结构化输出，审查标准写入 CLAUDE.md。

团队落地的第一步是整理“默认规则”和“按需流程”。默认规则包括代码风格、测试目录、错误处理约定、安全底线、提交信息格式。这些应写进项目级 CLAUDE.md 或目录级 CLAUDE.md。按需流程包括生成迁移、安全审查、重构计划、发布说明、性能分析，这些适合写成 slash command 或 Skill。这样开发者不需要每次复制长提示，也不会把一次性任务塞进常驻上下文。

第二步是定义权限边界。Claude Code 可以读文件、写文件、运行命令、连接 MCP。能力越强，越要按场景限制。代码审查命令通常只需要 Read/Grep/Glob；生成测试可能需要 Write/Edit；部署检查可能需要 Bash 但不应有生产写权限。`allowed-tools` 不是形式主义，而是把任务风险和工具能力对齐。考试题如果出现“审查流程误修改文件”或“分析 Skill 执行了危险命令”，正确方向就是收紧工具权限。

第三步是把反馈纳入流程。Claude Code 不应只在开发者本地运行，也可以进 PR、CI、代码评审、测试生成、文档更新。进入自动化后，输出必须结构化，失败必须可解析，重复评论必须避免。自然语言适合人读，不适合机器处理。CI 中用 JSON/schema 输出 findings，再由脚本映射成 inline comments，是可维护方式。

### 十、Plan Mode 与直接执行的组织规范

团队最好明确哪些任务必须先计划。比如修改认证、支付、权限、数据库 schema、公共 API、跨包依赖，都应先进入 Plan Mode；文案、单点 bug、简单校验可以直接执行。这个规范避免两个极端：所有事情都规划导致效率低，所有事情都直接改导致风险高。Plan Mode 的产物也应该有质量要求：列出影响范围、候选方案、风险、验证步骤、回滚策略。

直接执行也不是盲目执行。即使是小改动，也要先读相关文件、确认测试、修改后验证。Claude Code 的优势是能执行命令和编辑文件，但这也意味着它应该像工程师一样形成闭环：理解、修改、运行测试、解释结果。考试题如果只强调“让 Claude 直接修”而没有验证步骤，通常不是最佳工程答案。

### 十一、Skills 的设计质量

一个好的 Skill 应该像内部工具文档一样清楚。开头说明它解决什么问题，适合什么场景，不适合什么场景；frontmatter 限制工具权限；如果参数必要，用 `argument-hint` 提示；如果会大量探索，用 `context: fork`；正文给出步骤、输出格式、质量标准和失败处理。不要写一个巨大的万能 Skill，也不要让 Skill 依赖隐含上下文。

个人 Skill 和项目 Skill 要命名区分。个人可以有 `/my-review-style`，但不应覆盖团队 `/review`。团队 Skill 要稳定，变更应像代码一样 review，因为它影响所有成员的 Claude Code 行为。考试可能不会直接问 review 流程，但会通过“个人改动影响队友”来考作用域和命名隔离。

### 十二、Claude Code 与 MCP 的结合

Domain 3 和 Domain 2 的交叉点是 MCP 集成。Claude Code 通过 MCP 获得外部系统能力，例如 Jira、GitHub、数据库、文档库。项目级 MCP 配置让团队共享这些能力，但 CLAUDE.md 仍要告诉 Claude 何时使用它们。例如“查需求时优先使用 Jira MCP，不要用 Grep 搜本地缓存”“数据库 schema 通过 MCP resource 查看，不要运行生产查询”。没有这些规则，工具虽然接入了，模型不一定会正确选择。

MCP 也会影响 Plan Mode。计划阶段可能需要读 issue、看 schema、查文档；执行阶段可能需要改代码和跑测试。不同阶段的工具权限可以不同。成熟工作流会把探索、计划、执行、验证拆开，每一步只给必要工具。这样既降低风险，也让审计更清楚。

### 十三、考试中的场景化判断

如果题目是“新成员没有项目约定”，答案看配置层级；如果是“测试规则只在某目录生效”，答案看 path-specific rules；如果是“Skill 输出太多导致主对话混乱”，答案看 `context: fork`；如果是“CI 等待输入”，答案看 `-p`；如果是“PR 评论无法自动定位”，答案看 JSON/schema 和 inline comments；如果是“同一 Claude 生成后审查漏问题”，答案看独立实例。

Domain 3 的本质是把 Claude Code 从一个聊天助手变成工程系统组件。工程系统组件需要配置、作用域、权限、版本控制、自动化、验证和审计。只要答案体现这些要素，就比单纯“改 prompt”更接近官方思路。

### 十四、从失败案例反推正确配置

一个典型失败案例是：团队在个人 `~/.claude/CLAUDE.md` 中写了“所有测试必须使用 fixture”，老成员本地运行正常，新成员和 CI 都不遵守。正确修复不是让新成员复制个人文件，而是把规则移动到项目级 CLAUDE.md，并把 fixture 列表写清楚。另一个失败案例是：团队把所有前端、后端、Terraform、测试规则都塞进根目录 CLAUDE.md，导致 Claude 编辑任何文件都看到大量无关规则。正确修复是拆成 `.claude/rules/` 和目录级 CLAUDE.md，让规则按路径加载。

还有一种失败是 Skill 没有限制工具。比如安全审查 Skill 本来只需要读代码，却允许 Bash 和 Write，结果模型为了“修复”问题直接改文件。正确设计是审查 Skill 只读，修复 Skill 另设，并要求用户确认。把“审查”和“修改”拆成不同 workflow，是工程上更清楚的权限边界。

CI 失败也很常见：没有 `-p` 导致流水线挂起；输出 Markdown 导致脚本解析失败；没有带旧 findings 导致重复评论；没有提供现有测试导致建议重复测试；同一会话生成又审查导致漏掉问题。这些都不是模型能力问题，而是工作流设计问题。Domain 3 的题目经常用这些现象考你能否把 Claude Code 放进自动化系统。

### 十五、形成自己的复习地图

复习 Domain 3 时，可以按“配置、复用、条件加载、执行模式、迭代、自动化”六个词串起来。配置对应 CLAUDE.md 层级；复用对应 commands 和 Skills；条件加载对应 path-specific rules；执行模式对应 Plan Mode 与 direct execution；迭代对应 examples、tests、interview pattern；自动化对应 CI/CD。每个词都要能说出适用场景、反模式和诊断方法。

如果考试题描述的是“人和人之间不一致”，优先想到作用域和版本控制；如果描述的是“文件类型规则不一致”，想到 glob paths；如果描述的是“上下文污染”，想到 fork 和 Explore；如果描述的是“高风险变更”，想到 Plan Mode；如果描述的是“输出给机器用”，想到 JSON/schema；如果描述的是“审查质量”，想到独立实例。这样把症状映射到机制，比死背命令更可靠。

最后要记住，Claude Code 的配置不是越多越好，而是越贴近工作流越好。规则太少，模型不知道项目习惯；规则太多，模型被无关信息干扰。好的团队会定期清理 CLAUDE.md、commands 和 skills，删除过时约定，把重复提示沉淀为命令，把高风险流程拆成计划、执行、验证三个阶段。这样的配置体系才适合长期维护。

实际备考时，可以把每个 Claude Code 功能都放进“谁共享、何时加载、能做什么、如何验证”四个问题里。CLAUDE.md 解决共享规则，rules 解决条件加载，commands/skills 解决复用流程，allowed-tools 解决能做什么，Plan Mode 和 CI 验证解决如何安全落地。这样复习不会变成背路径，而是理解每个机制服务的团队工程目标，也能在场景题里快速判断应该改配置、改权限、改流程还是改自动化输出。真正掌握后，你会把 Claude Code 看成团队开发平台的一部分，而不是一次性问答工具，并能围绕协作、风险、权限和验证设计稳定流程，持续提升团队交付质量和协作一致性。

我会把它当成团队约定来审：谁能用、在哪加载、能不能写文件、失败后谁看结果。

---

## 临考速查

1. **新同事无规范** → 检查规范是否写在了**用户级** `~/.claude/CLAUDE.md`（而非项目级 `.claude/CLAUDE.md`）
2. **项目级 vs 用户级** → `.claude/CLAUDE.md` 提交到 git，团队共享；`~/.claude/CLAUDE.md` 不共享
3. **团队共享命令** → `.claude/commands/`（版本控制）；个人命令 → `~/.claude/commands/`
4. **`context: fork`** → 防止 Skill 的大量输出污染主对话上下文
5. **测试文件散落各处** → `.claude/rules/` + glob 模式（`**/*.test.tsx`），不能用目录级 CLAUDE.md
6. **Plan Mode** → 大规模变更、多文件影响、多个方案、高风险操作前使用
7. **CI 挂起** → 加 `-p`/`--print` 标志（Print Mode = 非交互）
8. **CI 结构化输出** → `--output-format json` + `--json-schema`
9. **CI 最小权限** → `--allowedTools "Read,Grep,Glob"`（审查只需读权限）
10. **独立审查实例** → 比同会话自查更客观，减少生成偏见
11. **/memory 命令** → 诊断当前会话加载了哪些配置文件
