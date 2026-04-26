# CCA-F 模拟题 — Domain 2：工具设计与MCP集成

> 共 60 题，建议完成时间 75 分钟。每题单选。

---

## Task 2.1 工具接口设计（Q1–Q12）

**Q1.** 你正在为数据分析系统设计一套工具。用户需要读取 CSV 文件、过滤行、聚合数据、生成图表。以下哪种工具设计方案最符合单一责任原则？

A) 一个名为 `analyze_data` 的工具，接收 CSV 路径和所有处理指令（过滤、聚合、图表），返回最终结果
B) 四个独立工具：`read_csv`、`filter_rows`、`aggregate_data`、`generate_chart`，每个工具职责明确
C) 一个通用工具 `process`，通过 `operation` 参数指定是读、过滤、聚合还是绘图
D) 两个工具：`data_io`（读写）和 `data_processing`（所有计算和可视化）

> **答案：B**
> 解析：单一责任原则要求每个工具专注一个职责。选项 B 的四个工具各司其职，便于复用、测试和维护。A 选项职责过重，C 选项通过参数模拟多工具（难维护），D 选项划分仍不够细致。

**Q2.** 某 API 工具的参数包括：`user_id`（必需）、`include_archived`（可选，布尔值，默认 false）、`max_results`（可选，整数，范围 1–100，默认 20）、`sort_by`（可选，枚举值："name"、"date"、"relevance"）。以下工具描述最清晰准确？

A) 获取用户数据，可选参数包括 include_archived、max_results、sort_by
B) 获取指定用户的数据记录。include_archived（布尔，默认 false）控制是否包含已归档项；max_results（1–100，默认 20）控制返回数量；sort_by（"name"|"date"|"relevance"，默认"name"）控制排序
C) 获取用户数据，参数为 user_id、include_archived、max_results、sort_by
D) 根据用户 ID 和各种过滤条件检索数据，支持归档项、结果限制和排序

> **答案：B**
> 解析：选项 B 明确列出了每个参数的类型、允许值、默认值和含义。A 选项过于简略，C 选项缺少类型信息，D 选项"各种过滤条件"表述不清。清晰的参数描述是工具设计的核心。

**Q3.** 一个文件系统工具需要支持删除操作。以下哪个工具名和描述最能准确传达其副作用和风险性？

A) `delete_file` 描述：删除文件
B) `remove_file` 描述：从文件系统删除指定文件（此操作不可逆，请谨慎使用）
C) `fs_delete` 描述：文件删除操作
D) `clean_file` 描述：清理文件

> **答案：B**
> 解析：工具名 `remove_file` 清晰直接；描述中强调"不可逆"和"请谨慎使用"能有效警示使用者。A 选项缺少警告，C 选项名称过缩写，D 选项"清理"含义模糊且未警示风险。

**Q4.** 你设计了一个 `search_documents` 工具，用户可以按标题、内容、标签、日期范围搜索。目前这个工具只接收一个 `query` 字符串参数。随着需求增长，应该如何扩展？

A) 增加 `query_title`、`query_content`、`query_tags`、`query_date_min`、`query_date_max` 五个新参数
B) 保持 `query` 参数，用户通过 "title:foo content:bar" 语法指定搜索字段
C) 设计 `filters` 对象参数，包含 title、content、tags、date_range 子参数，使用更灵活
D) 创建单独的工具 `search_by_title`、`search_by_content`、`search_by_tags`、`search_by_date`

> **答案：C**
> 解析：选项 C 使用结构化对象参数，既保持接口简洁（一个 `filters` 参数），又提供了灵活的搜索选项。A 选项参数膨胀，B 选项依赖复杂字符串解析，D 选项创建过多工具。结构化参数是扩展接口的最佳实践。

**Q5.** 某数据库查询工具有两个必需参数：`table_name`（表名）和 `where_clause`（WHERE 条件字符串）。以下哪个设计最有利于防止 SQL 注入攻击并保持接口清晰？

A) 保持现状，用户输入 WHERE 条件字符串
B) 将 `where_clause` 改为 `conditions` 对象，支持 `{"column": "name", "operator": "=", "value": "John"}` 结构
C) 增加参数验证：检查 `where_clause` 中是否包含危险关键字（如 DROP、DELETE）
D) 添加 `allow_unsafe_sql` 布尔参数，用户显式启用原始 SQL

> **答案：B**
> 解析：选项 B 使用结构化 conditions 对象，能让底层系统使用参数化查询，根本避免 SQL 注入。A 选项仍然危险，C 选项的黑名单过滤不完整，D 选项虽然明确但不能解决安全问题。工具接口设计应该在源头防止不安全操作。

**Q6.** 一个邮件发送工具需要支持多个收件人和抄送、密抄。以下参数设计最清晰且易于扩展？

A) `to`、`cc`、`bcc` 都是逗号分隔的字符串："user1@example.com,user2@example.com"
B) `to`、`cc`、`bcc` 都是字符串数组：["user1@example.com", "user2@example.com"]
C) 单个 `recipients` 参数，包含角色标注："to:user1@example.com;cc:user2@example.com"
D) 三个参数，每个都是可选的单一字符串值

> **答案：B**
> 解析：字符串数组（选项 B）既清晰（类型明确为多个地址），又易于编程语言操作和验证。A 选项需要解析逗号，C 选项需要复杂的字符串解析，D 选项不支持多收件人。明确的数据结构是工具接口的基石。

**Q7.** 你为内容管理系统设计 `publish_article` 工具。参数包括 `article_id`、`publish_date`（可选，ISO 8601 格式）、`notify_subscribers`（可选，布尔值）。以下哪个描述最准确地定义了工具的边界和行为？

A) 发布文章，可选择发布时间和是否通知订阅者
B) 根据 article_id 发布文章。publish_date（ISO 8601 格式，如 "2026-04-11T14:30:00Z"）指定发布时间；若不提供，则立即发布。notify_subscribers（布尔，默认 true）控制是否发送通知邮件给订阅者。若发布失败，返回错误信息
C) 标记文章为已发布，更新发布日期，可选发送通知
D) 执行文章发布流程，处理日期、通知和其他必要步骤

