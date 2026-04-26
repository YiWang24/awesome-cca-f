# CCA-F Simulation Questions — Domain 3: Claude Code Configuration and Workflows

> There are 60 questions total, and the recommended completion time is 75 minutes. single-choice questions.

---

## Task 3.1 CLAUDE.md level configuration (Q1–Q12)

**Q1.** How many levels is the hierarchy of CLAUDE.md in Claude Code generally divided into? What is the order of priority of these levels?

A) There are only two levels: global level and project level. The priority is project level > global level.
B) Three levels: user global level, project level and directory level, the priority is directory level > project level > user global level
C) There are only directory level and project level, directory level has higher priority
D) Five levels include workspace level, project level, team level, directory level and file level

> **Answer: B**
> Explanation: Claude Code supports a three-layer CLAUDE.md configuration system, which is located in ~/.claude/CLAUDE.md (user global), project root directory CLAUDE.md (project level) and CLAUDE.md in each directory (directory level). The priorities from high to low are directory level, project level, and user global level, that is, the most recent configuration takes priority over distant configurations.

**Q2.** Suppose a coding style specification is defined in the user's global ~/.claude/CLAUDE.md, but different standards are set for the same specification in a project's CLAUDE.md. When Claude Code is run in this project, which specification will be used?

A) Always adopt user global specifications to ensure consistency
B) Adopt project-level specifications because it is closer to the current work context
C) Prompt the user to choose
D) Both specifications are applied, and the user can choose compatibility mode

> **Answer: B**
> Resolution: In the CLAUDE.md hierarchy, configuration files closer to the execution context have higher priority. Project-level CLAUDE.md overrides user-global-level configuration to support project-specific conventions and specifications.

**Q3.** In a large monorepo, there are three independent microservices. Each microservice has different coding styles and project conventions. How should the CLAUDE.md hierarchy be used to optimally support this scenario?

A) Create a CLAUDE.md in the project root directory, containing the configuration of all three services
B) Create an independent CLAUDE.md in each microservice directory to allow directory-level configuration to provide specific specifications for each service
C) Create a separate .claude/ directory for each microservice
D) Use global CLAUDE.md to distinguish different services through conditional comments

> **Answer: B**
> Explanation: In a monorepo scenario, you should make full use of the flexibility of directory-level CLAUDE.md, create an independent CLAUDE.md file in each microservice directory, and let Claude Code load the most relevant configuration according to the current working directory. This keeps the overall project consistent while allowing for specific agreements on each service.

**Q4.** What is the role of the @import directive in CLAUDE.md? Please choose the most accurate description.

A) @import is used to import configuration fragments from other CLAUDE.md files, supporting parameterization and conditional import
B) @import is only used to import external document links and cannot import configurations
C) @import copies content from other files to the current file. After importing, the contents of the two files are exactly the same.
D) @import is a deprecated feature, modern Claude Code recommends using a separate rule file

> **Answer: A**
> Explanation: @import is a key directive in CLAUDE.md that allows the import of reusable configuration fragments from other CLAUDE.md or configuration files. This supports modular configuration management and the DRY principle to avoid configuration duplication.

**Q5.** Assume that there is an @import directive in CLAUDE.md in the project root directory that introduces a shared code style configuration file. When a directory-level CLAUDE.md is created in the src/ directory, how will this imported configuration behave?

A) The imported configuration will be completely overwritten by the directory-level CLAUDE.md
B) The imported configuration is ignored and only directory-level CLAUDE.md is loaded.
C) Directory-level CLAUDE.md can inherit and selectively override imported configurations
D) The two configurations will conflict, causing the loading to fail.

> **Answer: C**
> Explanation: Claude Code's inheritance mechanism allows directory-level configuration to inherit the parent's imported configuration and selectively override some of the settings. This provides flexible configuration management capabilities.

**Q6.** What types of information should the configuration content defined in CLAUDE.md mainly contain?

A) Contains only Claude's system prompt words and behavior specifications
B) Contains only project dependencies and build configurations
C) Contextual information such as coding conventions, best practice guidelines, project context, tool usage specifications, etc.
D) Complete project architecture documentation and all source code

> **Answer: C**
> Explanation: CLAUDE.md should contain the contextual information Claude Code needs to understand the project, such as coding conventions, technology stack descriptions, special specifications, etc., rather than the complete technical documentation or build configuration of the project.

**Q7.** In a certain project, the user-global CLAUDE.md contains common security best practices, and the project-level CLAUDE.md is extended for specific security requirements. How will these two levels of security regulations be handled?

