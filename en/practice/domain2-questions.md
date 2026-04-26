# CCA-F Simulation Questions — Domain 2: Tool Design and MCP Integration

> There are 60 questions total, and the recommended completion time is 75 minutes. single-choice questions.

---

## Task 2.1 Tool interface design (Q1–Q12)

**Q1.** You are designing a set of tools for a data analysis system. Users need to read CSV files, filter rows, aggregate data, and generate charts. Which of the following tool design options best adheres to the single responsibility principle?

A) A tool called `analyze_data` that receives the CSV path and all processing instructions (filtering, aggregation, charting) and returns the final result
B) Four independent tools: `read_csv`, `filter_rows`, `aggregate_data`, `generate_chart`, each tool has clear responsibilities
C) A general tool `process`, specifying whether to read, filter, aggregate or draw through the `operation` parameter
D) Two tools: `data_io` (read and write) and `data_processing` (all calculations and visualizations)

> **Answer: B**
> Explanation: The single responsibility principle requires each tool to focus on one responsibility. The four tools of option B perform their own duties and are easy to reuse, test and maintain. Option A has too many responsibilities, option C simulates multiple tools through parameters (difficult to maintain), and option D is still not detailed enough.

**Q2.** The parameters of an API tool include: `user_id` (required), `include_archived` (optional, boolean value, default false), `max_results` (optional, integer, range 1–100, default 20), `sort_by` (optional, enumeration value: "name", "date", "relevance"). Which of the following tools is most clearly and accurately described?

A) Obtain user data, optional parameters include include_archived, max_results, sort_by
B) Get the data records of the specified user. include_archived (Boolean, default false) controls whether archived items are included; max_results (1–100, default 20) controls the number of returns; sort_by ("name"|"date"|"relevance", default "name") controls sorting
C) Get user data, the parameters are user_id, include_archived, max_results, sort_by
D) Retrieve data based on user ID and various filter conditions, supporting archive items, result restrictions and sorting

> **Answer: B**
> Explanation: Option B explicitly lists each parameter's type, allowed values, default value, and meaning. Option A is too brief, option C lacks type information, and option D "various filter conditions" is unclear. Clear parameter description is at the core of tool design.

**Q3.** A file system tool needs to support delete operations. Which of the following tool names and descriptions best conveys its side effects and risks?

A) `delete_file` Description: Delete file
B) `remove_file` Description: Delete the specified file from the file system (this operation is irreversible, please use with caution)
C) `fs_delete` Description: File deletion operation
D) `clean_file` Description: Clean file

> **Answer: B**
> Explanation: The tool name `remove_file` is clear and direct; the description emphasizing "irreversible" and "please use with caution" can effectively warn users. Option A lacks warnings, option C has an over-abbreviated name, and option D "cleans up" is ambiguous and does not warn of risks.

**Q4.** You designed a `search_documents` tool that allows users to search by title, content, tags, and date range. Currently this tool only accepts a `query` string parameter. How should it scale as demand grows?

A) Added five new parameters: `query_title`, `query_content`, `query_tags`, `query_date_min`, `query_date_max`
B) Keep the `query` parameter, the user specifies the search field through the "title:foo content:bar" syntax
C) Design the `filters` object parameters, including title, content, tags, date_range sub-parameters, for more flexible use
D) Create separate tools `search_by_title`, `search_by_content`, `search_by_tags`, `search_by_date`

> **Answer: C**
> Explanation: Option C uses structured object parameters, which keeps the interface simple (a `filters` parameter) and provides flexible search options. Option A bloats parameters, option B relies on complex string parsing, and option D creates too many tools. Structured parameters are a best practice for extending interfaces.

**Q5.** A certain database query tool has two required parameters: `table_name` (table name) and `where_clause` (WHERE condition string). Which of the following designs is best at preventing SQL injection attacks and keeping interfaces clear?

A) Maintain the status quo and the user enters the WHERE condition string
B) Change `where_clause` to `conditions` object, supporting `{"column": "name", "operator": "=", "value": "John"}` structure
C) Add parameter verification: check whether `where_clause` contains dangerous keywords (such as DROP, DELETE)
D) Add `allow_unsafe_sql` boolean parameter, user explicitly enables raw SQL

> **Answer: B**
> Explanation: Option B uses a structured conditions object, which enables the underlying system to use parameterized queries and avoids SQL injection at all. Option A is still dangerous, Option C has incomplete blacklist filtering, and Option D is clear but does not address the security concerns. Tool interface design should prevent unsafe operations at the source.

**Q6.** An email sending tool needs to support multiple recipients, CC, and BCC. Which of the following parameter designs is the clearest and easiest to extend?

A) `to`, `cc`, `bcc` are all comma-separated strings: "user1@example.com,user2@example.com"
B) `to`, `cc`, `bcc` are all string arrays: ["user1@example.com", "user2@example.com"]
C) A single `recipients` parameter, including role annotation: "to:user1@example.com;cc:user2@example.com"
D) Three parameters, each an optional single string value

> **Answer: B**
> Explanation: Arrays of strings (option B) are both clear (types are clearly multiple addresses) and easy to manipulate and verify in programming languages. Option A requires parsing commas, option C requires complex string parsing, and option D does not support multiple recipients. Explicit data structures are the cornerstone of tool interfaces.

**Q7.** You design the `publish_article` tool for a content management system. Parameters include `article_id`, `publish_date` (optional, ISO 8601 format), `notify_subscribers` (optional, boolean). Which of the following descriptions most accurately defines the tool's boundaries and behavior?

A) Publish an article, you can choose the publishing time and whether to notify subscribers
B) Publish articles based on article_id. publish_date (ISO 8601 format, such as "2026-04-11T14:30:00Z") specifies the publishing time; if not provided, it will be published immediately. notify_subscribers (boolean, default true) controls whether notification emails are sent to subscribers. If publishing fails, an error message will be returned
C) Mark the article as published, update the publication date, and optionally send a notification
D) Execute the article publishing process, handling dates, notifications and other necessary steps