> **答案：B**
> 解析：选项 B 清晰定义了每个参数的格式、默认值和行为，还说明了缺省情况下的处理方式。A 选项过于笼统，C 选项"更新发布日期"表述不准确，D 选项"其他必要步骤"模糊。优质的工具描述应该让使用者明确预期。

**Q8.** 某文件转换工具支持输入格式 PDF、Word、PowerPoint，输出格式 PDF、HTML、Markdown。应该如何设计工具参数？

A) 单个 `format` 参数，用户指定 "PDF_to_HTML" 或 "Word_to_Markdown"
B) 两个参数 `input_format` 和 `output_format`，各自用枚举值列表定义
C) 单个 `conversion_type` 参数，预定义所有支持的转换组合（共 9 种）
D) 一个通用 `convert` 参数，用户传入任意格式组合

> **答案：B**
> 解析：选项 B 分离关注点，`input_format` 和 `output_format` 各自独立枚举，既清晰又易于维护（如果后续添加新格式，只需扩展枚举）。A 选项需要枚举所有组合，C 选项限制灵活性，D 选项缺乏验证。明确的参数设计便于扩展。

**Q9.** 一个报表生成工具需要接收数据源信息。用户可能使用 SQL 查询字符串、现成的数据集 ID，或上传的 CSV 文件路径。以下设计最能处理这种灵活性？

A) 单个 `data_source` 参数，用户指定 "sql:SELECT...", "dataset:ID", 或 "file:/path/to.csv"
B) 三个参数 `sql_query`、`dataset_id`、`csv_file_path`，用户填充其中一个
C) `data_source_type`（枚举："sql"|"dataset"|"file"）和 `data_source_value`（对应值）
D) 单个灵活的 `source` 参数，接收任何格式的字符串

> **答案：C**
> 解析：选项 C 使用类型和值的分离，既明确了用户意图（通过 `data_source_type`），又清晰地接收相应数据。A 选项需要字符串前缀解析，B 选项需要在工具层面强制"恰好一个参数"的逻辑，D 选项过于模糊。结构化设计是处理多种输入的最佳方式。

**Q10.** 某数据分析工具的 `aggregate` 方法支持多种聚合函数（SUM、COUNT、AVG、MAX、MIN）。用户需要对多个字段执行不同的聚合。应该如何设计参数？

A) `aggregation_functions` 参数为字符串："SUM(price),COUNT(*),AVG(rating)"
B) `aggregations` 参数为对象数组：[{"function": "SUM", "field": "price"}, {"function": "COUNT", "field": null}, ...]
C) 为每个函数创建单独的参数：`sum_fields`、`count_fields`、`avg_fields` 等
D) `functions` 数组和 `fields` 数组，依次配对："functions:["SUM","COUNT"]" 和 "fields:["price","quantity"]"

> **答案：B**
> 解析：选项 B 将每个聚合操作表示为结构化对象，既清晰（function 和 field 配对），又易于验证和扩展。A 选项需要复杂解析，C 选项参数膨胀，D 选项依赖位置关系容易出错。对象数组是表示多个复杂操作的标准方式。

**Q11.** 你正在为机器学习模型推理工具设计接口。模型可以接收多种输入：图像 URL、图像文件路径、文本、音频。以下设计最能清晰区分这些输入类型且防止混淆？

A) 单个 `input` 参数，根据内容自动检测类型
B) `input_type`（"image_url"|"image_file"|"text"|"audio"）和 `input_value`（对应的数据）
C) 四个可选参数 `image_url`、`image_file`、`text`、`audio`，用户填充其中一个
D) `input_data` 对象，包含 type 字段和 data 字段

> **答案：B**
> 解析：选项 B 明确分离类型和值，工具可以根据类型正确处理输入，避免自动检测的歧义。C 选项虽然也可行，但参数较分散；B 选项更紧凑。D 选项与 B 类似但多一层嵌套。选项 B 提供了最佳的清晰度和简洁性平衡。

**Q12.** 某权限管理工具的 `grant_permission` 方法需要指定授予的权限（读、写、删除、管理），以及是否递归应用到子资源。以下哪个设计最能防止无意中授予过度权限？

A) `permissions` 参数为字符串列表：["read", "write", "delete", "admin"]，`recursive` 为布尔值
B) `permission_level` 参数为单选枚举：1（只读）、2（读写）、3（读写删除）、4（完全管理），`recursive` 为布尔值
C) 分别设计 `can_read`、`can_write`、`can_delete`、`can_admin`，各为布尔值，`recursive` 为布尔值
D) `permissions` 为对象：{"read": true, "write": false, "delete": false, "admin": false}，`recursive` 为布尔值

> **答案：D**
> 解析：选项 D 的对象格式明确列出所有权限及其状态，防止遗漏。A 选项允许任意组合但缺乏默认值，B 选项层级固定不灵活（用户无法选择只读+删除），C 选项参数过多。对象设计既清晰又能体现完整的权限映射。

---

## Task 2.2 MCP 结构化错误响应（Q13–Q24）

**Q13.** MCP 工具结果应该使用以下哪种方式报告错误，以便 Claude 能够理解并决定是否重试？

A) 抛出 JavaScript/Python 异常，由 MCP 框架捕获并转换为错误消息
B) 返回包含 `isError` 字段为 true 的对象，附带结构化的错误信息（如 failure_type、isRetryable）
C) 返回错误码和错误信息字符串，由 Claude 进行日志记录
D) 在工具结果的 content 字段中包含 "ERROR:" 前缀

> **答案：B**
> 解析：MCP 标准规定使用 `isError: true` 加上结构化错误对象，让 Claude 能够理解错误类型、是否可重试等上下文。A 选项会导致异常终止而非优雅错误处理，C 和 D 选项缺乏结构化信息，难以支持智能决策。

**Q14.** 某数据库查询工具在执行查询时发现表不存在。以下哪个错误响应最符合 MCP 最佳实践，能帮助 Claude 理解问题并采取适当行动？

