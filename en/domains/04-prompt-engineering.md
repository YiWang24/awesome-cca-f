# Domain 4: Prompt Engineering & Structured Output

> **Weight: 20%**
> Official documentation: [Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering) | [Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)

---

## Task Statement Coverage

| Task | Topic |
| ---- | ------------------------------------------ |
| 4.1 | Improving accuracy and reducing false positives with explicit criteria |
| 4.2 | Use few-shot prompting to improve output consistency |
| 4.3 | Use tool use and JSON Schema to force structured output |
| 4.4 | Design verification, retry and feedback closed loops for extraction quality |
| 4.5 | Designing efficient batch processing strategies |
| 4.6 | Designing a multi-instance and multi-round review architecture |

---

## Domain 4 Learning Main Line

Domain 4 does not test "being able to write prompt words", but whether it can turn Claude's output into a system component that is controllable, verifiable, and scalable. The exam will repeatedly distinguish three types of abilities: first, how to use explicit standards, few-shot and structured schema to improve output stability; second, how to use verification, retry, and feedback closed loops to deal with semantic errors that models still make; third, how to make engineering trade-offs based on SLA, cost, context restrictions, and risk of false positives in a batch processing, multi-instance, and multi-round review architecture.

When preparing for the exam, you should understand each task as a design question: when you see "high false positives", give priority to explicit report/skip standards, disable high false positive categories, and severe level examples; when you see "inconsistent formats", give priority to few-shot instead of continuing to pile long instructions; when you see "must be structured", give priority to tool use + JSON Schema instead of requiring the model to "output legal JSON"; when you see "Error in extraction results", you need to determine whether it is a retryable format/structural/semantic error, or whether the retry is invalid due to missing source documents; when you see "a large number of offline tasks", you need to consider the cost of the Batch API and the 24-hour processing window; when you see "large code review", you need to split it into independent instances and multiple passes to avoid self-review bias and dilution of attention.

---

### Task Statement 4.1: Design prompts with explicit criteria to improve precision and reduce false positives

#### Knowledge of:

- The importance of explicit criteria over vague instructions (e.g., "flag comments only when claimed behavior contradicts actual code behavior" vs "check that comments are accurate")
  - The core of the decidable criterion is that the model can directly make true/false decisions based on the code content, rather than relying on subjective modal particles. Abstract instructions (such as "more cautious") do not change the decision boundary and only confuse the model; explicit report/skip lists can make the same input get the same output. The more specific the criteria, the lower the false positive rate and the more consistent the output.
- How general instructions like "be conservative" or "only report high-confidence findings" fail to improve precision compared to specific categorical criteria
  - "High confidence" has no operational definition for the model because the model's self-reported confidence is uncalibrated and does not equal the actual accuracy. Setting evidence thresholds by category (such as "Only report if the behavior described by the annotation is verifiably inconsistent with the logic of the code") can truly constrain the scope of the output. Vague modal particles are suitable for conversation, not for production review systems.
- The impact of false positive rates on developer trust: high false positive categories undermine confidence in accurate categories
  - Trust is a system-level attribute, not a category-level attribute. Developers only need to be exposed to noise a few times before they stop reading all output, including real critical findings. Reducing the false positive rate of a single high false positive category often improves the credibility of the overall system more than simply improving the accuracy of other categories.

#### Skills in:

- Writing specific review criteria that define which issues to report (bugs, security) versus skip (minor style, local patterns) rather than relying on confidence-based filtering
  - Without an explicit skip list, the model will report coding style, naming preferences, and TODO comments as potential problems. Clearly "skip: simple style improvement suggestions and naming methods agreed within the team" in the prompt can effectively reduce low-value findings. The report/skip list should be double enhanced in prompt and few-shot.
- Temporarily disabling high false-positive categories to restore developer trust while improving prompts for those categories
  - Temporarily closing a high false positive category is a trust restoration strategy rather than abandoning the category. Continuously reporting low-quality findings will cause developers to completely ignore the system, which is more harmful than not reporting them temporarily. During the shutdown period, positive and negative samples of this category should be collected, explicit criteria should be improved, few-shot should be added, and then enabled after the false positive rate reaches an acceptable level.
- Defining explicit severity criteria with concrete code examples for each severity level to achieve consistent classification
  - When the severity level definition lacks code examples, the model will make fuzzy inferences based on language expressions, resulting in the same security vulnerability being rated as High today and Critical tomorrow. Each level requires at least one positive example code snippet, and one borderline example that "looks like this but doesn't hit this level." Examples anchor the boundaries of judgment, textual standards only provide the framework.

### Task Statement 4.2: Apply few-shot prompting to improve output consistency and quality

#### Knowledge of:

- Few-shot examples as the most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results
  - Elaboration can express rules but cannot show boundaries. The model will interpret the rules differently when dealing with fuzzy cases, resulting in different outputs from the same input. Few-shot locks in "how the rules are applied on real data" through specific examples, which is especially suitable for scenarios where both answers seem reasonable but only one meets the business needs; piling up more text explanations at this time is usually ineffective, and examples are effective interventions.
- The role of few-shot examples in demonstrating ambiguous-case handling (e.g., tool selection for ambiguous requests, branch-level test coverage gaps)
  - Boundary cases are areas where misjudgments are more likely to occur and where few-shots are most valuable. For example, `config.get("key", default)` and `config["key"]` are respectively safe access and KeyError risks in different contexts. The text is difficult to distinguish clearly, but the code examples can allow the model to accurately identify the differences. Each few-shot example should ideally include a rationale for choosing A over B.
- How few-shot examples enable the model to generalize judgment to novel patterns rather than matching only pre-specified cases
  - A low-quality few-shot only shows specific cases, and what the model learns is "you can answer this only after you have seen this"; a high-quality few-shot shows the basis for judgment, and the model can generalize the principles to new situations. Write "report/skip because of X" in the example so that the model learns the rules instead of just remembering the samples. Good generalization ability means that the model can make correct judgments when encountering new patterns that have not been seen in training examples.
- The effectiveness of few-shot examples for reducing hallucination in extraction tasks (e.g., handling informal measurements, varied document structures)
  - The risk of hallucinations in extraction tasks arises from the tension between the required fields in the schema and the actual missing information in the document. The few-shot example can demonstrate "when the source document does not have field X, return null instead of guessing" and clarify the strategy for handling missing information. The example should also cover non-standard layouts (such as totals in the header rather than at the bottom of the table) to let the model know where to look for fields.

#### Skills in:

- Creating 2-4 targeted few-shot examples for ambiguous scenarios that show reasoning for why one action was chosen over plausible alternatives
  - 2-4 high-quality examples are usually better than 20 general examples, because too many examples will dilute attention, increase context overhead, and even make the model fall into template matching instead of judgment reasoning. Each example should target a known false positive pattern or edge scenario, and explain "why this was chosen instead of that" in the example. Prioritize coverage over quantity.
- Including few-shot examples that demonstrate specific desired output format (location, issue, severity, suggested fix) to achieve consistency
  - Describing the output format in the prompt can easily lead to ambiguity. For example, "contains the location field" may be interpreted in multiple ways. Displaying the complete output with all expected fields directly in a few-shot example removes format ambiguity and allows the model to learn precise field names, nesting levels, and value types. Field stability is critical for downstream system resolution.
- Providing few-shot examples distinguishing acceptable code patterns from genuine issues to reduce false positives while enabling generalization
  - Giving only positive examples ("situations that should be reported") will cause the model to over-activate and report relevant but not need-to-report patterns. Adding counterexamples that "look like but should be skipped" in the few-shot clearly states "this situation is not reported because of X", which can effectively narrow the scope of the report. Positive and negative comparisons are more effective than a literal rule of "Don't report Y" alone because the model can directly perceive the boundaries from the examples.