> **Answer: B**
> Explanation: Option B clearly defines the format, default value, and behavior of each parameter, and also explains how it is handled by default. Option A is too general, option C "update release date" is inaccurate, and option D "other necessary steps" is vague. A good tool description should make the user's expectations clear.

**Q8.** A certain file conversion tool supports input formats PDF, Word, and PowerPoint, and output formats PDF, HTML, and Markdown. How should tool parameters be designed?

A) Single `format` parameter, user specifies "PDF_to_HTML" or "Word_to_Markdown"
B) Two parameters `input_format` and `output_format`, each defined with a list of enumeration values
C) Single `conversion_type` parameter, predefined all supported conversion combinations (9 types in total)
D) A general `convert` parameter, the user can pass in any combination of formats

> **Answer: B**
> Explanation: Option B separates concerns, `input_format` and `output_format` are enumerated independently, which is clear and easy to maintain (if new formats are added later, just extend the enumeration). Option A requires enumeration of all combinations, option C limits flexibility, and option D lacks validation. Clear parameter design facilitates expansion.

**Q9.** A report generation tool needs to receive data source information. Users might use SQL query strings, ready-made dataset IDs, or uploaded CSV file paths. Which of the following designs best handles this flexibility?

A) Single `data_source` parameter, user specified "sql:SELECT...", "dataset:ID", or "file:/path/to.csv"
B) Three parameters `sql_query`, `dataset_id`, `csv_file_path`, the user fills in one of them
C) `data_source_type` (enum: "sql"|"dataset"|"file") and `data_source_value` (corresponding value)
D) A single flexible `source` parameter that accepts any format string

> **Answer: C**
> Explanation: Option C uses the separation of types and values to both clarify user intent (via `data_source_type`) and clearly receive the corresponding data. Option A requires string prefix parsing, option B requires "exactly one argument" logic to be enforced at the tool level, and option D is too vague. Structured design is the best way to handle multiple inputs.

**Q10.** The `aggregate` method of a certain data analysis tool supports multiple aggregate functions (SUM, COUNT, AVG, MAX, MIN). Users need to perform different aggregations on multiple fields. How should parameters be designed?

A) `aggregation_functions` parameter is a string: "SUM(price),COUNT(*),AVG(rating)"
B) `aggregations` parameter is an object array: [{"function": "SUM", "field": "price"}, {"function": "COUNT", "field": null}, ...]
C) Create separate parameters for each function: `sum_fields`, `count_fields`, `avg_fields`, etc.
D) `functions` array and `fields` array, paired in sequence: "functions:["SUM","COUNT"]" and "fields:["price","quantity"]"

> **Answer: B**
> Explanation: Option B represents each aggregation operation as a structured object, which is both clear (function and field pairing) and easy to verify and extend. Option A requires complex parsing, option C has bloated parameters, and option D relies on positional relationships and is error-prone. Arrays of objects are a standard way of representing multiple complex operations.

**Q11.** You are designing an interface for a machine learning model inference tool. Models can accept a variety of inputs: image URL, image file path, text, audio. The following design best distinguishes these input types clearly and prevents confusion?

A) Single `input` parameter, automatically detecting type based on content
B) `input_type` ("image_url"|"image_file"|"text"|"audio") and `input_value` (corresponding data)
C) Four optional parameters `image_url`, `image_file`, `text`, `audio`, the user fills in one of them
D) `input_data` object, including type field and data field

> **Answer: B**
> Explanation: Option B explicitly separates types and values so that tools can correctly handle input based on type and avoid ambiguities in auto-detection. Although option C is also feasible, the parameters are more scattered; option B is more compact. Option D is similar to B but with one more level of nesting. Option B provides the best balance of clarity and simplicity.

**Q12.** The `grant_permission` method of a certain permission management tool needs to specify the granted permissions (read, write, delete, manage), and whether to apply recursively to sub-resources. Which of the following designs best prevents inadvertent granting of excessive permissions?

A) The `permissions` parameter is a list of strings: ["read", "write", "delete", "admin"], `recursive` is a Boolean value
B) The `permission_level` parameter is a single-selection enumeration: 1 (read-only), 2 (read-write), 3 (read-write delete), 4 (full management), `recursive` is a Boolean value
C) Design `can_read`, `can_write`, `can_delete`, `can_admin` respectively, each of which is a Boolean value, and `recursive` is a Boolean value
D) `permissions` is an object: {"read": true, "write": false, "delete": false, "admin": false}, `recursive` is a Boolean value

> **Answer: D**
> Explanation: The object format of option D explicitly lists all permissions and their status to prevent omissions. Option A allows any combination but lacks a default value, Option B has a fixed and inflexible hierarchy (users cannot choose read-only + delete), and Option C has too many parameters. The object design is both clear and reflects complete permission mapping.

---

## Task 2.2 MCP structured error response (Q13–Q24)

**Q13.** In which of the following ways should MCP tool results report errors so that Claude can understand and decide whether to retry?

A) A JavaScript/Python exception is thrown, caught by the MCP framework and converted into an error message
B) Return an object containing the `isError` field as true, with structured error information (such as failure_type, isRetryable)
C) Return error code and error message string, logged by Claude
D) Include the "ERROR:" prefix in the content field of the tool results

> **Answer: B**
> Explanation: The MCP standard stipulates the use of `isError: true` plus a structured error object, allowing Claude to understand the context of the error type, whether it can be retried, etc. Option A results in abnormal termination rather than graceful error handling, and options C and D lack structured information to support intelligent decision-making.