A) `{ isError: true, message: "Table not found" }`
B) `{ isError: true, failure_type: "validation_error", message: "Table 'users_backup' does not exist", alternatives: ["users", "users_archive", "users_temp"] }`
C) `{ isError: true, error_code: 404, message: "Table not found", retryable: false }`
D) `{ success: false, message: "The specified table could not be located in the database" }`

> **答案：B**
> 解析：选项 B 提供了完整的结构化错误信息：failure_type 说明错误类别（验证错误），具体错误信息，以及可能的替代表名。这样 Claude 可以理解是否应重试，或建议用户选择正确的表。A 选项过于简略，C 和 D 选项缺乏 `isError` 标准字段或包含了非标字段。

**Q15.** MCP 工具的错误响应中，`isRetryable` 字段应该设为 true 的场景是什么？

A) 所有错误，因为用户可能想重新尝试
B) 仅当错误是由临时网络中断或服务暂时不可用引起时
C) 当工具内部逻辑失败时（如参数验证失败）
D) 当错误来自外部 API 时

> **答案：B**
> 解析：`isRetryable: true` 应该仅用于临时性错误（如网络超时、服务临时宕机、速率限制），这些错误在条件改善后可能成功。验证错误（C 选项）和某些 API 错误（如授权失败）不应标记为可重试。

**Q16.** 某 API 调用工具在请求时收到 429（Too Many Requests）响应。应该返回什么样的错误结构，以便 Claude 能够理解这是暂时限流且应该稍后重试？

A) `{ isError: true, message: "Rate limit exceeded" }`
B) `{ isError: true, failure_type: "rate_limit", message: "Rate limit exceeded. Retry after 60 seconds", isRetryable: true, retry_after_seconds: 60 }`
C) `{ error: "TooManyRequests", status: 429, isRetryable: true }`
D) `{ isError: true, error_type: "transient", message: "Rate limited" }`

> **答案：B**
> 解析：选项 B 明确指定了 failure_type（rate_limit），isRetryable（true），以及 retry_after_seconds，让 Claude 能够理解限流情况并在适当时间后重试。A 选项缺乏完整信息，C 选项不符合 MCP 标准字段，D 选项缺乏重试延迟信息。

**Q17.** 某文件读取工具尝试打开一个不存在的文件。以下哪个错误响应最准确地反映了问题的性质？

A) `{ isError: true, failure_type: "file_not_found", message: "File '/data/missing.csv' not found" }`
B) `{ isError: true, failure_type: "validation_error", message: "The specified file path does not point to an existing file", attempted_file: "/data/missing.csv" }`
C) `{ isError: true, message: "Cannot read file", details: "File does not exist" }`
D) `{ isError: true, failure_type: "not_found", message: "File not found", isRetryable: false }`

> **答案：B**
> 解析：选项 B 使用 `validation_error` failure_type（输入验证失败），包含尝试的文件路径，清楚地表明这是输入问题而非系统故障。A 选项虽然简洁但 "file_not_found" 不是标准 failure_type，D 选项的 "not_found" 含义模糊（可能是业务数据不存在）。validation_error 是处理无效输入的标准做法。

**Q18.** 某网络请求工具在调用远程 API 时发生超时。应该返回什么错误类型，为什么？

A) failure_type: "timeout"，isRetryable: true（这是临时网络问题）
B) failure_type: "transient_error"，isRetryable: true（暂时性错误）
C) failure_type: "service_error"，isRetryable: false（外部服务问题）
D) failure_type: "network_error"，isRetryable: true（网络故障）

> **答案：A**
> 解析：failure_type 应该明确指定实际的错误类型（timeout），而不是笼统的分类（transient_error），这样 Claude 能够区分不同原因。isRetryable: true 是因为超时通常是暂时现象，可以重试。选项 B 太笼统，C 和 D 的分类不够精确。

**Q19.** 某工具成功处理请求但返回了部分数据（某些字段缺失）。应该如何结构化错误响应，既提醒 Claude 数据不完整，又允许使用部分结果？

A) 抛出异常，中止操作，直到所有数据都可用
B) 返回 `{ isError: false, content: [...], warning: "Some fields missing" }`
C) 返回 `{ isError: true, failure_type: "partial_failure", content: [...], message: "Some fields could not be retrieved" }`
D) 继续返回完整成功响应，在日志中记录缺失情况

> **答案：C**
> 解析：使用 `isError: true` 加上 `failure_type: "partial_failure"`，同时在 content 字段返回可用的部分数据，能够让 Claude 理解这既不是完全成功也不是完全失败，需要特殊处理。A 选项过于严格，B 选项用 warning 不够明确，D 选项隐藏了问题。部分失败是常见场景，应该有相应的结构。

**Q20.** 某用户权限检查失败（用户无权访问资源）。应该返回什么 failure_type？

A) "permission_denied"
B) "authorization_error"
C) "access_denied"
D) "security_error"

> **答案：A**
> 解析：MCP 标准中 "permission_denied" 是权限错误的标准 failure_type。B、C、D 选项虽然语义相近但不是标准用法。使用标准 failure_type 确保 Claude 能够正确理解并处理各种授权失败场景。

**Q21.** 某工具返回错误时，应该在 `message` 字段中包含什么内容，才能最好地帮助 Claude 理解问题？

A) 技术细节和堆栈跟踪信息，便于调试
B) 用户友好的错误描述，清晰说明出了什么问题和可能的原因
C) 只写 "错误" 或 "失败"，让 Claude 根据 failure_type 推断
D) 包含完整的 SQL 查询或请求内容，以便重现问题

> **答案：B**
> 解析：message 应该是用户和 Claude 都能理解的清晰描述（"用户 'alice@example.com' 无权删除该文件"）。A 选项的堆栈跟踪对 Claude 无用，C 选项过于简略，D 选项可能包含敏感信息。清晰的错误消息让 Claude 能够做出更好的决策。

**Q22.** 某工具尝试连接数据库但失败。以下哪个错误响应最完整地提供了 Claude 可能需要的上下文？