A) Only project-level specifications are used, global specifications are completely ignored
B) Only use global specifications to ensure consistency across all projects
C) Project-level specifications extend and can cover global specifications to form the final security policy of the project
D) If the two specifications conflict, Claude Code will throw an error

> **Answer: C**
> Explanation: This is the ideal use of CLAUDE.md inheritance and extension. Users can define general specifications at the global level as a basis, and project-level configurations can be specifically extended on this basis to form complete project-specific policies.

**Q8.** What are the essential differences between CLAUDE.md and the rule files in the .claude/rules/ directory?

A) Both are exactly the same, except for the different storage locations.
B) CLAUDE.md is a declarative global context configuration. The rule files in .claude/rules/ usually have scope restrictions for specific file modes (globs).
C) CLAUDE.md is used to configure tools, .claude/rules/ is used to define commands
D) CLAUDE.md has been deprecated and .claude/rules/ should be used instead

> **Answer: B**
> Explanation: The two are complementary. CLAUDE.md provides global context, while the rules files in .claude/rules/ provide more fine-grained control by applying targeted rules to specific sets of files via glob patterns.

**Q9.** A project needs to use a relaxed coding style during the development phase, but needs to switch to strict code review practices before release. How can I manage this requirement through the CLAUDE.md hierarchy without modifying the code base?

A) Create two separate CLAUDE.md files, one for each stage
B) Use conditional syntax to specify specifications for different stages in project CLAUDE.md
C) Maintain two versions in user-global or project-level CLAUDE.md, and change the effective configuration by switching
D) Use environment variables combined with CLAUDE.md to load different configuration versions according to the environment

> **Answer: D**
> Explanation: Although the code base is not required to be modified in the question, the best solution is to use Claude Code's support for environment variables to conditionally apply different sets of specifications based on environment variables in the global or project CLAUDE.md, allowing flexible switching.

**Q10.** In what scenarios should you choose to define configurations in directory-level CLAUDE.md instead of project-level CLAUDE.md?

A) should always be defined in the project-level CLAUDE.md to keep it simple
B) When a specific directory or sub-project has different conventions or special needs from the project as a whole
C) Only used when the global CLAUDE.md cannot meet the needs
D) Never use directory-level CLAUDE.md, which can lead to configuration confusion

> **Answer: B**
> Explanation: Directory-level CLAUDE.md is designed to support this scenario. Creating CLAUDE.md in a directory or component can provide precise contextual control when that directory or component has special coding conventions, framework-specific specifications, or a unique context.

**Q11.** Assume that the project has an @import "shared-rules.md" directive in the project-level CLAUDE.md, and shared-rules.md itself contains other @import directives. Is this type of chained import supported, and what are the possible risks?

A) Not supported, @import can only introduce first-level configuration files
B) Support chain import, no special risks
C) Chained import is supported, but care must be taken to avoid circular references, which may lead to infinite recursion in the configuration.
D) Supported but with performance overhead, it is not recommended to use import chains with more than three levels.

> **Answer: C**
> Explanation: Claude Code supports nested @import directives, but architects need to pay attention to prevent circular dependencies. For example, shared-rules.md cannot directly or indirectly import project-level CLAUDE.md, otherwise it will cause circular dependencies and configuration loading failures.

**Q12.** In a cross-team project, where the infrastructure team maintains a project-level CLAUDE.md, the application development team needs to add specific specifications in their respective code directories. What are the best practices?

A) The application development team directly modifies the project-level CLAUDE.md
B) Each application team creates an independent CLAUDE.md in its directory. These configurations inherit and extend the project-level specifications.
C) Create a shared configuration server to manage all configurations
D) Each team maintains its own global ~/.claude/CLAUDE.md

> **Answer: B**
> Explanation: This is a standard model for multi-team collaboration. Project-level CLAUDE.md is maintained by the infrastructure team as a basic configuration, and each application team creates CLAUDE.md in its directory to maintain consistency and allow team-specific customization.

---

## Task 3.2 Custom slash command (Q13–Q22)

**Q13.** Where are custom slash commands stored in Claude Code?

A) Stored in commands.json in the project root directory
B) Stored in the .claude/commands/ directory, each command corresponds to a .md file
C) Stored in the ~/.claude/commands/ global directory, shared by all projects
D) Stored in a dedicated commands section of CLAUDE.md

> **Answer: B**
> Explanation: Custom slash commands are stored in the .claude/commands/ directory of the project, and each command corresponds to a Markdown file. This facilitates version control and team sharing.

**Q14.** What does the $ARGUMENTS placeholder in the slash command do?

A) $ARGUMENTS is a system variable that automatically obtains Claude’s version information.
B) $ARGUMENTS is a placeholder, it will be replaced by the parameters passed in when the user calls the command
C) $ARGUMENTS defines the output format of the command
D) $ARGUMENTS is a deprecated feature