**Q14.** A certain database query tool found that the table did not exist when executing the query. Which of the following error responses best aligns with MCP best practices and will help Claude understand the problem and take appropriate action?

A) `{ isError: true, message: "Table not found" }`
B) `{ isError: true, failure_type: "validation_error", message: "Table 'users_backup' does not exist", alternatives: ["users", "users_archive", "users_temp"] }`
C) `{ isError: true, error_code: 404, message: "Table not found", retryable: false }`
D) `{ success: false, message: "The specified table could not be located in the database" }`

> **Answer: B**
> Explanation: Option B provides a complete structured error message: failure_type describes the error category (validation error), specific error message, and possible alternative table names. This way Claude can understand if he should try again, or advise the user to choose the correct table. Option A is too simplistic, and options C and D lack the `isError` standard field or include non-standard fields.

**Q15.** In the error response of the MCP tool, what are the scenarios when the `isRetryable` field should be set to true?

A) All errors because the user may want to try again
B) Only if the error is caused by temporary network outage or temporary unavailability of service
C) When the internal logic of the tool fails (such as parameter validation failure)
D) When the error comes from an external API

> **Answer: B**
> Explanation: `isRetryable: true` should only be used for temporary errors (such as network timeouts, temporary service outages, rate limits) that may succeed after conditions improve. Validation errors (C option) and certain API errors (such as authorization failures) should not be marked retryable.

**Q16.** An API call tool received a 429 (Too Many Requests) response when requesting. What error structure should be returned so that Claude understands that this is a temporary throttling and should try again later?

A) `{ isError: true, message: "Rate limit exceeded" }`
B) `{ isError: true, failure_type: "rate_limit", message: "Rate limit exceeded. Retry after 60 seconds", isRetryable: true, retry_after_seconds: 60 }`
C) `{ error: "TooManyRequests", status: 429, isRetryable: true }`
D) `{ isError: true, error_type: "transient", message: "Rate limited" }`

> **Answer: B**
> Explanation: Option B explicitly specifies failure_type (rate_limit), isRetryable (true), and retry_after_seconds, allowing Claude to understand the current limiting situation and retry after an appropriate time. Option A lacks complete information, option C does not comply with MCP standard fields, and option D lacks retry delay information.

**Q17.** A file reading tool tried to open a file that does not exist. Which of the following error responses most accurately reflects the nature of the problem?

A) `{ isError: true, failure_type: "file_not_found", message: "File '/data/missing.csv' not found" }`
B) `{ isError: true, failure_type: "validation_error", message: "The specified file path does not point to an existing file", attempted_file: "/data/missing.csv" }`
C) `{ isError: true, message: "Cannot read file", details: "File does not exist" }`
D) `{ isError: true, failure_type: "not_found", message: "File not found", isRetryable: false }`

> **Answer: B**
> Explanation: Option B uses a `validation_error` failure_type (input validation failure), including the attempted file path, clearly indicating that this is an input issue and not a system failure. Although option A is concise, "file_not_found" is not a standard failure_type, and option D's "not_found" has ambiguous meaning (maybe the business data does not exist). validation_error is standard practice for handling invalid input.

**Q18.** A network request tool timed out when calling a remote API. What error type should be returned and why?

A) failure_type: "timeout", isRetryable: true (this is a temporary network problem)
B) failure_type: "transient_error", isRetryable: true (transient error)
C) failure_type: "service_error", isRetryable: false (external service problem)
D) failure_type: "network_error", isRetryable: true (network failure)

> **Answer: A**
> Explanation: failure_type should clearly specify the actual error type (timeout), rather than a general classification (transient_error), so that Claude can distinguish different causes. isRetryable: true because timeouts are usually temporary and can be retried. Choice B is too general and the classifications of C and D are not precise enough.

**Q19.** A tool successfully processed the request but returned partial data (some fields were missing). How should the error response be structured to alert Claude that the data is incomplete while allowing partial results to be used?

A) Throw an exception and abort the operation until all data is available
B) Return `{ isError: false, content: [...], warning: "Some fields missing" }`
C) Return `{ isError: true, failure_type: "partial_failure", content: [...], message: "Some fields could not be retrieved" }`
D) Continue to return a complete successful response and record the missing situation in the log

> **Answer: C**
> Explanation: Using `isError: true` plus `failure_type: "partial_failure"`, and returning the available partial data in the content field, can allow Claude to understand that this is neither a complete success nor a complete failure, and requires special processing. Option A is too strict, option B is not clear enough with warning, and option D hides the problem. Partial failure is a common scenario and should be structured accordingly.

**Q20.** A user's permission check failed (the user does not have permission to access resources). What failure_type should be returned?

A) "permission_denied"
B) "authorization_error"
C) "access_denied"
D) "security_error"

> **Answer: A**
> Explanation: "permission_denied" in the MCP standard is the standard failure_type of permission error. Although options B, C, and D have similar semantics, they are not standard usage. Use the standard failure_type to ensure that Claude correctly understands and handles various authorization failure scenarios.

**Q21.** When a tool returns an error, what should be included in the `message` field to best help Claude understand the problem?

A) Technical details and stack trace information for easy debugging
B) User-friendly error description that clearly explains what went wrong and possible causes
C) Just write "error" or "failure" and let Claude infer based on failure_type
D) Include the complete SQL query or request content so you can reproduce the issue

> **Answer: B**
> Explanation: The message should be a clear description that both the user and Claude can understand ("User 'alice@example.com' does not have permission to delete this file"). Option A's stack trace is not useful to Claude, option C is too brief, and option D may contain sensitive information. Clear error messages allow Claude to make better decisions.

**Q22.** A tool tried to connect to the database but failed. Which of the following error responses most completely provides the context that Claude might need?