- Using few-shot examples to demonstrate correct handling of varied document structures (inline citations vs bibliographies, methodology sections vs embedded details)
  - The same type of information may appear in completely different formats in different documents: citations can be inline [1] or bibliography; methodologies can be in independent chapters or embedded in text footnotes. If the few-shot only covers one format, the model will fail to extract or produce null when encountering other layouts. The sample set should systematically cover the main structural variations of the document.
- Adding few-shot examples showing correct extraction from documents with varied formats to address empty/null extraction of required fields
  - Fields returning null may not necessarily be because there is no information in the document, but may also be because the model is not found in the correct location. When the field extraction under a specific layout fails, you must first diagnose whether it is really missing or a format matching failure, and then decide whether to add an example or allow null. Adding corresponding examples for layouts with known high failure rates is a direct means to improve field recall.

### Task Statement 4.3: Enforce structured output using tool use and JSON schemas

#### Knowledge of:

- Tool use (tool_use) with JSON schemas as the most reliable approach for guaranteed schema-compliant structured output, eliminating JSON syntax errors
  - The requirement to "output JSON" in prompt is only a prompt constraint. The model may add explanatory text before and after JSON, output Markdown code blocks, or generate syntax errors. Tool use combined with `input_schema` is a mandatory constraint at the API level: the tool call input generated by the model must conform to the schema, which naturally eliminates JSON syntax errors. When stable fields are required in a production environment, tool use should be preferred over prompt-only JSON extraction.
- The distinction between tool_choice: "auto" (model may return text instead of calling a tool), "any" (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)
  - `tool_choice: "auto"` allows the model to decide independently whether to call the tool. When it encounters input that it believes can be answered directly in text, the tool call may be bypassed, resulting in a lack of structured output. `tool_choice: "any"` forces the model to select a call from available tools, suitable for scenarios where the document type is unknown but must be output in a structured manner. Forced tool selection (`{"type": "tool", "name": "X"}`) is used when multiple steps must be performed in a specific order, such as metadata extraction first and then enhancement.
- That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)
  - JSON Schema verifies field existence, type, and enumeration range, but does not verify the business relationship between fields. Invoice line item amounts that do not add up to total_amount, date fields that are formatted correctly but come from the wrong text location, and category enumerations that are legal but have the wrong semantics are not captured by the schema. After structured output, a business verification layer must be added to check the calculation consistency and semantic correctness.
- Schema design considerations: required vs optional fields, enum fields with "other" + detail string patterns for extensible categories
  - Setting a field to required will put pressure on the model. When the source document does not have corresponding information, the model may generate false values that appear reasonable in order to satisfy the schema. Setting nullable (`["string", "null"]`) allows the model to honestly return null. When an Enum does not have the `"other"` option, the model may shoehorn unclassifiable values ​​into the closest enum instead of marking them as unknown.

#### Skills in:

- Defining extraction tools with JSON schemas as input parameters and extracting structured data from the tool_use response
  - The purpose of tool use is to have the model return data in a structured way rather than generating JSON in a natural language response. The structured result is located in the `input` field of the block with type `tool_use` in the response `content` array. The type is already Python dict, and no JSON parsing is required. If the code is still trying to parse the JSON of the `text` type block, it means there is a misunderstanding of the tool use response format.
- Setting tool_choice: "any" to guarantee structured output when multiple extraction schemas exist and the document type is unknown
  - When the system needs to process multiple document types (such as invoices, contracts, receipts) but the specific types are not known in advance, you can define multiple extraction tools and set `tool_choice` to `"any"` to let the model select the most appropriate tool to call based on the document content. This is more accurate than using a single universal schema, because different document types have different sets of fields, and using the best matching schema reduces the risk of null fields and fabrication.
- Forcing a specific tool with tool_choice: {"type": "tool", "name": "extract_metadata"} to ensure a particular extraction runs before enrichment steps
  - In the multi-step extraction process, some steps have strict execution order dependencies. For example, `extract_metadata` must be called first to obtain the document type before subsequent enhancement strategies can be determined accordingly. Forced tool selection (`{"type": "tool", "name": "extract_metadata"}`) ensures that this step must be executed to prevent the model from judging "can be skipped" or selecting other tools. This is a reliable means of achieving an orderly multi-step workflow.
- Designing schema fields as optional (nullable) when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields
  - When a field in the schema is marked as required, the model will try to fill it in - even if the source document has no corresponding information at all, it will generate a false value that looks reasonable. Making possible missing fields optional and allowing null (`"type": ["string", "null"]`, do not put it in the `required` array) gives the model a legal "undetermined" exit, which can effectively reduce illusions.
- Adding enum values like "unclear" for ambiguous cases and "other" + detail fields for extensible categorization
  - If the Enum field does not have the `"unclear"` or `"other"` option, when the model encounters a situation where the classification cannot be determined, the closest value will be hard selected from the existing enumeration, resulting in classification errors. Adding `"unclear"` allows the model to express uncertainty, and adding the detail field to `"other"` allows the model to record specific reasons. Downstream systems can route to human review for both types of values, rather than accepting an incorrect forced classification.
- Including format normalization rules in prompts alongside strict output schemas to handle inconsistent source formatting
  - JSON Schema can constrain the field type to string, but cannot constrain whether the date is ISO 8601 (`YYYY-MM-DD`) or `MM/DD/YYYY`. In the prompt, it is clear that "dates should be in ISO 8601 format, amounts should be in numbers without currency symbols, and units should be in the International System of Units" to avoid downstream parsing failures caused by legal fields but inconsistent formats. Format specifications and schema need to be used side by side, and one is indispensable.

### Task Statement 4.4: Implement validation, retry, and feedback loops for extraction quality

#### Knowledge of:

- Retry-with-error-feedback: appending specific validation errors to the prompt on retry to guide the model toward correction
  - Retrying without error information is equivalent to calling it again with the same input. The model has no new information and has a high probability of producing the same error. An effective retry requires attaching a specific description of the verification failure (such as "line_items totals 128.50 but total_amount is 132.50, a difference of 4.00") to the original document and the failure result, so that the model knows where the error is and what to change. The more specific the error, the more successful the model will be in self-correcting.
- The limits of retry: retries are ineffective when the required information is simply absent from the source document (vs format or structural errors)
  - The premise of retrying is that the error comes from the output problem of the model (format error, wrong field position, inconsistent calculation), rather than the lack of information in the source document itself. When the reason for verification failure is "this field does not exist in the document at all", no matter how many times the model is prompted, it will not be able to generate real information and will only increase the probability of hallucinations. The system needs to specify the stopping conditions: when a field is missing, it is marked as null and stops retrying, and when the information is in an external document, manual processing is upgraded.
- Feedback loop design: tracking which code constructs trigger findings (detected_pattern field) to enable systematic analysis of dismissal patterns
  - Adding the `detected_pattern` field to each finding (such as the code snippet or pattern description that triggers the rule) can convert each false positive into analyzable data. Counting which patterns are repeatedly ignored by developers can identify systemic problem categories that are either defined too broadly by the standard or have insufficient few-shot examples. Without structured schema fields, false positive analysis can only rely on manual comments and cannot be systematically improved.