A) `{ isError: true, message: "Database connection failed" }`
B) `{ isError: true, failure_type: "connection_error", message: "Failed to connect to PostgreSQL at postgres.example.com:5432", attempted_query: null, alternatives: ["Use different host", "Check network connectivity"], isRetryable: true }`
C) `{ isError: true, failure_type: "service_error", message: "Database unavailable", isRetryable: true, error_details: { code: "ECONNREFUSED", host: "postgres.example.com" } }`
D) `{ isError: true, message: "Cannot reach database server", retry: true }`

> **答案：B**
> 解析：选项 B 包含了 failure_type、清晰的错误描述（甚至包括具体主机和端口）、可能的替代方案以及 isRetryable 标志。C 选项缺乏 alternatives，A 和 D 选项信息不足。完整的错误信息帮助 Claude 理解问题的严重性和可能的解决方向。

**Q23.** 在 MCP 错误响应中，应该何时包含 `partial_results` 字段？

A) 每个错误响应都应该包含，即使为空
B) 仅当操作部分成功时（例如批量操作中某些项失败）
C) 当错误不可重试时
D) 当 isError 为 false 时

> **答案：B**
> 解析：`partial_results` 用于表示部分失败的场景，如批量删除 10 个文件但只有 8 个成功。A 选项会增加不必要的冗余，C 和 D 选项的条件都不对。partial_results 是帮助 Claude 理解"有多少工作已完成"的关键字段。

**Q24.** 某工具文档应该明确说明什么时候返回 `isError: true` 和什么时候返回 `isError: false`？

A) 仅在工具实现层面明确即可，无需在文档中说明
B) 在工具描述中列出可能的 failure_type 列表和各自的触发条件，例如 "permission_denied: 当用户无权执行操作时"
C) 在工具名称中包含 [可能失败] 标记
D) 用户应该基于返回值中是否有错误消息来判断

> **答案：B**
> 解析：工具文档应该明确列举所有可能的 failure_type 及其触发条件，这样 Claude 能够理解工具何时会失败以及如何处理各种错误。A 选项忽视了文档的重要性，C 选项不实用，D 选项容易导致误解。好的文档是正确错误处理的基础。

---

## Task 2.3 工具分配与 tool_choice 配置（Q25–Q36）

**Q25.** 在 Claude Agent SDK 中，`allowedTools` 数组的作用是什么？

A) 指定哪些工具可以在该 agent 中使用，实现最小权限原则
B) 按优先级排序工具列表，Claude 优先使用排在前面的工具
C) 定义在该 agent 中强制必须使用的工具
D) 为工具分配使用配额，限制每个工具的调用次数

> **答案：A**
> 解析：allowedTools 是安全控制机制，只有在列表中的工具才可被该 agent 使用。这体现了"最小权限原则"——给 agent 只分配完成其职责所需的工具。B 选项是工具选择策略而非 allowedTools 的目的，C 选项应该用 tool_choice 实现，D 选项不是 allowedTools 的功能。

**Q26.** 你正在为内容审核 agent 设计工具权限。该 agent 需要读取用户提交的内容并标记违规项，但不应该能够删除或修改用户数据。应该在 `allowedTools` 中包含什么？

A) `["read_content", "flag_violation"]`
B) `["read_content", "flag_violation", "delete_content", "modify_content"]`（虽然不会用，但保有备用）
C) `["read_content", "flag_violation", "send_notification"]`
D) 所有可用工具，然后在 agent 提示中说明不应删除数据

> **答案：A**
> 解析：allowedTools 应该仅包含该 agent 实际需要的工具。A 选项为审核功能必需。B 选项包含了不需要的危险工具（虽然不会用，但暴露了攻击面），D 选项依赖提示词而非配置来保护数据，安全性不足。配置级的限制比提示词更可靠。

**Q27.** `tool_choice: "auto"` 和 `tool_choice: "any"` 的区别是什么？

A) "auto" 和 "any" 完全相同，都让 Claude 自由选择是否使用工具
B) "auto" 让 Claude 自动判断是否需要使用工具；"any" 要求 Claude 必须使用 allowedTools 中的某个工具
C) "auto" 限制工具使用数量；"any" 允许无限次工具调用
D) "any" 只用于多工具场景，"auto" 用于单工具场景

> **答案：B**
> 解析：关键差异在于约束性。"auto" 让 Claude 自由决定是否使用工具（可能根本不调用）；"any" 强制 Claude 在每次回应中必须选择某个工具。对于问答任务用 "auto"，对于必须工具调用的任务用 "any"。C 选项和 D 选项都不准确。

**Q28.** 某 agent 需要在三个操作中选择一个执行：发送邮件、发送短信、或创建任务提醒。应该如何配置 `tool_choice`？

A) `tool_choice: "auto"`，让 Claude 判断是否以及选择哪个工具
B) `tool_choice: "any"`，让 Claude 选择其中一个工具
C) `tool_choice: { "type": "tool", "name": "send_email" }`，强制执行邮件
D) `tool_choice: { "type": "tool", "names": ["send_email", "send_sms", "create_reminder"] }`，让 Claude 从中选一个

> **答案：B**
> 解析：这是必须选择的场景（用户需要某种通知），所以用 "any" 确保 Claude 选择一个工具。A 选项的 "auto" 可能导致 Claude 不调用任何工具；C 选项强制选择邮件不灵活；D 选项的语法不正确（没有 "names" 字段）。

**Q29.** 在 agent 定义中，下列哪种配置最能体现"最小权限原则"，同时保证 agent 能完成其工作？

A) agent_1: { allowedTools: ["read_file", "write_file", "execute_bash", "delete_file"] }
B) agent_1: { allowedTools: ["read_file"] }，当需要写入时调用 agent_2: { allowedTools: ["write_file"] }
C) agent_1: { allowedTools: ["read_file", "write_file"] }，tool_choice: "auto"
D) 所有 agent 共享所有工具，由 agent 自行判断调用

> **答案：C**
> 解析：选项 C 为该 agent 配置了完成基本文件操作所需的两个工具，排除了不必要的危险操作（execute_bash、delete_file）。A 选项权限过多，B 选项过度分割增加复杂性，D 选项违反最小权限原则。在安全和功能间取得适当平衡是关键。