> **Answer: B**
> Explanation: $ARGUMENTS is a placeholder in the command file. When the user calls via /command-name arg1 arg2, arg1 arg2 will replace $ARGUMENTS so that the command supports parameterization.

**Q15.** What are the typical steps to create a custom slash command?

A) Declare the command name in CLAUDE.md, and then create the corresponding .md file in .claude/commands/
B) Define directly in the command panel of Claude Code without creating a file
C) Register in the project's commands.json and then create the implementation file
D) Just define it in ~/.claude/CLAUDE.md and the command will be automatically generated

> **Answer: A**
> Explanation: The standard process for creating a custom command is to declare the existence of the command in CLAUDE.md or project configuration, and then create a Markdown file with the corresponding name in the .claude/commands/ directory. The content of the file is the specific instructions of the command.

**Q16.** Suppose you need to create a command `/test-suite` which should build the project before running all tests. How should this command be implemented?

A) Include a sequence of commands in .claude/commands/test-suite.md: first build, then run the test, passing in the test arguments using $ARGUMENTS
B) Create two independent commands /build and /test, and the user executes them manually
C) Define automation rules in CLAUDE.md to execute this process
D) Integrate this feature by modifying the project's build script

> **Answer: A**
> Explanation: .claude/commands/test-suite.md should contain complete command definitions describing compilation and testing steps. $ARGUMENTS can pass in specific test filtering conditions to implement parameterized command calls.

**Q17.** What is the fundamental difference between slash commands and context configuration in CLAUDE.md?

A) Both are the same, just have different names
B) The slash command is a declarative configuration, CLAUDE.md is an imperative action instruction
C) CLAUDE.md is a declarative context configuration, and the slash command is an imperative executable workflow.
D) The slash command can only be used globally, CLAUDE.md is project-level

> **Answer: C**
> Explanation: This is the key conceptual difference. CLAUDE.md declaratively describes the contract and context of a project, while slash commands are imperative, immediately executable workflows or tasks.

**Q18.** A team wants to standardize the code review process and all developers should follow the same steps to perform reviews. How should this be achieved through a custom slash command?

A) Write out the review steps in CLAUDE.md in detail and let each developer perform them manually
B) Create .claude/commands/code-review.md, which contains a standard review checklist and steps that teams can quickly invoke via /code-review
C) Ask team members to configure personal global ~/.claude/commands/
D) Fully automated review using CI/CD process, no slash commands required

> **Answer: B**
> Explanation: This is a typical usage scenario of the slash command. By defining a standard review process in .claude/commands/code-review.md, team members can execute it consistently, ensuring review quality and standardization of the process.

**Q19.** How to share custom slash commands between teams via git?

A) Submit the .claude/commands/ directory to the git repository, which can be used by team members after cloning
B) Slash commands cannot be shared via git and must be created manually by each developer
C) Exchange command files via email
D) Upload to a central command library service that each developer subscribes to use

> **Answer: A**
> Explanation: The .claude/commands/ directory is stored within the project and can be completely controlled by git version. Team members will automatically obtain these commands after cloning the project, which improves collaboration efficiency.

**Q20.** Is there any special convention for naming the created slash commands? What naming strategy should be chosen?

A) No special agreement, you can use any name
B) should be written in all caps for easy identification
C) Clear, descriptive lowercase names should be used, with hyphens separating words (e.g. `/run-tests`, `/check-lint`) to avoid conflicts with system commands
D) Must be prefixed with the project name

> **Answer: C**
> Explanation: Good naming conventions improve usability. It is recommended to use clear lowercase names separated by hyphens (kebab-case) to ensure the meaning of the command is clear and avoid conflicts with Claude Code's built-in commands.

**Q21.** A certain slash command needs to receive multiple parameters and perform different operations. For example `/deploy env=prod region=us-east` . How should these parameters be handled in the command file?

A) Claude Code's $ARGUMENTS does not support named parameters, positional parameters must be used
B) Use $ARGUMENTS in the command file, and then prompt the user to pass in the parameters in a specific format, or explain the parameter format in the command description
C) Create a parameter configuration file to handle complex parameter passing
D) A command can only accept one parameter

> **Answer: B**
> Explanation: Although $ARGUMENTS is a universal parameter passing mechanism, the command file can clearly state the format and meaning of the parameters in the description, so that users know how to correctly pass in the parameters. Complex parameter handling can be achieved through clear documentation.

**Q22.** In a multi-project working environment, a certain slash command should be reused in multiple projects. What are the best practices?