- The difference between semantic validation errors (values don't sum, wrong field placement) and schema syntax errors (eliminated by tool use)
  - Tool use with JSON Schema can eliminate JSON syntax errors (braces mismatch, field type errors) at the API level, but it cannot detect semantic errors. Semantic errors refer to output that is structurally legal but has wrong business meaning. For example, the sum of line items does not match the total, the date field comes from the title instead of the text, and the category enumeration selects items with semantic errors. Semantic verification requires program logic: calculation cross-validation, regular format checking, and business rule assertions.

#### Skills in:

- Implementing follow-up requests that include the original document, the failed extraction, and specific validation errors for model self-correction
  - The retry prompt needs to contain three elements: the original document (to let the model refer to the source again), the failed extraction result (to let the model know what was output before), and the specific verification error (to let the model know where to change). Lack of any one of them will reduce the success rate of correction-without the original text model, it cannot be re-extracted, without the failure result model, it does not know what to correct, and without specific errors, the model may correct the wrong place.
- Identifying when retries will be ineffective (e.g., information exists only in an external document not provided) versus when they will succeed (format mismatches, structural output errors)
  - Distinguishing between "retryable errors" and "non-retryable errors" is the key to feedback closed-loop design. Format errors (date format mismatch, wrong field nesting level, incorrect case of enumeration values) are correctable, and can usually be fixed by retrying and providing specific error feedback. Missing information (the source document does not have the field, the information is in another document but is not provided) is not correctable, and retrying will only produce made-up values ​​and should simply return null or mark it as unclear and stop.
- Adding detected_pattern fields to structured findings to enable analysis of false positive patterns when developers dismiss findings
  - The `detected_pattern` field records the specific code structure or text pattern that triggered the finding, such as "direct access to dict without checking for key existence" or "parameter does not contain type annotation". When a developer dismisses a finding, the system can associate the dismiss action with a specific trigger mode and count which types of findings are continuously ignored. This is the smallest viable closed loop that turns human feedback into prompt improvement data.
- Designing self-correction validation flows: extracting "calculated_total" alongside "stated_total" to flag discrepancies, adding "conflict_detected" booleans for inconsistent source data
  - Require both `calculated_total` (the result of adding up the line items) and `stated_total` (the total amount declared in the document) in the schema, plus the `conflict_detected` Boolean field to let the model do the cross calculations and flag inconsistencies on its own, rather than extracting only one total field. This embeds logic that would otherwise require external validation into the extraction step, allowing both data and data quality metrics to be obtained with a single API call. Downstream systems can use this to decide whether to retry or escalate manually.

### Task Statement 4.5: Design efficient batch processing strategies

#### Knowledge of:

- The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA
  - The Message Batches API trades latency for cost: asynchronous processing provides roughly a 50% discount, but requests can take up to 24 hours and there is no low-latency SLA. It fits offline, non-blocking, latency-tolerant jobs such as nightly document processing, historical data labeling, and batch test generation. It does not fit user-facing requests, pre-merge checks, prerequisite gates, or agentic loops that need tool calls during processing.
- Batch processing is appropriate for non-blocking, latency-tolerant workloads (overnight reports, weekly audits, nightly test generation) and inappropriate for blocking workflows (pre-merge checks)
  - The core decision criterion is latency tolerance. If the call blocks a user, a developer waiting to merge, or a downstream automation step, use the synchronous API. Use the Batch API only when results can arrive later. Decide in this order: confirm the SLA and whether the workflow is blocking, check whether the 24-hour batch window is acceptable, then optimize for cost.
- The batch API does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)
  - Each request of the Message Batches API is an independent single-turn conversation: issue a prompt and get a complete response. If the task requires the model to call a tool, wait for the tool to return the results, and then continue inference (agentic loop), the Batch API cannot support this mode because the requests within the batch cannot exchange information during processing. Workflows that involve multiple rounds of tool calls must use the synchronous API and proceed step by step.
- custom_id fields for correlating batch request/response pairs
  - Responses to Batch requests do not guarantee order, and some requests within the batch may fail while others succeed. `custom_id` is a user-defined identifier used to accurately match each result in the output back to the corresponding input document. Without `custom_id`, you can only resubmit the entire document after failure; with `custom_id`, you can filter out the failed items, analyze the reasons for the failure, modify them according to the reasons, and then resubmit only this part of the document, saving costs and time.

#### Skills in:

- Matching API approach to workflow latency requirements: synchronous API for blocking pre-merge checks, batch API for overnight/weekly analysis
  - Cost optimization must not increase critical-path latency. Blocking workflows need the synchronous API; offline workflows can use the Batch API when the SLA permits delayed delivery. Real-time tasks should never move to batch purely to save money.
- Calculating batch submission frequency based on SLA constraints (e.g., 4-hour windows to guarantee 30-hour SLA with 24-hour batch processing)
  - Batch API latency is measured from submission, but end-to-end latency also includes the time an item waits for the next submission window. If a job is submitted every 4 hours and processing can take up to 24 hours, worst-case end-to-end latency is about 4 + 24 = 28 hours. Submission frequency should be derived from the business SLA, with extra room for failed-item retries.
- Handling batch failures: resubmitting only failed documents (identified by custom_id) with appropriate modifications (e.g., chunking documents that exceeded context limits)
  - The reasons for Batch request failure may vary: some are because the document exceeds the context window (chunk is required), some are prompt format issues (prompt needs to be adjusted), and some are when the model returns an incorrect structure (schema or few-shot needs to be changed). After using `custom_id` to filter out failed items, they must be grouped and processed according to the failure reasons. All failed documents cannot be resubmitted with the same modifications. Accurate handling of failed items is the key to batch system reliability.
- Using prompt refinement on a sample set before batch-processing large volumes to maximize first-pass success rates and reduce iterative resubmission costs
  - Do not run a full batch before validating the prompt. If the prompt is defective, feedback may arrive only after a long processing window, wasting both cost and latency budget. Standard practice is to test 20-50 representative samples with the synchronous API, iterate until the sample pass rate is acceptable, then submit the full batch.

### Task Statement 4.6: Design multi-instance and multi-pass review architectures

#### Knowledge of:

- Self-review limitations: a model retains reasoning context from generation, making it less likely to question its own decisions in the same session
  - The model forms internal reasoning paths when generating code or content, and these paths remain in the context of the same session. When asked to review something it just generated, the model tends to verify the output according to the reasoning it generated, rather than independently re-evaluating it. This kind of confirmation bias causes generators to easily ignore errors in their initial judgments, and it is difficult to completely eliminate them even if extended thinking is added.
- Independent review instances (without prior reasoning context) are more effective at catching subtle issues than self-review instructions or extended thinking
  - An independent Claude instance does not carry the original generated reasoning context and will conduct an independent evaluation when receiving the content to be reviewed, similar to having another engineer review the same piece of code. This isolation makes it easier for review instances to notice problems that the generated instance might have overlooked due to preconceptions, such as implicit precondition assumptions, interface contract violations, or missing boundary handling. The quality of independent reviews is generally higher than self-review instructions for the same model.
- Multi-pass review: splitting large reviews into per-file local analysis passes plus cross-file integration passes to avoid attention dilution and contradictory findings
  - Inputting dozens of files at once for code review will lead to attention dilution. The model may forget details of earlier files when analyzing subsequent files, resulting in contradictory or incomplete findings. The local pass only processes one file at a time and focuses on local bugs, exception handling, and boundary conditions; the integrated pass aggregates local results and focuses on cross-file data flow, interface contracts, and state consistency. The two types of passes use independent calls.

#### Skills in:

- Using a second independent Claude instance to review generated code without the generator's reasoning context
  - When implementing independent review, the review instance should only receive the code to be reviewed and the review task description, but not the reasoning process, design description, or intent explanation that generated the instance. Reasoning from the generative process implicitly leads reviewers to believe that the design is sound, thereby reducing critical findings. Clean context isolation is the prerequisite for independent review of effectiveness, and is also the essential difference from "requiring the same session to think again".
- Splitting large multi-file reviews into focused per-file passes for local issues plus separate integration passes for cross-file data flow analysis
  - The review scope of local passes is strictly limited to a single file: the correctness of variables, completeness of exception handling, boundary conditions, and local logic errors. The input of the integration pass is a structured summary of each local pass, focusing on the cross-module dimension: whether the data type transmitted by module A meets the expectations of module B, whether the status is propagated consistently among multiple modules, and whether all callers have been updated synchronously when the interface changes. The two types of passes have different goals, and mixing will interfere with each other.
- Running verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing
  - The self-reported confidence (high/medium/low) of the model is not equal to the actual accuracy before calibration. "High confidence" may correspond to 70% true accuracy, or it may correspond to 95%, depending on the model, task and domain. Only after using the labeled verification set to estimate the actual accuracy of each confidence level can a safe diversion threshold be set: those exceeding a certain accuracy can be automatically processed, and those below the threshold will be subject to manual review. Confidence levels are used to shut down manual review without calibration and are extremely risky.

---

## Task 4.1: Explicit criteria and reducing false positives

### What this tests

The core of this question is "precision comes from executable standards, not from adjectives." If a code review, content review or classification scenario is given in the exam and it is stated that there are too many false positives in the output, the right direction is usually not to change the prompt to "more cautious" and "only output high-confidence results", but to clearly break down the problem categories: which ones must be reported, which ones must be skipped, and what evidence corresponds to each severity level. In practice, false positives will directly damage developer trust, so high false positive categories can be temporarily closed and then restored with clearer standards and examples.

### Core principle: Fuzzy instructions → unstable results

```python
# Ambiguous instructions (unpredictable results)
system = "Please be more cautious and only report high confidence questions"

# Explicit classification criteria (predictable results)
system = """
Report issues in the following categories (required):
- Hardcoded keys, passwords, API Tokens
- SQL injection, command injection vulnerabilities
- Unhandled KeyError/IndexError risk

The following categories are skipped (not reported):
- Performance optimization suggestions (unless obviously O(n²) or worse)
- Code style issues (indentation, naming preferences)
- TODO comments
"""
```

### Practical Impact on False Alarm Rate

> High false positive rates not only damage trust in that category, but also in all categories, causing developers to ignore all output, including truly serious issues.

### Strategies to Reduce False Positives

1. **Explicitly list reported/skipped categories** (instead of vague "confidence filtering")
2. **Temporarily close the high false positive category**: restore user trust in the output first, and then gradually optimize the prompts in this category
3. **Define clear criteria** for each severity level with positive/negative code examples
4. **Distinguish between "definitely a problem" and "possibly a problem"**: report them separately, don't mix them together

### Tip structure skills

**XML tags** are used to clearly separate different parts of the prompt:

```xml
<system>
You are a code security review tool.

<report_categories>
Categories that must be reported:
- Hardcoded credentials
- Injection vulnerabilities
- Permission bypass
</report_categories>

<skip_categories>
Categories skipped:
- Style issues
- Performance recommendations
- TODO comments
</skip_categories>

<output_format>
Output in JSON array format, each item contains severity, category, description, line
</output_format>
</system>
```

**Prefill**: Prefill content at the beginning of the assistant's turn to guide a specific output format:

````python
messages = [
    {"role": "user", "content": "Analyze the following code:\n```python\n...\n```"},
    {"role": "assistant", "content": "["} # Prefill JSON array starter
]
# Claude will continue to fill in the array content instead of outputting explanation text first
````

---

## Task 4.2: Few-Shot Prompting

### What this tests

The test point of Few-shot is "using examples to teach judgment boundaries." When elaboration still yields inconsistencies, unstable formatting, misidentification of edge cases, or hallucinations of extraction tasks, a few-shot is often more effective than continuing to add rules. High-quality examples should show why one is chosen between two reasonable options, and also show the difference between "should report" and "should not report". In this way, the model learns the judgment principle rather than mechanically matching a few fixed samples.

### When to use

| Situation | Recommended Strategy | Reason |
| -------------------------------- | ---------------- | ------------------ |
| Output is inconsistent with detailed descriptions alone | Few-shot examples | Examples are more precise than descriptions |
| Fuzzy boundary scenarios (acceptable vs real problem) | Comparison examples | Examples showing both sides of the boundary |
| Analogical generalization of new patterns | Few-shot | Let the model understand the intent of the pattern |
| The format has specific requirements | Few-shot | Directly display the expected format |

### Effective Few-Shot design principles

1. **Quantity is not the key, representativeness is**: 2-4 high-quality examples > 20 general examples
2. **Covering Boundary Scenarios**: The best examples target situations where the model is most likely to hesitate.
3. **Positive and Negative Contrast**: Display "should be reported" and "should not be reported" at the same time
4. **Format Consistent**: All examples use the exact same input/output format

````python
FEW_SHOT_SYSTEM = """You are a code security review tool.

--- Example 1 (must be reported) ---
Code:
```python
API_KEY = "sk-prod-abc123-secret"
````

Output: {"severity": "critical", "category": "hardcoded_credential", "line": 1}

---Example 2 (skip)---
Code:

```python
# TODO: Optimize this loop
for i in range(len(items)):
    process(items[i])
```

Output: {"severity": "skip", "reason": "Performance optimization suggestions are not within the scope of the report"}

--- Example 3 (Boundary: Acceptable Access Pattern vs Real Problem) ---
Acceptable: `config.get("timeout", 30)` → skip (safe default access)
Real problem: `config["timeout"]` → reported without validating key existence (KeyError risk)
"""

````

---

## Task 4.3: JSON Schema forces structured output

### What this tests

This section focuses on distinguishing between "looking like JSON" and "structured output guaranteed by the API mechanism". When the production system requires stable fields, tool use `input_schema` should be used, and then the results should be read from `tool_use.input`; simply requiring JSON in the prompt cannot eliminate syntax errors. Also remember that schema can only constrain the shape and cannot guarantee that the sum of amounts, field semantics, and source consistency are correct, so subsequent business verification is still required.

### Comparison of three structured output methods

| Method | Reliability | Applicable scenarios |
|------|--------|---------|
| Normal prompt requires JSON | Low (may output description text + JSON) | Not recommended for production environment |
| `tool_choice: "any"` + JSON Schema | High (guaranteed to call the tool and return structure) | When the document type is unknown |
| `tool_choice: {"type": "tool", "name": "..."}` | Highest (force specific tool) | Known that specific extraction must be run first |

### temperature and structured output