A) `{ isError: true, message: "Database connection failed" }`
B) `{ isError: true, failure_type: "connection_error", message: "Failed to connect to PostgreSQL at postgres.example.com:5432", attempted_query: null, alternatives: ["Use different host", "Check network connectivity"], isRetryable: true }`
C) `{ isError: true, failure_type: "service_error", message: "Database unavailable", isRetryable: true, error_details: { code: "ECONNREFUSED", host: "postgres.example.com" } }`
D) `{ isError: true, message: "Cannot reach database server", retry: true }`

> **Answer: B**
> Explanation: Option B contains the failure_type, a clear error description (even the specific host and port), possible alternatives, and the isRetryable flag. Option C lacks alternatives, and options A and D lack information. The complete error message helps Claude understand the severity of the problem and possible solutions.

**Q23.** When should the `partial_results` field be included in an MCP error response?

A) Every error response should contain, even if empty
B) Only when the operation is partially successful (for example, some items in the batch operation failed)
C) When the error is not retryable
D) When isError is false

> **Answer: B**
> Explanation: `partial_results` is used to represent partial failure scenarios, such as batch deletion of 10 files but only 8 of them successful. Option A adds unnecessary redundancy, and options C and D are both incorrect. partial_results is the key field that helps Claude understand "how much work has been completed".

**Q24.** Should a tool documentation clearly state when it returns `isError: true` and when it returns `isError: false`?

A) It only needs to be clear at the tool implementation level, no need to explain it in the document
B) Include a list of possible failure_types and their respective trigger conditions in the tool description, such as "permission_denied: when the user does not have permission to perform the operation"
C) Include the [may fail] tag in the tool name
D) Users should base their judgment on whether there is an error message in the return value

> **Answer: B**
> Explanation: Tool documentation should clearly enumerate all possible failure_types and their triggering conditions, so Claude can understand when the tool will fail and how to handle various errors. Option A ignores the importance of documentation, option C is impractical, and option D can easily lead to misunderstandings. Good documentation is the basis for correct error handling.

---

## Task 2.3 Tool allocation and tool_choice configuration (Q25–Q36)

**Q25.** In Claude Agent SDK, what is the purpose of the `allowedTools` array?

A) Specify which tools can be used in the agent to implement the principle of least privilege
B) Sort the list of tools by priority, and Claude will use the top tools first.
C) Define tools that are mandatory in this agent
D) Assign usage quotas to tools and limit the number of calls for each tool

> **Answer: A**
> Explanation: allowedTools is a security control mechanism. Only tools in the list can be used by the agent. This reflects the "principle of least privilege" - assigning agents only the tools they need to complete their responsibilities. Option B is a tool selection strategy rather than the purpose of allowedTools, option C should be implemented using tool_choice, and option D is not a function of allowedTools.

**Q26.** You are designing tool permissions for a content moderation agent. The agent needs to read user submissions and flag violations, but should not be able to delete or modify user data. What should be included in `allowedTools`?

A) `["read_content", "flag_violation"]`
B) `["read_content", "flag_violation", "delete_content", "modify_content"]` (Although it will not be used, it is kept as a spare)
C) `["read_content", "flag_violation", "send_notification"]`
D) All available tools, then state in the agent prompt that the data should not be deleted

> **Answer: A**
> Explanation: allowedTools should only contain tools actually needed by the agent. Option A is required for auditing functionality. Option B contains unnecessary and dangerous tools (although it will not be used, it exposes the attack surface), and option D relies on prompt words rather than configuration to protect data, which is insufficient security. Configuration-level constraints are more reliable than prompt words.

**Q27.** What is the difference between `tool_choice: "auto"` and `tool_choice: "any"`?

A) "auto" and "any" are exactly the same, both give Claude the freedom to choose whether to use the tool or not.
B) "auto" allows Claude to automatically determine whether a tool needs to be used; "any" requires Claude to use a tool in allowedTools
C) "auto" limits the number of tool uses; "any" allows unlimited tool calls
D) "any" is only used in multi-tool scenarios, "auto" is used in single-tool scenarios

> **Answer: B**
> Explanation: The key difference is constraint. "auto" leaves Claude free to decide whether to use the tool (possibly not calling it at all); "any" forces Claude to select a tool in each response. Use "auto" for question-and-answer tasks, and "any" for tasks that require tool calls. Neither option C nor D is accurate.

**Q28.** An agent needs to choose one of three operations to perform: send an email, send a text message, or create a task reminder. How should `tool_choice` be configured?

A) `tool_choice: "auto"`, let Claude decide whether and which tool to choose
B) `tool_choice: "any"`, let Claude choose one of the tools
C) `tool_choice: { "type": "tool", "name": "send_email" }`, force email execution
D) `tool_choice: { "type": "tool", "names": ["send_email", "send_sms", "create_reminder"] }`, let Claude choose one from them

> **Answer: B**
> Explanation: This is a must-select scenario (the user needs some kind of notification), so use "any" to make sure Claude selects a tool. Option A's "auto" may cause Claude to not call any tools; option C's forced selection of emails is inflexible; option D's syntax is incorrect (no "names" field).

**Q29.** In the agent definition, which of the following configurations best reflects the "principle of least privilege" while ensuring that the agent can complete its work?

A) agent_1: { allowedTools: ["read_file", "write_file", "execute_bash", "delete_file"] }
B) agent_1: { allowedTools: ["read_file"] }, when writing is required, call agent_2: { allowedTools: ["write_file"] }
C) agent_1: { allowedTools: ["read_file", "write_file"] }, tool_choice: "auto"
D) All tools are shared by all agents and are called by the agent at its own discretion.

> **Answer: C**
> Explanation: Option C configures the agent with two tools required to complete basic file operations, excluding unnecessary dangerous operations (execute_bash, delete_file). Option A has too many permissions, option B excessively fragments and increases complexity, and option D violates the principle of least privilege. Striking the right balance between security and functionality is key.