A) Copy a command file in each project to .claude/commands/
B) Create this command in the user global ~/.claude/commands/, which can be accessed by all projects
C) Choose according to the specific situation: general commands can be placed globally, and project-specific commands can be placed in the project's .claude/commands/
D) Create a shared command library that all projects reference via URL

> **Answer: C**
> Explanation: This is the most flexible solution. Common, cross-project commands can be placed in the user's global ~/.claude/commands/, and project-specific commands are stored in .claude/commands/ within the project. The two can coexist, and Claude Code will load commands from both locations.

---

## Task 3.3 glob rule file configuration (Q23–Q32)

**Q23.** What are the rule files in the .claude/rules/ directory mainly used for?

A) CI/CD rules for storing projects
B) Store rules with YAML frontmatter and glob patterns to apply targeted configuration to specific filesets
C) Store database migration rules
D) Store user access control rules

> **Answer: B**
> Explanation: The rule files in .claude/rules/ use YAML frontmatter to define the glob pattern so that the rules can specifically act on files matching the pattern. This provides finer-grained control than the global CLAUDE.md.

**Q24.** What is the standard format for YAML frontmatter in rules files?

A) `[globs]\nsrc/**/*.ts\n[/globs]`
B)
```yaml
---
globs: ["src/**/*.ts", "lib/**/*.ts"]
---
```
C) `// globs: src/**/*.ts`
D) `{globs: ["src/**/*.ts"]}`

> **Answer: B**
> Explanation: The rule file uses the standard YAML frontmatter format, surrounded by three hyphens ---, and globs is an array containing one or more glob patterns.

**Q25.** In a project, different coding conventions need to be applied for TypeScript source files (src/) and test files (tests/). How should I use glob rules?

A) Create two rule files: rules/typescript-src.md (globs: ["src/**/*.ts"]) and rules/typescript-tests.md (globs: ["tests/**/*.ts"]) to define different specifications respectively.
B) Use two different glob patterns in one rules file, but this will cause conflicts
C) Create a common rule that applies to all TypeScript files
D) Use conditional statements in CLAUDE.md to differentiate

> **Answer: A**
> Explanation: This is the standard usage of glob rules. By creating multiple rule files, each targeting different file modes, different specifications can be applied to different areas of code, providing flexible configuration.

**Q26.** What is the difference between glob patterns `src/**/*.ts` and `src/*/*.ts`?

A) Both are exactly the same
B) The former matches .ts files in src and all its subdirectories, while the latter only matches .ts files in direct subdirectories of src
C) The former is not supported and is an old syntax.
D) The latter is more efficient and should be used first

> **Answer: B**
> Explanation: `**` is a recursive wildcard, matching directories of any depth; `*` only matches a single directory level. `src/**/*.ts` matches src/a/b/c/file.ts, while `src/*/*.ts` only matches src/a/file.ts.

**Q27.** Suppose there is a rule file that defines globs: ["src/**/*.ts"], but there are two files src/utils/helper.ts and src/utils/helper.d.ts. How can I make the rule only apply to .ts files and not .d.ts files?

A) Use negation pattern ["src/**/*.ts", "!src/**/*.d.ts"] in glob
B) Specific files cannot be excluded in the rules file, separate rules must be created
C) Add an excludeGlobs field in frontmatter
D) This is not possible, glob rules cannot do fine-grained exclusions

> **Answer: A**
> Explanation: The glob pattern supports negative syntax. Add a ! sign before the pattern that needs to be excluded. This allows precise control over which files should be included or excluded.

**Q28.** What happens when the glob patterns of multiple rule files overlap (for example, they all match src/app/main.ts)?

A) Only the first matching rule is applied
B) All matching rules are applied, which may cause conflicts
C) Claude Code will throw an error and ask for conflict resolution
D) The last matching rule overrides other rules

> **Answer: B**
> Explanation: All matching rules will be applied. If there are conflicts between rules, they need to be handled explicitly in the rules file or by optimizing the glob pattern to avoid overlap.

**Q29.** In what scenarios should I choose to use the glob rules in .claude/rules/ instead of defining global rules in CLAUDE.md?

A) Global CLAUDE.md should always be used, .claude/rules/ is redundant
B) When the rule only needs to be applied to specific files or directories of the project, using glob rules can provide precise scope control.
C) Only used when global rules cannot be executed
D) .claude/rules/ is deprecated

> **Answer: B**
> Explanation: This is the complementary relationship between the two solutions. The global CLAUDE.md provides common project-wide configuration, while .claude/rules/ provides targeted rules for specific sets of files via glob patterns.