```python
# Structured extraction uses low temperature (reduces randomness and improves consistency)
response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=512,
    temperature=0, # 0 = most deterministic; default is usually 1.0
    tools=[extraction_tool],
    tool_choice={"type": "any"},
    messages=messages
)

# Idea generation uses high temperature (increases diversity)
response = client.messages.create(
    temperature=1.0, #higher randomness
    ...
)
````

### JSON Schema design pattern

```python
extraction_tool = {
    "name": "extract_invoice_data",
    "description": "Extract structured data from invoice documents",
    "input_schema": {
        "type": "object",
        "properties": {
            # Required fields
            "invoice_number": {
                "type": "string",
                "description": "Invoice number, fill in if it does not exist'UNKNOWN'"
            },
            # nullable field (may not exist in the document)
            "total_amount": {
                "type": ["number", "null"], # Prevent the model from fabricating non-existent amounts
                "description": "Total invoice amount (null if unclear in document)"
            },
            # enum constraints (limited options)
            "currency": {
                "type": "string",
                "enum": ["USD", "EUR", "CNY", "GBP", "other"],
                "description": "currency type"
            },
            # other + detail mode (handling situations where enum cannot be exhausted)
            "currency_detail": {
                "type": ["string", "null"],
                "description": "When currency is'other'Fill in the specific currency name when"
            },
            # Confidence field (helps downstream determine whether manual review is required)
            "confidence": {
                "type": "string",
                "enum": ["high", "medium", "low", "unclear"],
                "description": "Extract confidence"
            },
            # Array field
            "line_items": {
                "type": ["array", "null"],
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "amount":      {"type": "number"}
                    },
                    "required": ["description", "amount"]
                }
            }
        },
        "required": ["invoice_number", "currency", "confidence"]
        # total_amount, currency_detail, line_items are not in required
        # They are nullable and can return null
    }
}
```

### Schema Design Principles Quick Check

| Design Decisions | Principles |
| ------------------ | ----------------------------------------------- |
| **Field may not exist** | Set to `["type", "null"]` nullable to prevent model fabrication |
| **Limited Options** | Use the `enum` constraint to ensure the value is machine-processable |
| **enum cannot be exhausted** | Add `"other"` + `detail` fields |
| **Uncertain case** | Add `"unclear"` to enum, don't let the model guess |
| **Array** | Define `items` schema, including required fields |

### `tool_choice` Selection Guide

```python
# Not sure about the document type, but the output must be structured (most common)
tool_choice = {"type": "any"}

# Certain extraction steps must be run first (forced order)
tool_choice = {"type": "tool", "name": "extract_metadata"}

# Possible direct text answer (not suitable for scenarios that require structure)
tool_choice = {"type": "auto"} # Do not use it in scenarios where structured output is required
```

---

## Task 4.4: Verification, retry and feedback closed loop

### What this tests

Verification and retry tests "knowing when to try again and when to stop." If the error comes from formatting, field misplacement, structural inconsistency, or verifiable semantic conflict, bring the original text, failure results, and specific errors when retrying; if the information is not in the source document at all, continuing to retry will only increase the risk of fabrication. The feedback closed loop requires turning failures and false positives into analyzable data, such as using `detected_pattern` to record trigger patterns, and using `calculated_total`, `stated_total`, and `conflict_detected` to expose semantic conflicts.

### Retry-with-Error-Feedback pattern

When retrying, append **specific verification errors** to the prompt to guide model correction:

```python
def retry_with_feedback(document: str, failed_result: dict, errors: list) -> dict:
    """Retry with a specific error message instead of simply saying "try again"."""
    The extraction result before retry_prompt = f"""has the following verification error. Please correct it and re-extract:

Error list:
{chr(10).join(f"- {e}" for e in errors)}

Previous (wrong) result:
{json.dumps(failed_result, ensure_ascii=False)}

Original document:
{document}

Please correct each of the above errors and try again. """

    return client.messages.create(
        tools=[extraction_tool],
        tool_choice={"type": "any"},
        messages=[{"role": "user", "content": retry_prompt}]
    )
```

### Verification type and processing method

| Verification failure type | Reason | Processing method |
| -------------------- | --------------------------------------------- | -------------------------- |
| JSON syntax error | Rarely occurs (tool use can basically eliminate it) | Try again |
| Semantic error (wrong format) | Date format "2024.01.15" instead of "2024-01-15" | Try again with specific errors |
| Business rule error | The amount is negative and the date is outside the reasonable range | Try again with specific errors |
| Information does not exist in document | The document itself does not have this field | Do not retry, mark as null |
| The information is ambiguous and cannot be confirmed | The numbers in the document cannot be determined whether they are before tax or after tax | Mark as unclear, stop retrying |

### Self-correction verification (Computed vs Stated)

Extract the results by calculating cross-validation:

```python
def validate_invoice(extracted: dict) -> list[str]:
    errors = []

    # 1. Format verification
    if extracted.get("invoice_date"):
        if not re.match(r"\d{4}-\d{2}-\d{2}", str(extracted["invoice_date"])):
            errors.append(
                f"invoice_date format error, should be YYYY-MM-DD,"
                f"Current value:'{extracted['invoice_date']}'"
            )

    # 2. Numeric range verification
    if extracted.get("total_amount") is not None:
        if extracted["total_amount"] <= 0:
            errors.append("total_amount must be greater than 0")

    # 3. Cross-validation (calculated total vs declared total)
    if extracted.get("line_items") and extracted.get("total_amount"):
        computed = sum(item["amount"] for item in extracted["line_items"])
        stated = extracted["total_amount"]
        if abs(computed - stated) > 0.01: # Allow floating point error
            errors.append(
                f"The line item sum ({computed}) does not match the stated total ({stated})."
                f"Difference: {stated - computed}"
            )

    return errors
```

---

## Task 4.5: Batch processing strategy

### What this tests

The entry point for batch-processing questions is SLA. The Message Batches API trades a longer processing window for lower cost. It is suitable for nightly reports, weekly audits, batch classification, and batch test generation; it is not suitable for blocking pre-merge checks or interactive flows where users are waiting. Use `custom_id` to correlate inputs and outputs, rerun only failed items based on failure cause, and tune prompts on a sample set before submitting a large batch. Otherwise one bad batch amplifies both waiting time and cost.

### Message Batches API

| Features | Description |
| -------------------------- | ------------------------------------ |
| **Cost Savings** | About 50% (compared to standard API) |
| **Processing Time** | Up to a 24-hour processing window (asynchronous) |
| **Maximum Batch** | Maximum of 10,000 requests per batch |
| **Applicable scenarios** | Latency-tolerant, non-blocking, offline tasks |
| **Not applicable** | Blocking workflows, multi-turn tool calls, real-time interaction |

### Suitable vs Not Suitable for Batch API

```
Suitable (can be completed within 24 hours):
- Nightly technical debt analysis report
- Batch document classification (large amounts of unstructured documents)
- Large-scale code base scanning
- Historical data annotation

Not suitable (immediate response required):
- Blocking check before PR merge
- Real-time customer service response
- Tasks that require multiple rounds of tool calls in a single request (Batch API cannot execute the tool midway and return the results to the same request to continue inference)
- Sequential processing that relies on the previous result
```

### Batch processing implementation mode

```python
import anthropic

client = anthropic.Anthropic()

# Create batch request
batch_requests = [
    {
        "custom_id": f"doc_{doc_id}", # used to match requests and results
        "params": {
            "model": "claude-haiku-4-5-20251001",
            "max_tokens": 512,
            "messages": [{"role": "user", "content": f"Classification document: {doc_text}"}]
        }
    }
    for doc_id, doc_text in documents.items()
]

# Submit batch
batch = client.messages.batches.create(requests=batch_requests)

# Polling results (can be delayed to 24 hours later)
while True:
    batch_status = client.messages.batches.retrieve(batch.id)
    if batch_status.processing_status == "ended":
        break
    time.sleep(60) # Wait 1 minute before checking again

# Match results by custom_id
for result in client.messages.batches.results(batch.id):
    doc_id = result.custom_id.replace("doc_", "")
    if result.result.type == "succeeded":
        output = result.result.message.content[0].text