**Q30.** 某系统有两个 agent：agent_finance（处理财务数据）和 agent_public（生成公开报告）。应该如何分配工具权限来防止财务数据泄露？

A) 两个 agent 都可以访问所有工具，依靠提示词和审查来防止滥用
B) agent_finance 的 allowedTools 只包含财务数据库工具；agent_public 的 allowedTools 只包含报告模板和聚合统计工具，不包含原始数据访问
C) agent_finance 可以访问所有工具，agent_public 只能访问报告工具
D) 使用统一的 allowedTools，但在 tool_choice 中限制 agent_public 的工具选择

> **答案：B**
> 解析：选项 B 正是最小权限原则的应用。agent_public 完全不能访问原始财务数据工具，只能使用经过聚合的统计接口。A 选项依赖提示词不安全，C 选项中 agent_finance 权限不必要地大，D 选项通过 tool_choice 限制不如通过 allowedTools 限制可靠。

**Q31.** 某工作流需要强制使用特定工具完成某步操作。应该用什么配置？

A) `tool_choice: { "type": "tool", "name": "specific_tool_name" }`
B) `allowedTools: ["specific_tool_name"]` 加 `tool_choice: "auto"`
C) 在 agent 提示中明确指示使用该工具
D) 创建一个只包含该工具的 subagent

> **答案：A**
> 解析：选项 A 使用 tool_choice 的具体工具指定方式，强制 Claude 调用指定工具。B 选项虽然限制了可用工具但 "auto" 仍可能导致不调用，C 选项依赖提示不够可靠，D 选项过度工程化。tool_choice 字段的正是专为强制工具选择而设计。

**Q32.** 以下哪个配置组合最能防止某个不受信任的 agent 滥用敏感工具？

A) allowedTools: [所有工具]，tool_choice: "auto"，依靠 agent 提示词的指导
B) allowedTools: []（不分配任何工具），让 agent 只能通过 agent.call 委托给其他 agent
C) allowedTools: [业务需要的最少工具]，tool_choice: "auto"，定期审计 agent 行为
D) allowedTools: [业务需要的最少工具]，tool_choice: "any"

> **答案：C**
> 解析：选项 C 结合了最小权限原则（allowedTools 最少化）、适度自由（"auto"）和事后监控（审计），是实际可行的安全方案。A 选项过于开放，B 选项过于限制，D 选项中 "any" 要求强制工具调用未必需要。安全的工程实践需要配置和监控的结合。

**Q33.** 在 subagent 架构中，parent agent 应该如何控制 subagent 的工具权限？

A) 在 subagent 的 allowedTools 中定义权限，parent agent 不能覆盖
B) 在 parent agent 中动态指定 subagent 的工具列表（如果支持）
C) 既在 subagent 的 allowedTools 中定义权限，也在 parent agent 的配置中确认，取两者的交集
D) 完全由 parent agent 控制，subagent 不需要定义 allowedTools

> **答案：A**
> 解析：subagent 的权限应该在其自身定义中明确（allowedTools），这确保了权限是可配置的、可审计的，不依赖 parent agent 的正确配置。B 选项虽然提供了灵活性但难以审计，C 选项增加复杂性，D 选项将权限控制推给 parent agent 容易出错。最小权限原则应该在每个 agent 级别执行。

**Q34.** 某任务需要 agent 调用工具，如果调用失败则返回错误给用户（不重试）。应该使用什么 tool_choice 策略？

A) `tool_choice: "auto"` 加 max_retries: 0
B) `tool_choice: "any"` 加 max_retries: 0
C) `tool_choice: "auto"`，让 Claude 自行决定是否调用和如何处理错误
D) 自定义 tool_choice，明确指定要调用的工具

> **答案：B**
> 解析：B 选项确保了三点：1) "any" 强制工具调用（完成任务），2) max_retries: 0 表示错误时不重试，3) Claude 会在工具失败时向用户返回错误。A 选项的 "auto" 可能导致根本不调用工具，C 选项可能导致不调用，D 选项虽然可行但不如 "any" 简洁。

**Q35.** 在多工具场景中，应该如何区别使用 `allowedTools` 和 `tool_choice` 来管理工具访问？

A) allowedTools 和 tool_choice 功能完全相同，选一个即可
B) allowedTools 定义安全边界（哪些工具可用），tool_choice 定义选择策略（是否强制使用、如何选择）
C) allowedTools 用于多工具场景，tool_choice 用于单工具场景
D) allowedTools 在配置时设定，tool_choice 在运行时动态设定

> **答案：B**
> 解析：这是两个不同维度的配置。allowedTools 答的是"什么工具可以使用"（安全性），tool_choice 答的是"如何选择使用"（策略性）。A 选项错误，C 选项和 D 选项都不准确。理解两者的区别是正确设计 agent 权限的关键。

**Q36.** 某 agent 需要偶尔调用工具（不是每次都必须），但当调用时必须是指定的三个工具之一。应该如何配置？

A) `allowedTools: ["tool_a", "tool_b", "tool_c"]`，`tool_choice: "auto"`
B) `allowedTools: ["tool_a", "tool_b", "tool_c"]`，`tool_choice: "any"`
C) `tool_choice: [{ "type": "tool", "name": "tool_a" }, { "type": "tool", "name": "tool_b" }, { "type": "tool", "name": "tool_c" }]`
D) `allowedTools: ["tool_a", "tool_b", "tool_c"]`，`tool_choice: { "type": "tool", "names": ["tool_a", "tool_b", "tool_c"] }`

> **答案：A**
> 解析：选项 A 既限制了可用工具范围（allowedTools），又允许 Claude 自由决定是否调用（tool_choice: "auto"）。B 选项会强制每次都调用某个工具，C 和 D 选项的语法不正确。理想的配置是既保有安全边界（allowedTools）又提供选择自由（"auto"）。

---

## Task 2.4 MCP 服务器集成配置（Q37–Q48）

**Q37.** `.mcp.json` 和 `~/.claude.json` 两个配置文件有什么主要区别？