**Q30.** A project has multiple rules files that define specifications for TypeScript files. What should be done if there are priority requirements between these rules (one rule should take precedence over others)?

A) Add a numerical prefix to the file name to control the loading order, such as 01-strict-rules.md, 02-standard-rules.md
B) Priority is not supported between rule files and only one unified rule file can be created.
C) Explicitly specify priority using specific YAML fields in the rules file
D) Implicitly control priority through file modification time

> **Answer: A**
> Explanation: In order to control the loading order and priority of multiple rule files, file naming conventions (such as numeric prefixes) are usually used. This ensures that more restrictive or higher priority rule files are loaded later, overwriting earlier rules.

**Q31.** In addition to the globs field, what other information can the YAML frontmatter contain in the rules file?

A) Can only contain globs fields
B) Can contain metadata such as description, priority, author, etc., as well as specific content configuration fields of the rules
C) Can contain arbitrary YAML data, but only globs are recognized by Claude Code
D) No other fields can be added, otherwise it will cause parsing errors

> **Answer: B**
> Explanation: YAML frontmatter can contain multiple fields. In addition to globs, you can also add metadata such as description, priority, author, etc., as well as specific configuration of rules. This information helps document and manage rules.

**Q32.** Assume that there is a .claude/rules/framework-specific.md rules file in the project, and its glob pattern is ["src/react/**/*.tsx"]. At the same time, common TypeScript coding specifications are defined in the project-level CLAUDE.md. How will these two configurations work together?

A) Framework-specific rules fully cover the general specifications
B) Only general specifications are applied, globs rules are ignored
C) For files matching src/react/**/*.tsx, framework-specific rules extend and possibly override the general specifications in CLAUDE.md; for files that do not match, only the general specifications are applied
D) Two configurations conflict, causing loading to fail.

> **Answer: C**
> Explanation: This is the correct way for rules and CLAUDE.md to work together. CLAUDE.md provides a global foundation, and the glob rules in .claude/rules/ extend or cover specific file sets at a finer granularity. The two form a hierarchical configuration system.

---

## Task 3.4 Claude Code CLI flags and modes (Q33–Q46)

**Q33.** What is the main function of Claude Code's `-p` or `--print` flag?

A) Print the directory structure of the project
B) Enable non-interactive mode. Claude Code will exit immediately after executing a task without waiting for further user input.
C) Print a list of all available commands
D) Output a summary of Claude's response

> **Answer: B**
> Explanation: The -p/--print flag switches Claude Code to non-interactive mode, suitable for CI/CD processes or scripting scenarios. Claude will perform the specified task, output the results, and then exit without entering an interactive session.

**Q34.** In the CI/CD process, it is necessary to obtain the results of Claude Code execution, especially the details of tool calls. What flag should be used?

A) `--verbose` flag
B) `--output-format json` flag, returns structured JSON output, including tool calls, parameters and results
C) `--debug` flag
D) `--api-response` flag

> **Answer: B**
> Explanation: The --output-format json flag causes Claude Code to output in structured JSON format, including tool call chains, parameters and execution results, for easy programmatic processing.

**Q35.** Suppose an application needs to automatically generate an API response, and the response must conform to a specific JSON schema. How should I use Claude Code to ensure the correctness of the output format?

A) Use the --json-schema flag to provide the desired schema and Claude Code will ensure that the output conforms to that schema
B) Manually require a specific format in the prompt
C) Use the --output-format json flag (this is only about output packaging, not content validation)
D) This is impossible, Claude Code cannot guarantee schema compliance

> **Answer: A**
> Explanation: The --json-schema flag allows specifying a JSON schema, and Claude Code will generate output based on the schema to ensure the correctness of the data structure and format. This is important when you need to produce output that conforms to a specific data structure.

**Q36.** How does Claude Code's plan mode work?

A) Planning mode is an automation mode, Claude will perform all operations directly without user confirmation
B) In plan mode, Claude will first generate an execution plan, display it to the user, and then perform specific operations after obtaining user confirmation.
C) Scheduled mode for backup and recovery
D) Planning mode has been deprecated

> **Answer: B**
> Explanation: Planning mode is an interactive security mechanism. Claude first analyzes the task and generates a detailed execution plan. After the user reviews and confirms it, Claude executes the plan. This prevents unexpected or harmful operation.

**Q37.** How to enable planning mode in Claude Code and require user confirmation after generating the plan?

A) Use the `--plan` flag to start Claude Code, Claude will enter plan mode
B) Configure `plan_mode: true` in CLAUDE.md
C) Select planning mode via Claude Code’s interactive menu
D) Planning mode is automatically enabled and no special configuration is required