**Q30.** A system has two agents: agent_finance (processing financial data) and agent_public (generating public reports). How should tool permissions be assigned to prevent financial data breaches?

A) Both agents have access to all tools, relying on prompt words and censorship to prevent abuse
B) The allowedTools of agent_finance only include financial database tools; the allowedTools of agent_public only include report templates and aggregate statistics tools, and do not include raw data access.
C) agent_finance can access all tools, agent_public can only access reporting tools
D) Use unified allowedTools, but limit the tool selection of agent_public in tool_choice

> **Answer: B**
> Explanation: Option B is the application of the principle of least privilege. agent_public does not have access to raw financial data tools at all and can only use the aggregated statistics interface. Option A relies on prompt words that are unsafe, the agent_finance permissions in option C are unnecessarily large, and option D through tool_choice restrictions are not as reliable as through allowedTools restrictions.

**Q31.** A certain workflow requires the use of a specific tool to complete a certain step. What configuration should be used?

A) `tool_choice: { "type": "tool", "name": "specific_tool_name" }`
B) `allowedTools: ["specific_tool_name"]` plus `tool_choice: "auto"`
C) Explicit instructions on using the tool in the agent prompt
D) Create a subagent that contains only the tool

> **Answer: A**
> Explanation: Option A uses the specific tool specification method of tool_choice to force Claude to call the specified tool. Option B limits available tools but "auto" may still result in no calls, option C relies on hints that are not reliable enough, and option D is over-engineered. The tool_choice field is specifically designed to force tool selection.

**Q32.** Which of the following configuration combinations best prevents an untrusted agent from abusing sensitive tools?

A) allowedTools: [all tools], tool_choice: "auto", relying on the guidance of agent prompt words
B) allowedTools: [] (do not allocate any tools), so that the agent can only be delegated to other agents through agent.call
C) allowedTools: [minimum tools required by the business], tool_choice: "auto", regularly audit agent behavior
D) allowedTools: [minimum tools required by the business], tool_choice: "any"

> **Answer: C**
> Explanation: Option C combines the principle of least privilege (minimization of allowedTools), moderate freedom ("auto") and post-event monitoring (auditing), and is a practical security solution. Option A is too open, option B is too restrictive, and the "any" requirement in option D may not necessarily require forced tool calls. Secure engineering practices require a combination of configuration and monitoring.

**Q33.** In the subagent architecture, how should the parent agent control the tool permissions of the subagent?

A) Define permissions in allowedTools of subagent, which cannot be overridden by parent agent
B) Dynamically specify the subagent's tool list in the parent agent (if supported)
C) Define the permissions in the allowedTools of the subagent and confirm it in the configuration of the parent agent, and take the intersection of the two.
D) Completely controlled by parent agent, subagent does not need to define allowedTools

> **Answer: A**
> Explanation: The subagent's permissions should be specified in its own definition (allowedTools), which ensures that the permissions are configurable and auditable and do not depend on the correct configuration of the parent agent. Option B provides flexibility but is difficult to audit, option C adds complexity, and option D pushes permission control to the parent agent and is error-prone. The principle of least privilege should be enforced at each agent level.

**Q34.** A certain task requires the agent to call the tool. If the call fails, an error will be returned to the user (without retrying). What tool_choice strategy should be used?

A) `tool_choice: "auto"` plus max_retries: 0
B) `tool_choice: "any"` plus max_retries: 0
C) `tool_choice: "auto"`, let Claude decide whether to call and how to handle errors
D) Customize tool_choice and clearly specify the tool to be called

> **Answer: B**
> Explanation: The B option ensures three things: 1) "any" forces the tool to be called (to complete the task), 2) max_retries: 0 means no retries on error, 3) Claude will return an error to the user when the tool fails. Option A of "auto" may result in the tool not being called at all, option C may cause no call, and option D is possible but not as concise as "any".

**Q35.** In a multi-tool scenario, how should `allowedTools` and `tool_choice` be used differently to manage tool access?

A) allowedTools and tool_choice have the same functions, just choose one
B) allowedTools defines the security boundary (which tools are available), tool_choice defines the selection strategy (whether to force use, how to choose)
C) allowedTools is used in multi-tool scenarios, tool_choice is used in single-tool scenarios
D) allowedTools is set during configuration, tool_choice is set dynamically at runtime

> **Answer: B**
> Explanation: These are configurations in two different dimensions. allowedTools answers "what tools can be used" (security), tool_choice answers "how to choose to use" (strategy). Option A is incorrect, and options C and D are both inaccurate. Understanding the difference between the two is key to correctly designing agent permissions.

**Q36.** An agent needs to call a tool occasionally (not every time), but when called it must be one of the three specified tools. How should it be configured?

A) `allowedTools: ["tool_a", "tool_b", "tool_c"]`, `tool_choice: "auto"`
B) `allowedTools: ["tool_a", "tool_b", "tool_c"]`, `tool_choice: "any"`
C) `tool_choice: [{ "type": "tool", "name": "tool_a" }, { "type": "tool", "name": "tool_b" }, { "type": "tool", "name": "tool_c" }]`
D) `allowedTools: ["tool_a", "tool_b", "tool_c"]`, `tool_choice: { "type": "tool", "names": ["tool_a", "tool_b", "tool_c"] }`

> **Answer: A**
> Explanation: Option A not only limits the scope of available tools (allowedTools), but also allows Claude to freely decide whether to call it (tool_choice: "auto"). Option B forces a tool to be called every time, options C and D have incorrect syntax. The ideal configuration is one that maintains security boundaries (allowedTools) while providing freedom of choice ("auto").

---## Task 2.4 MCP server integration configuration (Q37–Q48)

**Q37.** What are the main differences between the two configuration files `.mcp.json` and `~/.claude.json`?