A) 功能完全相同，位置不同
B) `.mcp.json` 在项目根目录，用于项目共享配置（纳入 git）；`~/.claude.json` 在用户主目录，用于个人配置（包含个人凭证）
C) `.mcp.json` 用于开发环境，`~/.claude.json` 用于生产环境
D) `.mcp.json` 由 Claude 管理，`~/.claude.json` 由用户管理

> **答案：B**
> 解析：这是关键区别。.mcp.json 应该纳入版本控制，与团队共享，包含公共配置；~/.claude.json 仅在本地，包含个人 API 密钥和敏感凭证。C 选项混淆了环境与文件位置，D 选项的表述不准确。

**Q38.** 某 MCP 服务器需要从环境变量 `OPENAI_API_KEY` 读取密钥。在 `.mcp.json` 中应该如何配置，才能实现环境变量展开？

A) `"env": { "API_KEY": "OPENAI_API_KEY" }`
B) `"apiKey": "$OPENAI_API_KEY"`
C) `"apiKey": "${OPENAI_API_KEY}"`
D) `"apiKey": "process.env.OPENAI_API_KEY"`

> **答案：C**
> 解析：MCP 标准使用 `${ENV_VAR}` 语法展开环境变量。选项 C 正确。A 选项是配置映射而非展开，B 选项缺少花括号，D 选项是 JavaScript 语法不适用。了解 ${} 展开语法是配置 MCP 的基础。

**Q39.** 某 MCP 服务器配置如下：

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

如果环境变量 `MYSERVER_API_KEY` 未设置，会发生什么？

A) MCP 会使用默认值 "MYSERVER_API_KEY"
B) MCP 会抛出错误并拒绝启动该服务器
C) 环境变量保持原样 "${MYSERVER_API_KEY}"（未展开），由服务器处理
D) MCP 会提示用户设置环境变量

> **答案：C**
> 解析：如果环境变量不存在，${} 展开通常保持原样（取决于实现）或展开为空字符串。大多数情况下，是否必需某个环境变量应该由服务器本身决定，而不是 MCP 框架。B 选项过于严格，A 和 D 选项都不符合通常行为。

**Q40.** 某项目的 `.mcp.json` 和用户的 `~/.claude.json` 都定义了 `myserver` 服务器，但配置不同。哪个会生效？

A) 两个配置会合并，冲突部分由 `.mcp.json` 优先
B) 两个配置会合并，冲突部分由 `~/.claude.json` 优先
C) 只有 `.mcp.json` 生效，`~/.claude.json` 被忽略
D) 只有 `~/.claude.json` 生效，`.mcp.json` 被忽略

> **答案：B**
> 解析：用户配置（~/.claude.json）通常优先级更高，允许用户在本地覆盖项目配置。这对于添加个人凭证或调整参数很重要。A 选项的优先级反了，C 和 D 选项太绝对。优先级设计让项目和个人配置能够协同。

**Q41.** 某 MCP 服务器使用 stdio 传输方式。配置中应该如何指定？

A) `"transport": "stdio"`
B) `"type": "stdio"`
C) `"protocol": "stdio"`
D) 配置 `"command"` 和 `"args"`，stdio 是隐含的默认值

> **答案：D**
> 解析：stdio 是最常见的传输方式，当配置中包含 `command` 和 `args` 时，默认使用 stdio。无需显式指定 "stdio" 关键字。A、B、C 选项的字段名都不正确。了解隐含默认值能简化配置。

**Q42.** 某 MCP 服务器需要使用 SSE（Server-Sent Events）传输方式连接。配置应该如何写？

A) `"transport": "sse"，"url": "https://mcp.example.com/sse"`
B) `"type": "sse"，"endpoint": "https://mcp.example.com/sse"`
C) `"command": "sse"，"args": ["https://mcp.example.com/sse"]`
D) `"protocol": "sse"，"connection": "https://mcp.example.com/sse"`

> **答案：A**
> 解析：SSE 传输需要指定 `transport: "sse"` 和 URL。选项 A 的字段名最接近标准。B、C、D 选项的字段名都不准确。SSE 是远程连接的传输方式，与 stdio（本地命令）不同。

**Q43.** 某 MCP 服务器配置需要在启动时传递多个参数：

```
python my_server.py --port 5000 --debug --config /etc/config.yaml
```

在 `.mcp.json` 中应该如何配置？

A) `"command": "python my_server.py --port 5000 --debug --config /etc/config.yaml"`
B) `"command": "python"，"args": ["my_server.py", "--port", "5000", "--debug", "--config", "/etc/config.yaml"]`
C) `"command": "python my_server.py"，"arguments": "--port 5000 --debug --config /etc/config.yaml"`
D) `"script": "python my_server.py --port 5000 --debug --config /etc/config.yaml"`

> **答案：B**
> 解析：`args` 数组将命令行参数分解为单个元素，这样 MCP 能够正确解析和处理。A 选项在单个字符串中包含参数（需要 shell 解析），C 和 D 选项的字段名不正确。明确的 args 数组更便于程序处理。

**Q44.** 在 `~/.claude.json` 中为某 MCP 服务器添加个人 API 凭证时，应该使用什么格式来引用项目 `.mcp.json` 中定义的同一个服务器？

A) 在 `~/.claude.json` 中完整重新定义 mcpServers 及所有参数
B) 在 `~/.claude.json` 中只定义要覆盖的字段（如 API 凭证），使用相同的服务器名称
C) 创建新的服务器名称，如 `"myserver_personal"`
D) 在项目 `.mcp.json` 中预留占位符，`~/.claude.json` 只需提供值

> **答案：B**
> 解析：MCP 配置支持合并，用户可以在 `~/.claude.json` 中只写要覆盖的部分（通常是凭证）。B 选项最高效，避免重复。A 选项导致维护成本高，C 和 D 选项都不实际。配置合并是 MCP 设计的关键特性。

**Q45.** Claude Code CLI 启动时，应该用什么命令行标志指定自定义 MCP 配置文件？