> **Answer: A**
> Explanation: The --plan flag enables planning mode. Claude generates and presents an action plan to the user, who can review, approve, or reject the plan before executing it.

**Q38.** What is the purpose of the `--resume [session-id]` flag? In what scenarios is it useful?

A) Pause the current session and save the state
B) Resume a previously interrupted Claude Code session, allowing continuation of the previous conversation and task context
C) Restart a new session
D) Delete previous session history

> **Answer: B**
> Explanation: The --resume flag is used to restore a previously saved session. This is useful in long-running tasks so that if the session is interrupted, work can be continued from a saved checkpoint without starting from scratch.

**Q39.** In the CI/CD process, the output of Claude Code is used for logging and subsequent processing. How should the execution of Claude Code be configured to facilitate integration?

A) Use the `-p` flag to enter non-interactive mode, use `--output-format json` to get structured output, and may also need `--json-schema` to validate the output
B) Run Claude Code directly, no special configuration is required
C) Use the `--verbose` flag to log all details
D) Configure a log file path

> **Answer: A**
> Explanation: This is a combination of best practices for CI/CD integration. Non-interactive mode ensures automatic execution, JSON output facilitates programmatic processing, and JSON schema validation ensures output quality.

**Q40.** What is the purpose of the `--allowedTools` and `--disallowedTools` flags in the Claude Code command line?

A) For installation and uninstallation of configuration tools
B) Limit the set of tools that Claude can use at execution time, providing security controls and workflow restrictions
C) Version used for management tools
D) These flags are deprecated

> **Answer: B**
> Explanation: These two flags provide tool-level access control. --allowedTools specifies that Claude can only use the listed tools, --disallowedTools excludes specific tools. This is useful in security-sensitive environments or when specific functionality needs to be restricted.

**Q41.** Suppose an automation script uses Claude Code to generate code. If the task fails, how should the script determine the cause of the failure through the Claude Code's exit code?

A) Claude Code does not use exit codes, users need to parse the output to determine
B) Claude Code uses the standard Unix exit code: 0 means success, non-zero value means error, the specific error type can be judged by the error message of --output-format json
C) All errors return the same exit code
D) The exit code is random and cannot be used to determine the status.

> **Answer: B**
> Explanation: Claude Code follows the Unix convention and uses exit codes to indicate execution status. The script can check the exit code, combined with detailed error information in the JSON output, to determine the specific cause of the failure.

**Q42.** In a task that requires unattended execution, how to prevent the confirmation step of the plan mode from blocking the automation execution when using Claude Code?

A) Schedule mode cannot be used in an unattended environment
B) Use the `-p` flag to enter non-interactive mode, which skips plan confirmation, or predefine tasks in scripts to avoid complex planning requirements
C) Set a timer to automatically confirm the plan
D) Unavoidable and must be confirmed manually

> **Answer: B**
> Explanation: Non-interactive mode (-p flag) bypasses interactive confirmation but still uses plan output to guide work. For completely unattended execution, the task definition should be carefully designed to have minimal dependence on the confirmation step.

**Q43.** Suppose a project uses Claude Code to perform multiple analysis tasks. In order to improve efficiency and facilitate debugging, how should the execution of Claude Code be configured?

A) Run multiple instances of Claude Code, each handling a task
B) Use the `-p` flag to quickly execute a single task and `--output-format json` for easy parsing and integration of results
C) Perform all tasks within an interactive session
D) Use a background process and ignore output

> **Answer: B**
> Explanation: For batch tasks, using the -p flag for non-interactive mode and the JSON output format can improve efficiency. Each task is completed quickly and structured results are output for easy integration and debugging.

**Q44.** When using the Claude Code CLI, how should sensitive output information (such as API keys or database credentials) be handled, especially in CI/CD logs?

A) Direct output, relying on CI/CD system to mask
B) Automatically mask sensitive information through the security configuration of CLAUDE.md, or perform log filtering in scripts
C) Not using the CLI to handle sensitive operations
D) Hardcoding sensitive information in scripts

> **Answer: B**
> Explanation: Security best practices require configuring Claude in CLAUDE.md to understand the processing specifications of sensitive information. At the same time, the script side should have a log filtering or masking mechanism to prevent sensitive data from leaking into CI/CD logs.

**Q45.** In what scenarios is the headless mode of Claude Code particularly useful?

A) Execute Claude Code in a server or container environment without a monitor for automated workflows
B) Only used during debugging
C) This mode is deprecated
D) For unit testing only

> **Answer: A**
> Explanation: The headless mode allows Claude Code to run in an environment without UI, which is especially suitable for servers, containers and CI/CD processes. This is key to enabling automated workflows and integrations.