```

---

## Task 4.6: Multi-instance review architecture

### What this tests

The test point of multi-instance and multi-pass is "isolating context and splitting attention". When the same model self-checks the code it just generated in the same session, it will inherit the original reasoning path, making it easier to confirm your decision; the independent Claude instance does not generate context, which is more suitable for discovering subtle problems. Large reviews need to be split into file-by-file local analysis and cross-file integrated analysis. The former looks for local bugs, while the latter looks at data flow, interface contracts and cross-module consistency to avoid stuffing a large number of files at once, resulting in missed detections or conflicting conclusions.

### Examples of independent review vs self-assessment

| How | Reliability | Why |
| ------------------------ | ------ | ----------------------------------------------- |
| **Independent Claude instance review** | High | Does not retain the reasoning context during generation, and has no "self-justification" bias |
| **Same session self-check** | Low | The model tends to believe that the content it generates is correct |

### System prompts vs user prompts

```python
# System prompt: role definitions and constraints that are always valid
system = """You are a code security review expert.
Only reported security vulnerabilities, not style or performance issues.
Output JSON array. """

# User prompt: specific tasks
user = f"""Please review the following code changes:
<code>
{diff_content}
</code>"""

# Put "unchanged constraints" in system and "changed content" in user
```

### Review strategy for large code bases

```
Phase 1: File-by-file local analysis (stand-alone task)
→ One independent Claude call per file
→ Only report problems within the file
→ Prevent context overload (especially important when the number of files is large)

Phase 2: Integrated analysis across files (run independently, passing in Phase 1 results)
→ Focus on: inter-file data flow, interface compatibility, cross-module consistency
→ Input: Structured summary of Phase 1 (not full code)
→ Do not mix into the same context as Phase 1
```

### Confidence calibration process

```
1. The model outputs the confidence of each finding (high/medium/low)
        ↓