A) `--mcp-config /path/to/config.json`
B) `--config-file /path/to/config.json`
C) `-c /path/to/config.json`
D) `--mcp-servers-config /path/to/config.json`

> **答案：A**
> 解析：`--mcp-config` 是标准 Claude Code 标志，用于指定 MCP 配置文件路径。其他选项虽然看起来合理但不是实际的 CLI 接口。了解正确的 CLI 标志避免配置错误。

**Q46.** 某 MCP 服务器需要访问项目的私钥文件（位于项目目录内），环境变量和相对路径应该如何配置？

A) `"env": { "PRIVATE_KEY_PATH": "/absolute/path/to/key.pem" }` 直接使用绝对路径
B) `"env": { "PRIVATE_KEY_PATH": "./keys/key.pem" }` 使用相对路径（相对于项目根目录）
C) `"env": { "PRIVATE_KEY_PATH": "${PROJECT_ROOT}/keys/key.pem" }` 如果 MCP 支持 PROJECT_ROOT 变量
D) 在服务器启动脚本中硬编码路径

> **答案：C**
> 解析：使用变量（如 `${PROJECT_ROOT}`）最灵活，允许配置在不同机器间移植。B 选项的相对路径取决于工作目录可能不稳定，A 选项缺乏灵活性，D 选项硬编码不可维护。环境变量展开是跨机器配置的标准做法。

**Q47.** 团队的 `.mcp.json` 中定义了某个 MCP 服务器，但某个团队成员的本地开发环境使用了不同的配置（端口、凭证）。应该如何处理，既能共享项目配置，又能支持个人差异？

A) 每个开发者维护自己的 `.mcp.json` 副本，不纳入 git
B) 项目 `.mcp.json` 定义共享配置（如服务器命令），个开发者在 `~/.claude.json` 中覆盖个人参数（如端口、凭证）
C) 创建 `.mcp.json.template`，开发者复制后手动填充
D) 在项目 `.mcp.json` 中为每个开发者创建分支配置

> **答案：B**
> 解析：选项 B 是最佳实践。项目配置（command、args 等）在 `.mcp.json` 中共享，个人凭证和参数在 `~/.claude.json` 中（不纳入 git）。这样既保持项目的统一，又支持个人差异。A 选项导致配置分散，C 选项需要手动同步，D 选项过于复杂。

**Q48.** 某 MCP 服务器使用的 API 密钥因定期轮换而需要更新。这个密钥应该存储在哪里，为什么？

A) 存储在项目 `.mcp.json` 中，便于团队同步
B) 存储在用户 `~/.claude.json` 中，作为个人秘密
C) 存储在操作系统环境变量中（如 `MYSERVER_API_KEY`），`~/.claude.json` 引用它
D) 存储在项目根目录的 `.env` 文件中，由工具自动加载

> **答案：C**
> 解析：API 密钥是敏感凭证，应该存储在操作系统级环境变量中，而不是文件。`~/.claude.json` 可以引用这些环境变量（${MYSERVER_API_KEY}），既安全又便于轮换。A 选项暴露凭证，B 选项局限于单台机器，D 选项的 .env 文件通常也不纳入版本控制但不如环境变量安全。环境变量是凭证管理的标准做法。

---

## Task 2.5 内置工具选择与使用（Q49–Q60）

**Q49.** 在以下场景中，应该使用哪个内置工具：你需要读取一个 1000 行的文本文件，但只关心前 100 行。

A) 使用 `Read` 工具，不指定 offset/limit，读取整个文件然后在代码中处理
B) 使用 `Read` 工具，指定 `offset: 0, limit: 100`，只读取需要的部分
C) 使用 `Bash` 命令 `head -n 100 file.txt`
D) 使用 `Grep` 工具搜索前 100 行内容

> **答案：B**
> 解析：`Read` 工具支持 offset 和 limit 参数，允许高效读取文件的特定部分，这样不浪费带宽和内存。A 选项读取不必要的内容，C 选项虽然也能工作但不如 Read 工具直接，D 选项用途不匹配。了解工具的优化参数能提升效率。

**Q50.** 需要修改文件中的一个特定字符串（例如将"old_value"替换为"new_value"）。应该使用哪个工具？

A) 使用 `Read` 读取整个文件，然后用 `Write` 重新写入修改后的内容
B) 使用 `Edit` 工具，指定 old_string: "old_value", new_string: "new_value"
C) 使用 `Bash` 命令 `sed -i 's/old_value/new_value/g' file.txt`
D) 使用 `Write` 工具直接覆盖文件

> **答案：B**
> 解析：`Edit` 工具专为原地修改设计，指定 old_string 和 new_string 后自动替换，比 A 选项（读改写）更简洁。C 选项虽然也行但需要 bash 执行，D 选项会覆盖整个文件。Edit 是针对性的修改工具。

**Q51.** 需要在一个大型代码库中搜索所有包含 "TODO" 的行。应该使用哪个工具？

A) 使用 `Read` 工具读取所有文件
B) 使用 `Bash` 命令 `grep -r "TODO" .`
C) 使用 `Grep` 工具，指定 pattern: "TODO"
D) 使用 `Glob` 工具查找所有文件然后逐个读取

> **答案：C**
> 解析：`Grep` 工具基于 ripgrep，专门为模式搜索设计，支持递归搜索和多种输出格式。C 选项最直接有效。A 选项低效，B 选项虽然也能工作但不如 Grep 工具的集成更好，D 选项需要多步处理。Grep 是搜索的首选工具。

**Q52.** 需要找出项目中所有 TypeScript 文件。应该使用哪个工具？

A) 使用 `Bash` 命令 `find . -name "*.ts"`
B) 使用 `Glob` 工具，指定 pattern: "**/*.ts"
C) 使用 `Grep` 工具搜索文件扩展名
D) 使用 `Read` 工具遍历目录结构

> **答案：B**
> 解析：`Glob` 工具专为文件模式匹配设计，"**/*.ts" 是标准 glob 模式。B 选项最直接。A 选项虽然也行但需要 bash，C 选项不适合（Grep 搜索内容而非文件名），D 选项不支持目录遍历。Glob 是文件查找的专用工具。