**Q46.** A project needs to run Claude Code in GitHub Actions to automate code analysis and generation tasks. How should Claude Code's CLI flag combinations be configured?

A) `claude code -p --output-format json --allowedTools analysis,generation` and other necessary tool restrictions
B) Run `claude code` directly, no special flags required
C) Use `--verbose --debug` to get detailed information
D) Claude Code cannot be used in GitHub Actions

> **Answer: A**
> Explanation: This is a best practice for using Claude Code in CI/CD. -p enables non-interactive mode, --output-format json gets structured output for easy processing, --allowedTools limits available tools to ensure safety and control.

---

## Task 3.5 Skill design and configuration (Q47–Q60)

**Q47.** In Claude Code, what is the storage location and file structure of skills?

A) Skills are stored in .claude/skills/[name]/SKILL.md. SKILL.md contains the definition and configuration of skills.
B) Skills are stored in the skills section of CLAUDE.md
C) Skills are stored in the global ~/.claude/skills/ directory
D) Skills do not need to be stored and are defined directly in memory

> **Answer: A**
> Explanation: Each skill corresponds to a directory .claude/skills/[skill-name]/, where SKILL.md is the core file. This structure facilitates version control and modular management of skills.

**Q48.** What key fields should the YAML frontmatter in the SKILL.md file contain?

A) Contains only name and description
B) context, allowed-tools, argument-hint, etc. These fields define the execution environment, permissions and usage of skills.
C) Only allowed-tools is required
D) YAML frontmatter is optional in SKILL.md

> **Answer: B**
> Explanation: The YAML frontmatter of SKILL.md contains several key configuration fields, specifically context (execution environment), allowed-tools (permission restrictions) and argument-hint (usage hints), which define how the skill is run and used.

**Q49.** What is the difference between `context: fork` and `context: inherit` in skills?

A) Both are the same, just have different names
B) `context: fork` creates an isolated execution environment, and skills run in an independent context; `context: inherit` inherits the context of the parent
C) fork is used to create new files, inherit is used to read files
D) These two fields have been deprecated

> **Answer: B**
> Explanation: This is the core concept of skill isolation. The fork mode creates an independent, isolated context for the skill, which is not affected by the parent; the inherit mode inherits all the context, variables and configuration of the parent.

**Q50.** Suppose a skill needs to perform some sensitive operation (such as deleting a file), but should be limited to the use of a specific tool. How should it be configured in frontmatter of SKILL.md?

A) This is impossible, skills cannot limit tool usage
B) Use `allowed-tools: ["file_delete", "file_read"]` in frontmatter to only allow the use of the listed tools
C) List allowed tools in the Markdown content of the skill
D) Use the `disallowed-tools` field to exclude unwanted tools

> **Answer: B**
> Explanation: The allowed-tools field precisely defines the set of tools that the skill can call. This is a critical security mechanism, especially when skills need to perform sensitive operations.

**Q51.** What is the `argument-hint` field and what role does it play in skills?

A) Prompt user for password or security credentials
B) Provide instructions and tips for users using skills on how to pass in parameters
C) A deprecated field
D) for configuring error handling

> **Answer: B**
> Explanation: argument-hint provides usage hints, telling the user how to call the skill and what parameters should be passed in. This increases the usability and understandability of skills.

**Q52.** What is the essential difference between skill and slash command?

A) They are exactly the same, just have different names
B) Slash commands are simple executable workflows, while skills are more complex components that are configurable, with isolated execution environments, tool permission restrictions, and parameter prompts.
C) Skills have been replaced by slash commands
D) Slash commands are more powerful than skills

> **Answer: B**
> Explanation: This is the key difference. Skills are a higher-level abstraction that supports context isolation (fork/inherit), fine-grained tool permission control and structured parameter prompts, and is suitable for complex workflows that require controlled execution.

**Q53.** How to call a skill in Claude Code? What mode should be used?

A) Command via slash /[skill-name]
B) Through the Skill tool, pass in the skill name and parameters, Claude Code will load and execute the skill
C) Directly referenced in CLAUDE.md
D) Through the @import directive

> **Answer: B**
> Explanation: Call skills using the Skill tool, which is a standard Claude Code tool calling mode. The Skill tool is responsible for loading the skill's SKILL.md, parsing the configuration, and setting the execution environment and permission restrictions.

**Q54.** Suppose a project has two skills, each of which needs to access different sensitive resources (skill A needs to access the database, skill B needs to access the file system). How should the permissions of these two skills be configured?

A) Both skills should have full permissions
B) Configure allowed-tools respectively in SKILL.md: Skill A allows ["database-read", "database-write"], Skill B allows ["file-read", "file-write"]
C) Create a unified permission profile
D) Different permissions cannot be set for different skills