A) The functions are exactly the same, but the location is different
B) `.mcp.json` is in the project root directory and is used for project shared configuration (including git); `~/.claude.json` is in the user home directory and is used for personal configuration (including personal credentials)
C) `.mcp.json` is used for development environment, `~/.claude.json` is used for production environment
D) `.mcp.json` is managed by Claude, `~/.claude.json` is managed by users

> **Answer: B**
> Explanation: This is the key difference. .mcp.json should be in version control, shared with the team, and contain public configuration; ~/.claude.json should be local only, and contain personal API keys and sensitive credentials. Option C confuses the environment with file locations, and option D is inaccurate.

**Q38.** An MCP server needs to read the key from the environment variable `OPENAI_API_KEY`. How should it be configured in `.mcp.json` to achieve environment variable expansion?

A) `"env": { "API_KEY": "OPENAI_API_KEY" }`
B) `"apiKey": "$OPENAI_API_KEY"`
C) `"apiKey": "${OPENAI_API_KEY}"`
D) `"apiKey": "process.env.OPENAI_API_KEY"`

> **Answer: C**
> Explanation: MCP standard uses `${ENV_VAR}` syntax to expand environment variables. Option C is correct. Option A is a configuration map rather than an expansion, Option B is missing curly braces, and Option D is that JavaScript syntax is not applicable. Understanding the ${} expansion syntax is fundamental to configuring MCP.

**Q39.** An MCP server is configured as follows:
```json
{
  "mcpServers": {
    "myserver": {
      "command": "node",
      "args": ["./server.js"],
      "env": {
        "API_KEY": "${MYSERVER_API_KEY}",
        "DEBUG": "true"
      }
    }
  }
}
```
What happens if the environment variable `MYSERVER_API_KEY` is not set?

A) MCP will use the default value "MYSERVER_API_KEY"
B) MCP will throw an error and refuse to start the server
C) The environment variable remains as is "${MYSERVER_API_KEY}" (unexpanded) and is processed by the server
D) MCP will prompt the user to set environment variables

> **Answer: C**
> Explanation: If the environment variable does not exist, ${} expansion usually remains unchanged (depending on the implementation) or expands to an empty string. In most cases, whether an environment variable is required or not should be determined by the server itself, not the MCP framework. Option B is too restrictive, and neither A nor D conforms to usual behavior.

**Q40.** A certain project's `.mcp.json` and the user's `~/.claude.json` both define the `myserver` server, but the configurations are different. Which one will take effect?

A) The two configurations will be merged, and the conflicting part will be taken precedence by `.mcp.json`
B) The two configurations will be merged, and the conflicting part will be taken precedence by `~/.claude.json`
C) Only `.mcp.json` takes effect, `~/.claude.json` is ignored
D) Only `~/.claude.json` takes effect, `.mcp.json` is ignored

> **Answer: B**
> Explanation: User configuration (~/.claude.json) usually takes higher priority, allowing users to override project configuration locally. This is important for adding personal credentials or adjusting parameters. The priority of option A is reversed, and options C and D are too absolute. Prioritized design allows projects and personal configurations to work together.

**Q41.** A certain MCP server uses stdio transmission method. How should it be specified in the configuration?

A) `"transport": "stdio"`
B) `"type": "stdio"`
C) `"protocol": "stdio"`
D) Configure `"command"` and `"args"`, stdio is the implicit default

> **Answer: D**
> Explanation: stdio is the most common transmission method. When the configuration contains `command` and `args`, stdio is used by default. No need to specify the "stdio" keyword explicitly. The field names for options A, B, and C are all incorrect. Understanding implicit defaults simplifies configuration.

**Q42.** A certain MCP server needs to be connected using the SSE (Server-Sent Events) transmission method. How should the configuration be written?

A) `"transport": "sse", "url": "https://mcp.example.com/sse"`
B) `"type": "sse", "endpoint": "https://mcp.example.com/sse"`
C) `"command": "sse", "args": ["https://mcp.example.com/sse"]`
D) `"protocol": "sse", "connection": "https://mcp.example.com/sse"`

> **Answer: A**
> Resolution: SSE transport requires specifying `transport: "sse"` and URL. Option A has field names that are closest to the standard. The field names of options B, C, and D are all incorrect. SSE is a transport for remote connections and is different from stdio (local command).

**Q43.** A certain MCP server configuration requires multiple parameters to be passed on startup:
```
python my_server.py --port 5000 --debug --config /etc/config.yaml
```
How should it be configured in `.mcp.json`?

A) `"command": "python my_server.py --port 5000 --debug --config /etc/config.yaml"`
B) `"command": "python", "args": ["my_server.py", "--port", "5000", "--debug", "--config", "/etc/config.yaml"]`
C) `"command": "python my_server.py", "arguments": "--port 5000 --debug --config /etc/config.yaml"`
D) `"script": "python my_server.py --port 5000 --debug --config /etc/config.yaml"`

> **Answer: B**
> Explanation: The `args` array breaks down command line arguments into individual elements so that MCP can parse and process them correctly. The A option contains arguments in a single string (requires shell parsing), and the C and D options have incorrect field names. An explicit args array is easier for the program to handle.

**Q44.** When adding personal API credentials for an MCP server in `~/.claude.json`, what format should be used to reference the same server defined in project `.mcp.json`?

A) Completely redefine mcpServers and all parameters in `~/.claude.json`
B) Define only the fields you want to override (like API credentials) in `~/.claude.json`, using the same server name
C) Create a new server name, such as `"myserver_personal"`
D) Reserve placeholders in the project `.mcp.json`, `~/.claude.json` only needs to provide the value

> **Answer: B**
> Explanation: MCP configuration supports merging, users can write only the part to be overwritten (usually credentials) in `~/.claude.json`. Option B is the most efficient and avoids duplication. Option A results in high maintenance costs, and options C and D are both impractical. Configuration merging is a key feature of MCP design.