**Q53.** 需要执行一个系统命令，如 `npm install` 或 `python script.py`。应该使用哪个工具？

A) 使用 `Bash` 工具执行命令
B) 使用 `Write` 工具创建脚本文件，然后运行
C) 使用 `Read` 工具检查脚本是否存在
D) 使用 `Grep` 工具查找命令

> **答案：A**
> 解析：`Bash` 工具就是为执行 shell 命令设计的。A 选项直接、高效。B 选项不必要地复杂，C 和 D 选项用途不匹配。Bash 是执行系统命令的标准工具。

**Q54.** 需要从网络获取 JSON 数据（如 API 响应）。应该使用哪个工具？

A) 使用 `Bash` 命令 `curl https://api.example.com/data`
B) 使用 `WebFetch` 工具，指定 URL
C) 使用 `Read` 工具直接读取 URL
D) 使用 `Grep` 工具搜索网络内容

> **答案：B**
> 解析：`WebFetch` 工具专为 HTTP 请求设计，返回结构化响应。B 选项最适合。A 选项虽然也能工作但需要 bash，C 选项 Read 不支持 URL（仅支持本地文件），D 选项用途不匹配。WebFetch 是网络请求的标准工具。

**Q55.** 你需要在项目中搜索所有调用某个函数的位置，例如 `processUser()`。应该使用哪个工具组合最高效？

A) 使用 `Read` 读取所有文件并手动查找
B) 使用 `Bash` 命令 `grep -r "processUser(" .` 加上 `--include="*.js"`
C) 使用 `Glob` 查找所有 .js 文件，然后用 `Grep` 搜索函数调用
D) 使用 `Grep` 工具，pattern: "processUser\(" 和 glob filter: "*.js"

> **答案：D**
> 解析：`Grep` 工具支持 glob filter 参数，能在一个工具调用中完成文件过滤和内容搜索。D 选项最高效、最简洁。A 选项低效，B 选项虽然也行但混合工具，C 选项多步不如 D 简洁。了解工具的高级参数能简化任务。

**Q56.** 需要写入一个 500 行的新文件。应该如何使用 `Write` 工具？

A) 一次性写入所有 500 行
B) 分成多次调用，每次最多 25-30 行，使用 mode: "append"
C) 先用 `Read` 检查文件是否存在，然后决定是否写入
D) 分解为多个小文件，每个文件 50-100 行

> **答案：B**
> 解析：Write 工具的最佳实践是分块写入（每次 25-30 行），这样不会导致超大请求。第一次用 mode: "rewrite"，后续用 mode: "append"。B 选项符合工具使用指南。A 选项可能导致请求过大，C 选项不必要，D 选项人为复杂化。

**Q57.** 需要修改一个文件中的多处内容（例如替换多个字符串）。应该如何使用 `Edit` 工具？

A) 多次调用 `Edit` 工具，每次替换一处
B) 用一个 `Edit` 调用替换所有（如果 old_string 包含所有要改的部分）
C) 使用 `Bash` 的 sed 命令一次性替换
D) 用 `Read` 读整个文件，手动修改，再用 `Write` 覆盖

> **答案：A**
> 解析：当需要替换多处且每处不同时，应该多次调用 `Edit`，每次指定准确的 old_string 和 new_string。A 选项最安全（每步可验证）。B 选项如果 old_string 包含多个要替换的部分则有歧义，C 选项需要 bash，D 选项风险较高（整体覆盖）。多步修改比一步到位更稳妥。

**Q58.** 需要搜索包含特定正则表达式的代码，例如所有使用 `console.log(` 的地方。应该使用哪个工具和参数？

A) 使用 `Grep` 工具，pattern: "console\\.log\\("（转义字符）
B) 使用 `Grep` 工具，pattern: "console.log("，output_mode: "content"
C) 使用 `Bash` 命令 `grep -r "console.log(" .`
D) 使用 `Grep` 工具，pattern: "(console)(log)(\()"

> **答案：A**
> 解析：`Grep` 支持正则表达式（基于 ripgrep），需要转义特殊字符。A 选项正确地转义了点和括号。B 选项缺少转义，C 选项用 bash 可行但不如 Grep 工具集成，D 选项虽然是正则但语法不如 A 简洁。掌握 ripgrep 语法能更有效地搜索。

**Q59.** 比较以下两种方法修改配置文件：方法1 用 `Edit` 工具替换单行；方法2 用 `Read` 读取整个文件，修改，再用 `Write` 覆盖。哪个更好，为什么？

A) 两者等价，选择取决于个人喜好
B) 方法 1（Edit）更好，因为它是原地修改，不涉及整个文件的读写，更安全高效
C) 方法 2（Read+Write）更好，因为能同时进行多处修改
D) 方法 1 容易出错，方法 2 更可靠

> **答案：B**
> 解析：`Edit` 工具专为原地修改设计，只影响特定字符串，更安全（不会意外修改其他部分）。方法 2 虽然支持多处修改，但涉及整个文件的重写，更容易出错。B 选项正确。了解工具特性能选择最恰当的方法。

**Q60.** 某任务需要：1) 查找所有包含特定错误的日志行，2) 提取这些行中的用户 ID，3) 将用户 ID 保存到新文件。应该使用哪个工具组合最高效？

A) 使用 `Bash` 一次性完成所有步骤（grep + awk + > output）
B) 使用 `Grep` 查找日志行，手动提取 ID，使用 `Write` 保存
C) 使用 `Bash` 执行复杂管道 > 使用 `Read` 验证结果 > 使用 `Write` 保存最终结果（过度工程化）
D) 使用 `Grep` 查找日志行（output_mode: "content"），在代码中提取 ID，使用 `Write` 保存

> **答案：D**
> 解析：选项 D 是最佳平衡。Grep 工具专门搜索日志，output_mode 控制输出格式。在代码中提取 ID 更清晰，Write 保存最终结果。A 选项虽然可行但 shell 管道复杂易出错，B 选项"手动提取"不自动化，C 选项层级过多。这个组合既清晰又高效。

---