2. Calibrate the threshold using a labeled validation set (known correct answers)
   (Do not directly use uncalibrated confidence scores; the model's "high" does not mean truly high.)
        ↓
3. Set the shunt threshold (based on calibrated accuracy):
   - High confidence (>0.95 calibration accuracy) → automatic processing
   - Medium confidence (0.7-0.95) → Sampling manual review
   - Low confidence (<0.7) → Full manual review
        ↓
4. Continuous monitoring: regularly sample from "high confidence" cases to verify the true error rate
```

---

## Domain 4 deep dive: prompt engineering, structured output, and quality feedback loops

Domain 4 is not testing "whether you can write prompt", but whether you can treat prompt as an iterable, verifiable, and manageable software component. Prompts in production systems are not just one-off copywriting, they need to have clear standards, examples, structured output, validation, retries, batching strategies, and review structures. Common forms of exam questions are: An automated review or data extraction system presents many false positives, unstable formats, high batch costs, and semantically incorrect structured results, and then asks you to choose the most effective improvement solution. The correct answer is usually not to "make Claude more careful" but to turn the goal into a concrete criterion, schema, few-shot, verification loop, or multi-instance review.
### 1. Explicit criteria beat vague instructions

"Be conservative," "only high confidence," "be careful" — these instructions sound reasonable but are not enforceable for the model. They do not tell the model which categories to report and which categories to skip, what the evidence threshold is, and how to classify the severity levels. Explicit standards are different: report hardcoded credentials, SQL injection, permission bypass; skip formatting styles, local naming preferences, TODO comments; report annotation issues only when the behavior declared by the annotation contradicts the actual behavior of the code. Such rules can directly change the model judgment boundary.

False positive rates impact trust. Developers who see low-value findings continuously will start to ignore all output, including really serious problems. If the question in the exam says that a certain category has a lot of false positives, the best strategy may be to temporarily close this category to restore users' trust in the system first, and then use more specific standards and few-shot optimization. Don't try to solve false positives by saying "only report high-confidence questions" because the model's self-reported confidence is not a stable filter.

Severity levels also need to be explicitly defined. Critical is an exploitable security vulnerability or data loss, High is a clear functional bug or permission bypass, Medium is a failure under boundary conditions, and Low is a maintainability recommendation. It is best to have code examples for each level. The severity level of no example will drift. The same issue will be judged as High today and Medium tomorrow. A structured review system should pursue consistency rather than language fluency every time.

### 2. Few-shot teaches judgment, not just format

The value of few-shot is not just to make the output JSON look the same, but more importantly to show edge cases. For example, what access modes are safe defaults and what are KeyError risks; what test gaps are worth reporting and what are just branch coverage formalisms; what document fields should be extracted and what should return null. A good few-shot allows the model to learn principles and generalize to new situations.

The number of examples doesn’t have to be high, 2-4 high quality examples is usually better than 20 mediocre examples. Each example should target areas where the model tends to hesitate and include positive and negative contrasts. If only positive examples are given, the model will tend to overreport; adding negative examples that "should be skipped" can reduce false positives. The examples must have a consistent format, and the input, inference basis, and output fields must be stable, otherwise the model will learn a confusing format.

In retrieval tasks, few-shot reduces hallucinations. For example, the invoice total may appear at the bottom of the table, in the header, or in handwritten form; the citation may be an inline citation or a bibliography; the method information may be in the methodology section or embedded in a footnote. With examples showing these structural differences, it's easier for models to find real fields rather than making them up under the pressure of required fields.

### 3. Structured output relies on tool use and JSON Schema

Just saying "please output JSON" in the prompt is not reliable enough. Models may output Markdown wrappers, explanatory text, endnotes, and even JSON syntax errors. Tool use combined with JSON Schema is a more reliable approach, because the model generates tool input that is naturally constrained by the schema. `tool_choice: "any"` guarantees the model must call a tool; forced tool selection guarantees a specific tool is called.

But schema solves syntax and shape, not semantics. The invoice line_items all conform to the schema, which does not mean that their sum is equal to total; the date field is a string, which does not mean that it comes from the correct location; the category is an enum, which does not mean that the selection semantics are correct. Therefore, business verification is still required after structured output. In the exam, if the option says "strict schema is used so no verification is required", it is usually wrong. The correct solution is schema + semantic validation.

The schema is designed to allow legal export of the model. Possibly missing fields are set to nullable; fuzzy categories are added with `unclear`; limited but extensible categories are set with `other + detail`; amounts, dates, and units should be given normalization rules in the prompt. Don't make all fields required, otherwise the model will be made up to fit the schema. The goal of a structured extraction system is not to fill the table but to faithfully represent the source document.

### 4. Verification, retry and feedback closed loop

The prerequisite for retries to be effective is that the error can be corrected. The date format is wrong, the field position is wrong, the total calculation is inconsistent, and the output structure does not meet the requirements. These specific errors can be fed back to the model to try again. Feedback must be specific, such as "line_items sum is 128.50 but stated_total is 132.50" rather than "Incorrect result, please try again." The retry prompt should include the original text, failure results, and verification errors so that the model can correct itself.Invalid retries should also be identified. If the information is not in the provided document at all, retrying will only increase the probability of hallucination. The correct thing to do is to return null, unclear, or upgrade manually instead of continuing to try. If the exam question says "the required information is in an external document but is not provided to the model", the best answer is usually not to retry, but to get the missing source or tag that cannot be judged.

The feedback loop also includes analyzing false positive patterns. `detected_pattern` is added to structured finding to count which code patterns are often triggered but dismissed by developers. For example, all "missing annotation" findings are ignored, indicating that this category has low value; a certain type of SQL construct is often misjudged, and the standards or examples need to be adjusted. Without structured schema fields, the team can only read scattered comments and cannot improve the system.

### 5. Batch processing must match SLA

The Message Batches API is suitable for non-blocking, latency-tolerant, large-scale work such as nightly document extraction, weekly audits, and offline test generation. Its strengths are cost and throughput. Its constraints are the 24-hour processing window, no low-latency SLA, and no mid-request multi-turn tool execution. Pre-merge checks, real-time user requests, prerequisite gates, and multi-turn agent loops are not suitable for batch. If an exam question says developers are waiting for merge results, choose the synchronous API; if it says tens of thousands of documents are processed every night, batch is appropriate.

Batch also requires `custom_id` to correlate inputs and outputs. Without `custom_id`, it is difficult to know which document needs to be rerun after a failure. Failure handling should resubmit only failed documents and adjust each input based on the failure cause: chunk documents that exceed the context window, preprocess abnormal formats, and fix schema or few-shot issues when returned structures are wrong. Testing prompts on a sample set before a full run reduces both rework cost and waiting time.

Batch submission frequency should be derived backward from the SLA. If the business requires completion within 30 hours and batch processing can take 24 hours, do not wait until the queue is full before submitting. Submit every 4 hours or on an even shorter window if retries need buffer time. When an exam question gives both an SLA and a batch window, account for submission delay, processing delay, and retry delay.

### 6. Multi-instance and multi-pass review to reduce deviations

Model self-review has natural limitations. After generating code in the same session, the model retains the reasoning context at that time, making it easier to believe that your design is reasonable. The independent review instance does not have these contexts and is more like a new reviewer that can find subtle problems. Extended thinking does not equal independent review, and letting the same model reflect for a long time may only strengthen the null hypothesis.

Large reviews require more passes. File-by-file pass focuses on local bugs, exception handling, and boundary conditions; cross-file pass focuses on data flow, permission propagation, interface contracts, and state consistency. Cramming all the documents and all review dimensions into it at once dilutes the focus. Multiple passes can also reduce conflict findings: local passes are not responsible for cross-document conclusions, and integration passes make global judgments based on structured summaries.

Confidence can be used as a shunt signal, but must be calibrated. The model saying high confidence does not mean that it is actually highly accurate; a labeled verification set needs to be used to estimate the true accuracy of different confidence levels. High confidence can be automatically processed or sampled for review, while medium and low confidence can be subject to manual review. Using uncalibrated confidence levels to automatically turn off manual review is a high-risk practice.

### 7. Diagnosing Domain 4 questions

When encountering Prompt Engineering questions, first determine the type of failure. If there are many false positives, give priority to explicit report/skip standards and few-shot counterexamples; if the format is unstable, give priority to tool use + JSON Schema; if the syntax is correct but the business is wrong, give priority to semantic validation; if correctable errors occur repeatedly, give priority to retry-with-error-feedback; if information is missing, stop retrying and return null/unclear; for large-scale offline tasks, consider batch; for real-time blocking tasks, use synchronization; for poor review quality, use independent instances and multiple passes.

Wrong answers usually have these characteristics: just say "be more careful"; use self-reported confidence as the only filter; all fields are required; consider schema to eliminate all errors; infinite retry for missing information; use batch for pre-merge checks; let the same session self-review; cram multiple file reviews into one call. Correct answers turn language problems into structured, verifiable, and iterable engineering mechanisms.

### 8. Treat Prompt as a testable asset

Production-grade prompts should be tested just like code. Before a prompt goes online, it should have a sample set, expected output, scoring criteria, failure samples, and version records. It is a high-risk approach to only try one or two manual inputs, and then go online if you think the results are good. Especially for tasks such as review, extraction, and classification, small changes in prompts may significantly change the false positive or false negative rate. Many questions in Domain 4 actually test “whether you have eval thinking”.The evaluation set does not have to be large to begin with, but it must cover critical boundaries. Code review prompts should contain real vulnerabilities, acceptable patterns, style noise, and cross-file issues; document extraction prompts should contain standard documents, missing field documents, poorly formatted documents, and conflicting data; structured output prompts should contain unknown categories, other, unclear, and nullable fields. Without boundary samples, it's not known whether prompt only works on ideal input.

Prompt version management is also important. Every time you adjust the report/skip standard, few-shot, schema, and temperature, you should record why you changed it, what failures were solved, and whether new problems were introduced. In an automated system, prompts are part of the behavior; without managing prompt versions, there is no way to explain the sudden increase in false alarms starting on a certain day. Although the exam may not directly mention "version control", whenever the question mentions continuous improvement and dismissal pattern, feedback closed loop and systematic analysis should be thought of.

### 9. End-to-end link of structured output system

A complete data extraction system is not as simple as "Claude outputs JSON". The input layer needs to do document preprocessing and chunking; the prompt layer needs to provide tasks, field definitions, format rules, and few-shots; the tool layer needs to use JSON Schema constraint structures; the verification layer needs to check business semantics outside of the schema; the retry layer needs to provide specific error feedback; the manual review layer needs to handle low confidence and conflicts; and the monitoring layer needs to count field-level accuracy and error patterns. Each Task in Domain 4 corresponds to a part of this link.For example, invoice extraction: the model first identifies the document type and selects the invoice extractor; the schema requires invoice_number, currency, line_items; total_amount allows null; currency has enum and other_detail; prompt indicates that the date is in ISO and the amount is in digital; the validator checks the total of line_items and stated_total; if they are inconsistent, try again and give the specific difference; if they are still inconsistent, output conflict_detected and enter manual review. This process is much more reliable than "please pull invoice JSON".

The code review system is similar: prompt defines report/skip categories, few-shot displays boundaries, and the output schema includes file, line, severity, category, detected_pattern, confidence, and suggested_fix; post-processing removes duplication, and historical findings are passed in to avoid duplicate comments; high false positive categories are adjusted after dismissal pattern analysis. This way the review results can go into CI and PR instead of becoming noise that developers ignore.

### 10. Temperature and certainty

Tasks such as structured extraction, classification, and review often require low temperatures because the goal is to be stable and reproducible. Idea generation, brainstorming, copywriting exploration can increase temperature because the goal is variety. In the exam, if the scene requires "stable production of the same structured output", choose low temperature; if it requires "proposing multiple creative directions", high temperature is reasonable.Temperature is not a universal stabilizer, however. Low temperature does not compensate for fuzzy criteria, nor does it guarantee correct field semantics. It just reduces randomness. True reliability comes from clear standards, schema, few-shots, validation and feedback. Questions that use "set the temperature to 0" as the only solution to fix false positives or semantic errors are usually incomplete.

### 11. Division of labor between Prompt and tools

Prompt is suitable for expressing judgment criteria, context, format preferences and task goals; tools/schema is suitable for mandatory structures; program verification is suitable for deterministic rules; manual review is suitable for high-risk and uncertain scenarios. Putting everything in prompt will make the system fragile; putting everything in schema will not be able to express complex semantics; relying solely on manual review will be too costly. The best answers to Domain 4 usually reflect layering: prompt guidance, tool use constraints, validator checks, retry corrections, and human review to provide the bottom line.

For example, "reducing false positives" cannot rely solely on schema, because false positives are a matter of semantic judgment; "ensuring that JSON is legal" cannot rely solely on prompt, because format constraints are more suitable for tool use; "the total amount must be consistent" cannot rely solely on model awareness, because the program can check with certainty; "source conflict but high business risk" cannot be hard-selected by the model, because manual or coordinator judgment is required. When reviewing, ask: At which level is this problem best solved?

### 12. Eliminating wrong answers on the exam

Among the Domain 4 options, be wary of answers that rely on vague words and few mechanisms. For example, "ask Claude to be more accurate", "tell it to avoid mistakes", "increase confidence threshold" and "retry several times" are not sufficient engineering solutions. Better options will specify mechanisms such as report/skip criteria, few-shot boundaries, tool_choice, JSON Schema, validation errors, custom_id, independent instance, per-file pass, etc.

Another rule of thumb is to see if restrictions are recognized. A good solution will admit that schema does not prevent semantic errors, retry does not solve missing information, batch is not suitable for low latency, multiple instances are more objective than self-review, and confidence needs to be calibrated. Bad solutions often claim that a single solution solves all problems. Official exams are more biased towards production judgment, so options that illustrate boundaries and trade-offs are generally preferred.

### 13. Understand Domain 4 from two typical systems

The first typical system is automated code review. Its failure modes are typically false positives, duplicate comments, inconsistent severity levels, and missed detection of cross-file issues. The solution path is: first define clear report/skip standards and close high false positive categories; use few-shot to show boundaries between what should and shouldn't be reported; output structured findings including location, category, severity, fix suggestions, and `detected_pattern`; do per-file and cross-file passes for large PRs; use an independent instance for review; pass in old findings to avoid duplication. Each step corresponds directly to an official task statement.

The second typical system is document structured extraction. Its failure modes are usually invalid JSON, fabrication when fields are missing, unstable extraction of different formats, inconsistent totals, and high batch processing costs. The solution path is: use tool use and JSON Schema to ensure the structure; set nullable when fields may be missing; add unclear to fuzzy categories; use few-shot to cover multiple layouts; use validators to check amounts, dates, and field positions; retry-with-feedback when errors can be corrected; stop retrying when information is missing; use batch for offline large-scale tasks, and use custom_id to rerun failed items.

These two systems illustrate that the core of Domain 4 is not to write beautiful prompts, but to establish a closed loop around output quality. prompt defines the goal, few-shot defines the boundary, schema defines the shape, validator defines the correctness, retry defines the repair method, batch defines the cost strategy, and multi-pass defines the review architecture. The best answers to questions usually combine these mechanics, rather than picking just one isolated technique.

### 14. How to write maintainable prompts

Maintainable prompts should be clearly partitioned. Differentiate tasks, inputs, report categories, skip categories, severity levels, examples, output formats, constraints with XML or Markdown headers. Inputs that change frequently are placed in user messages, and roles and rules that remain unchanged are placed in system. Keep examples short and precise, and don’t overwhelm the rules with examples. The output format must be consistent with the downstream system, the field names, enumerations, and missing value strategies must be stable.Maintainable prompts also avoid implicit standards. For example, "Report important issues" is weaker than "Report problems that can lead to security vulnerabilities, data loss, and user-visible functional errors"; "Don't be too wordy" is not as good as "Each finding can be up to 80 words and must contain an executable fix suggestion"; "Extract all information" is not as good as "Extract only the fields listed in the schema, and return null if the source document does not have them." The more specific you are, the more you can evaluate and iterate.

### 15. Turn failures into assets

Every time model output fails, it's an opportunity to improve the data. False positives can be turned into negative few-shot; false negatives can be turned into positive few-shot; format errors can be turned into schema or prefill adjustments; semantic errors can be turned into validators; manual correction can be entered into the calibration set; batch failures can be classified by custom_id. Don’t just fix the current output, but precipitate the failure mode into a test sample. In this way, the prompt will become more and more stable, instead of relying on manual on-the-spot repairs every time.

If the exam asks "How to continuously reduce false positives" or "How to analyze the findings of developer dismissals", the answer should include structured fields and feedback analysis, rather than just "Adjust prompts". Continuous improvement requires aggregable data. `detected_pattern`, category, severity, and dismiss_reason are all keys to turning the prompt project into a project.

### 16. Three distinct quality layers in model output

When evaluating Claude output, it can be broken down into format quality, semantic quality, and business quality. Format quality refers to whether the JSON is legal, whether the fields are complete, and whether the enumeration is correct; semantic quality refers to whether the fields come from the correct text, whether the classification meets the meaning, and whether the finding is really a problem; business quality refers to whether the result is valuable to the user or process, such as whether the review opinions are executable, whether the extraction results can enter the downstream system, and whether low-confidence fields enter manual review. Different quality levels must be guaranteed by different mechanisms.

Format quality mainly relies on tool use, JSON Schema, prefill, stop sequence or output parsing; semantic quality mainly relies on explicit criteria, few-shot, verification rules and retry feedback; business quality mainly relies on product rules, manual review, confidence calibration, false positive monitoring and user feedback. In the exam, if the question only says "JSON is often bad", don't prioritize the few-shot discussion; if the question says "JSON is legal but the amount is wrong", don't think that the schema has been solved; if the question says "Developers don't trust the review results", focus on the false positive category and trust restoration.

### 17. The ultimate goal of the Prompt project

Prompt engineering is not to make a single answer look smarter, but to make the system stable in the face of large amounts of input, edge scenarios, multi-person collaboration, and automated execution. A truly mature prompt will reduce the explanation space, clarify uncertainty, give the model a legitimate "unable to judge" exit, and turn failure into improveable data. It is not a piece of copy written once, but a system asset that evolves with evaluation sets, schemas, validators, and monitoring.

### 18. Final judgment principles in the examination room

If an option simply changes the wording without changing the information structure, constraint mechanism, or feedback path, it is usually not the best answer. For example, "Please be more accurate", "Please be more conservative" and "Please check again" are all weak interventions. Strong intervention will change the system behavior: add report/skip standards, add few-shot counterexamples, force tool_use, adjust schema nullable, add validators, feed errors back to retry, rerun failed batch processing according to custom_id, or use independent instances to review.

It also depends on whether the plan fits the task's cost and latency goals. Real-time tasks require low latency and should not move to batch purely to save money; offline tasks optimize for throughput and can accept a 24-hour window; high-risk output requires manual review, while low-risk and high-confidence output can be automated. High-scoring Domain 4 answers usually balance quality, cost, latency, and risk.

After learning this part thoroughly, you will find that prompt engineering is not a metaphysics. It has input design, example design, output constraints, verification feedback, batch execution and review architecture. What the exam really wants to confirm is whether you can turn the model output from a disposable text into a reliable production process.

I will treat prompt as a piece of code that will cause bugs: there are test samples, failure records, and reasons for rollback.

---

## Pre-exam checklist

1. **Reduce False Positives** → Explicitly list reported/skipped categories, don’t rely on vague “confidence filtering”
2. **Unstable format** → Few-shot examples (more effective than long descriptions)
3. **Few-shot design** → For boundary scenes, compare positive and negative, 2-4 is appropriate
4. **Guaranteed tool call** → `tool_choice: "any"` (not `"auto"`)
5. **temperature=0** → structured extraction and classification tasks; high temperature → creative generation
6. **nullable field** → Set to `["type", "null"]` to prevent model fabrication
7. **Retry valid** → Carrying specific verification errors (wrong date format, negative amount, etc.)
8. **Retry invalid** → The information itself is not in the document → Mark as null, stop retrying
9. **Batch processing** → Roughly 50% cost discount, up to a 24-hour processing window; **not suitable** for multi-turn tool calls, blocking workflows, or real-time tasks
10. **Independent Review** → More credible than same-session self-review; large PRs first file by file and then across files
11. **XML tags** → clearly separate different parts of the prompt (constraints/examples/formats)
12. **Prefill** → Prefill assistant turn guide specific format