**Q45.** What command line flag should be used to specify a custom MCP profile when starting the Claude Code CLI?

A) `--mcp-config /path/to/config.json`
B) `--config-file /path/to/config.json`
C) `-c /path/to/config.json`
D) `--mcp-servers-config /path/to/config.json`

> **Answer: A**
> Explanation: `--mcp-config` is a standard Claude Code flag used to specify the MCP configuration file path. The other options look reasonable but are not actual CLI interfaces. Learn the correct CLI flags to avoid configuration errors.

**Q46.** An MCP server needs to access the project's private key file (located in the project directory). How should the environment variables and relative paths be configured?

A) `"env": { "PRIVATE_KEY_PATH": "/absolute/path/to/key.pem" }` directly use the absolute path
B) `"env": { "PRIVATE_KEY_PATH": "./keys/key.pem" }` Use relative paths (relative to the project root directory)
C) `"env": { "PRIVATE_KEY_PATH": "${PROJECT_ROOT}/keys/key.pem" }` if MCP supports PROJECT_ROOT variable
D) Hardcoding the path in the server startup script

> **Answer: C**
> Explanation: Using variables (such as `${PROJECT_ROOT}`) is the most flexible and allows configuration to be ported between different machines. Option B may be unstable depending on the relative path of the working directory, option A lacks flexibility, and option D is hardcoded and unmaintainable. Environment variable expansion is standard practice across machine configurations.

**Q47.** An MCP server is defined in the team's `.mcp.json`, but a team member's local development environment uses a different configuration (ports, credentials). How should this be done so that project configurations can be shared while supporting individual differences?

A) Each developer maintains his or her own copy of `.mcp.json`, which is not included in git
B) Project `.mcp.json` defines shared configuration (such as server commands), and individual developers override personal parameters (such as ports, credentials) in `~/.claude.json`
C) Create `.mcp.json.template` and fill it in manually after the developer copies it
D) Create branch configuration for each developer in the project `.mcp.json`

> **Answer: B**
> Explanation: Option B is best practice. Project configuration (commands, args, etc.) is shared in `.mcp.json`, personal credentials and parameters are in `~/.claude.json` (not included in git). This maintains project unity while supporting individual differences. Option A results in fragmented configuration, option C requires manual synchronization, and option D is too complex.

**Q48.** The API key used by an MCP server needs to be updated due to periodic rotation. Where should this key be stored and why?

A) Stored in the project `.mcp.json` to facilitate team synchronization
B) Stored in user `~/.claude.json` as a personal secret
C) Stored in operating system environment variables (such as `MYSERVER_API_KEY`), `~/.claude.json` refers to it
D) Stored in the `.env` file in the project root directory and automatically loaded by the tool

> **Answer: C**
> Resolution: API keys are sensitive credentials and should be stored in operating system-level environment variables, not files. `~/.claude.json` can reference these environment variables (${MYSERVER_API_KEY}), which is safe and easy to rotate. Option A exposes credentials, option B is limited to a single machine, and option D's .env file is also usually not under version control but is not as secure as environment variables. Environment variables are standard practice for credential management.

---

## Task 2.5 Selection and use of built-in tools (Q49–Q60)

**Q49.** Which built-in tool should be used in the following scenario: You need to read a 1000-line text file, but only care about the first 100 lines.

A) Use the `Read` tool, without specifying offset/limit, to read the entire file and process it in code
B) Use the `Read` tool, specify `offset: 0, limit: 100`, and only read the required part
C) Use the `Bash` command `head -n 100 file.txt`
D) Use the `Grep` tool to search the first 100 lines of content

> **Answer: B**
> Explanation: The `Read` tool supports offset and limit parameters, allowing efficient reading of specific parts of the file without wasting bandwidth and memory. Option A reads unnecessary content, option C also works but is not as direct as the Read tool, and option D does not match the purpose. Understanding the tool’s optimization parameters can improve efficiency.

**Q50.** A specific string in the file needs to be modified (such as replacing "old_value" with "new_value"). Which tool should I use?

A) Use `Read` to read the entire file, then use `Write` to re-write the modified content
B) Use the `Edit` tool to specify old_string: "old_value", new_string: "new_value"
C) Use the `Bash` command `sed -i 's/old_value/new_value/g' file.txt`
D) Use the `Write` tool to directly overwrite the file

> **Answer: B**
> Explanation: The `Edit` tool is specially designed for in-place modification. It automatically replaces old_string and new_string after specifying them, which is more concise than option A (read, modify, and write). Although option C is also possible, it requires bash execution, and option D will overwrite the entire file. Edit is a targeted modification tool.

**Q51.** Need to search for all lines containing "TODO" in a large code base. Which tool should I use?

A) Use the `Read` tool to read all files
B) Use the `Bash` command `grep -r "TODO" .`
C) Use the `Grep` tool, specify pattern: "TODO"
D) Use the `Glob` tool to find all files and read them one by one

> **Answer: C**
> Explanation: The `Grep` tool is based on ripgrep, specially designed for pattern search, and supports recursive search and multiple output formats. Option C is the most direct and effective. Option A is inefficient, option B works but is not as integrated as the Grep tool, and option D requires multiple steps. Grep is the tool of choice for searching.

**Q52.** Need to find all TypeScript files in the project. Which tool should I use?

A) Use the `Bash` command `find . -name "*.ts"`
B) Use the `Glob` tool and specify pattern: "**/*.ts"
C) Use the `Grep` tool to search for file extensions
D) Use the `Read` tool to traverse the directory structure

> **Answer: B**
> Explanation: The `Glob` tool is specially designed for file pattern matching, and "**/*.ts" is a standard glob pattern. Option B is the most direct. Option A works but requires bash, option C is not suitable (Grep searches for content rather than filenames), and option D does not support directory traversal. Glob is a specialized tool for file search.