> **Answer: B**
> Explanation: This is the best practice for skill permission management. Each skill clearly declares the required tool permissions based on its functional requirements, following the principle of least privilege to improve security.

**Q55.** A skill needs to clean up the environment before execution and verify it after execution. How should this skill be designed?

A) Describe the sanitization and validation steps in the Markdown content of the skill and have Claude execute them when called
B) Create two separate skills to handle cleanup and main tasks respectively
C) Use context: fork to create an isolated environment, and clearly describe the steps of initialization, execution and cleanup in the description of the skill
D) Use external scripts to handle environmental protection and validation

> **Answer: C**
> Explanation: Using fork context can isolate the environment and prevent state pollution. Clearly describe the stages in the skills document and have Claude step through them to ensure proper initialization and cleanup.

**Q56.** If a skill's context is set to fork, how will modifications to the file system be handled during skill execution?

A) Modifications are permanently saved to the file system
B) Modifications are only made in an isolated context. After the execution is completed, it will be decided whether to commit or rollback based on the configuration of the skill.
C) All modifications are discarded
D) This depends on the specific file system implementation

> **Answer: B**
> Explanation: The fork context creates an isolated execution environment. The modification is performed in the sandbox, and finally it is decided whether to apply it to the actual file system based on the execution results and configuration policies. This provides transactional execution guarantees.

**Q57.** Suppose a skill needs to receive complex parameters (such as a JSON object). How should argument-hint be configured in SKILL.md?

A) argument-hint cannot support complex parameters
B) Provide clear instructions in argument-hint to explain the format, required fields and example values of the parameters to help users correctly pass in parameters
C) Create a separate parameter configuration file
D) Parameters must be simple types

> **Answer: B**
> Explanation: argument-hint is a text field that can provide detailed usage instructions. For complex parameters, the structure of the parameter, required fields, data types, and specific examples should be included in the prompt.

**Q58.** How to share and version control skills through git in a team?

A) Skills cannot be shared via git, each developer must create them independently
B) Submit the entire .claude/skills/ directory to git. Team members can access all skills after cloning the project
C) Create a central skills library that teams reference via URL or package manager
D) Skills must be stored in the global directory and cannot be shared at the project level

> **Answer: B**
> Explanation: .claude/skills/ is stored within the project and fully supports git version control. Teams can work together to maintain and evolve skills, ensuring consistency and auditability within the team.

**Q59.** Suppose a skill encounters an error during execution (such as a tool call failure). How should this error be handled in context: fork mode?

A) Error will automatically roll back all modifications
B) The skill should describe the error handling strategy in the Markdown description, allowing Claude to clean up and report appropriately when an error is encountered
C) The error is ignored and execution continues
D) Unable to handle errors in fork mode

> **Answer: B**
> Explanation: While forking provides isolation, error handling needs to be explicit in the design of the skill. Skills should describe recovery strategies under different failure scenarios, allowing Claude to make the right decisions.

**Q60.** What best practices should be followed when designing a reusable, high-quality skill?

A) Just need to implement the function, no special structure or documentation is required
B) Clearly define context (fork or inherit), minimize allowed-tools permissions, provide detailed argument-hint, write clear Markdown documents, contain error handling and validation logic, and design for sharing and version control
C) Skills should use all available tools as much as possible
D) There is no best practice in skill design

> **Answer: B**
> Explanation: This is a complete list of best practices for skill design. Good skills should be secure (least privileges), understandable (clear documentation), maintainable (easy to test and debug), and shareable (support version control).

---

## Answer Guide

**Score evaluation criteria:**
- 55-60 points: Excellent (master all core concepts of Domain 3)
- 48-54 points: Good (mastered most of the core content, some concepts need to be strengthened)
- 40-47 points: Pass (master the basic concepts, but need improvement in advanced applications and integration)
- Score below 40: Need to review relevant content and retake the exam

**Key areas for review (if the correct answer rate is less than 70%):**
- If the accuracy of Q1 — Q12 is less than 70%: focus on reviewing the CLAUDE.md three-level system and inheritance mechanism
- If the accuracy of Q13 — Q22 is less than 70%: focus on reviewing the creation and usage scenarios of the slash command
- If the accuracy of Q23 — Q32 is less than 70%: focus on reviewing the glob rule file and YAML frontmatter configuration
- If the accuracy of Q33 — Q46 is less than 70%: focus on reviewing the use and combination of Claude Code CLI flags
- If the accuracy of Q47 — Q60 is less than 70%: focus on reviewing the context, permissions and configuration mechanism of skills