**Q53.** Requires executing a system command such as `npm install` or `python script.py`. Which tool should I use?

A) Use the `Bash` tool to execute commands
B) Use the `Write` tool to create a script file and then run
C) Use the `Read` tool to check if the script exists
D) Use the `Grep` tool to find the command

> **Answer: A**
> Explanation: The `Bash` tool is designed for executing shell commands. Option A is straightforward and efficient. Option B is unnecessarily complex and options C and D have mismatched uses. Bash is the standard tool for executing system commands.

**Q54.** Need to get JSON data from the network (such as API response). Which tool should I use?

A) Use the `Bash` command `curl https://api.example.com/data`
B) Use the `WebFetch` tool to specify the URL
C) Use the `Read` tool to read the URL directly
D) Use the `Grep` tool to search web content

> **Answer: B**
> Explanation: The `WebFetch` tool is designed for HTTP requests and returns structured responses. Option B is the most suitable. Option A works but requires bash, Option C Read does not support URLs (only supports local files), and Option D has mismatched uses. WebFetch is the standard tool for network requests.

**Q55.** You need to search the project for all locations where a certain function is called, such as `processUser()`. Which combination of tools should be used to be most efficient?

A) Use `Read` to read all files and search manually
B) Use the `Bash` command `grep -r "processUser(" .` plus `--include="*.js"`
C) Use `Glob` to find all .js files, then use `Grep` to search for function calls
D) Use the `Grep` tool, pattern: "processUser\(" and glob filter: "*.js"

> **Answer: D**
> Explanation: The `Grep` tool supports the glob filter parameter, which can complete file filtering and content search in one tool call. Option D is the most efficient and concise. Option A is inefficient, option B works but uses mixed tools, and option C is not as concise as D because of its multiple steps. Understanding the tool's advanced parameters can simplify the task.

**Q56.** A new file of 500 lines needs to be written. How should I use the `Write` tool?

A) Write all 500 rows at once
B) Divide into multiple calls, each time up to 25-30 lines, use mode: "append"
C) First use `Read` to check whether the file exists, and then decide whether to write
D) Break into multiple small files, 50-100 lines each

> **Answer: B**
> Explanation: The best practice for the Write tool is to write in chunks (25-30 lines at a time) so that it does not result in extremely large requests. Use mode: "rewrite" for the first time, and mode: "append" for subsequent times. Option B complies with the tool usage guidelines. Option A may result in a request that is too large, option C is unnecessary, and option D is artificially complicated.

**Q57.** Multiple contents in a file need to be modified (such as replacing multiple strings). How should I use the `Edit` tool?

A) Call the `Edit` tool multiple times, replacing one place each time
B) Replace everything with an `Edit` call (if old_string contains all parts to be changed)
C) Use `Bash`’s sed command to replace in one go
D) Use `Read` to read the entire file, modify it manually, and then use `Write` to overwrite it.

> **Answer: A**
> Explanation: When multiple places need to be replaced and each place is different, `Edit` should be called multiple times, specifying the exact old_string and new_string each time. Option A is the safest (every step can be verified). Option B is ambiguous if old_string contains multiple parts to be replaced, Option C requires bash, and Option D is riskier (whole coverage). Multi-step modification is more reliable than one-step modification.

**Q58.** Need to search code that contains a specific regular expression, such as all uses of `console.log(`. Which tool and parameters should be used?

A) Use the `Grep` tool, pattern: "console\\.log\\(" (escape character)
B) Use the `Grep` tool, pattern: "console.log(", output_mode: "content"
C) Use the `Bash` command `grep -r "console.log(" .`
D) Use the `Grep` tool, pattern: "(console)(log)(\()"

> **Answer: A**
> Explanation: `Grep` supports regular expressions (based on ripgrep), special characters need to be escaped. Option A correctly escapes dots and brackets. Option B lacks escaping, option C is feasible with bash but not as integrated as the Grep tool, option D is regular but the syntax is not as concise as A. Master ripgrep syntax to search more efficiently.

**Q59.** Compare the following two methods to modify the configuration file: Method 1 uses the `Edit` tool to replace a single line; Method 2 uses `Read` to read the entire file, modify it, and then use `Write` to overwrite it. Which one is better and why?

A) Both are equivalent, the choice depends on personal preference
B) Method 1 (Edit) is better because it is modified in place and does not involve reading and writing the entire file, making it safer and more efficient.
C) Method 2 (Read+Write) is better because multiple modifications can be made at the same time
D) Method 1 is error-prone, method 2 is more reliable

> **Answer: B**
> Explanation: The `Edit` tool is specially designed for in-place modification. It only affects specific strings and is safer (other parts will not be accidentally modified). Although method 2 supports multiple modifications, it involves rewriting the entire file and is more error-prone. Option B is correct. Understanding tool characteristics allows you to choose the most appropriate method.

**Q60.** A task requires: 1) finding all log lines that contain a specific error, 2) extracting the user IDs in these lines, 3) saving the user IDs to a new file. Which combination of tools should be used to be most efficient?

A) Use `Bash` to complete all steps at once (grep + awk + > output)
B) Use `Grep` to find the log line, manually extract the ID, and use `Write` to save it
C) Use `Bash` to execute complex pipelines > Use `Read` to verify results > Use `Write` to save final results (over-engineering)
D) Use `Grep` to find the log line (output_mode: "content"), extract the ID in the code, and use `Write` to save it

> **Answer: D**
> Explanation: Option D is the best balance. The Grep tool specializes in searching logs, and output_mode controls the output format. Extracting IDs in code is cleaner, and Write saves the final result. Although option A is feasible, the shell pipeline is complex and error-prone, option B "manual extraction" is not automated, and option C has too many levels. This combination is both clear and efficient.

---