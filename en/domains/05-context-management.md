# Domain 5: Context Management & Reliability

> **Weight: 15%**
> Official documentation: [Context Windows](https://docs.anthropic.com/en/docs/build-with-claude/context-windows) | [Long Context](https://docs.anthropic.com/en/docs/build-with-claude/long-context-tips)

---

## Task Statement Coverage

| Task | Topic |
| ---- | ------------------------------------ |
| 5.1 | Managing context to preserve key information during long interactions |
| 5.2 | Designing effective escalation and disambiguation patterns |
| 5.3 | Implementing error propagation strategies in multi-agent systems |
| 5.4 | Effectively manage context in large code base exploration |
| 5.5 | Designing manual review workflow and confidence calibration |
| 5.6 | Preserving source chains and handling uncertainty in multi-source synthesis |

---

### Task Statement 5.1: Manage conversation context to preserve critical information across long interactions

#### Knowledge of:

- Progressive summarization risks: condensing numerical values, percentages, dates, and customer-stated expectations into vague summaries
  - Progressive summarization compresses text, but key facts (order number, amount, deadline, user's clear expectations) should not be replaced by vague words such as "about" and "a few days ago". Once the natural language summary covers these details, the model in subsequent rounds cannot recover the original values ​​and may make incorrect judgments. This type of precise information should go into a structured case facts block, kept separate from the summary.
- The "lost in the middle" effect: models reliably process information at the beginning and end of long inputs but may omit findings from middle sections
  - "Lost in the Middle" is a known characteristic of long context: the model handles the beginning and end of the input more reliably, and the content of the middle paragraph has a higher probability of being missed. When synthesizing multiple sources, key claim summaries and evidence sources should be placed at the beginning of the merged input, with detailed file content grouped into sections at the end. For multiple rounds of conversations, important constraints can be injected continuously in the system prompt, rather than appearing once and then entering the middle of the historical message.
- How tool results accumulate in context and consume tokens disproportionately to their relevance (e.g., 40+ fields per order lookup when only 5 are relevant)
  - Tool calls are one of the main sources of context growth. A customer information query may return 40+ fields, but refund judgment only requires order number, amount, date, return eligibility and current status. Keeping all fields in context will dilute relevant information in subsequent rounds and increase the probability of model errors. Tool results should be filtered by code before entering the messages list, retaining only fields related to the current task.
- The importance of passing complete conversation history in subsequent API requests to maintain conversational coherence
  - The Anthropic API is stateless: each call to `messages.create` is an independent request, and the server does not store any session state. When there are multiple rounds of dialogue, all relevant historical messages must be explicitly passed into the messages array by the client at each call. If history is omitted, the model has no way of knowing the information the user has already provided or previous decisions, and will start over as if it were a completely new conversation.

#### Skills in:

- Extracting transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block included in each prompt, outside summarized history
  - Case facts block is a structured injection mechanism: key facts are passed in in JSON or XML format at the system prompt or at the beginning of the conversation on every API call, and will not be lost or overwritten by the summary regardless of the length of the session. Natural language summaries are suitable for describing conversational processes, but are not suitable for storing precise values ​​and states. Managing the two separately ensures that the model is still based on the same precise numerical decisions in round 30 as it was in round 1.
- Extracting and persisting structured issue data (order IDs, amounts, statuses) into a separate context layer for multi-issue sessions
  - Users may be dealing with multiple independent issues (refund issues + billing disputes) in the same session, each with its own order number, amount, timeline, and processing status. By mixing the facts for all questions into a history of natural language, the model might apply the amount of money in one question to the judgment of another. Maintaining separate structured status objects for each issue and referencing them separately in prompts is a standard pattern for solving the clutter of multi-issue sessions.
- Trimming verbose tool outputs to only relevant fields before they accumulate in context (e.g., keeping only return-relevant fields from order lookups)
  - Field filtering should be done at the code level instead of leaving it to the model to "ignore irrelevant fields by itself". The filtering criteria are determined by the current task: the refund task retains the order ID, amount, date, and refund eligibility; the bill dispute task retains the bill number, disputed item, and payment status. This active clipping can control the context growth rate and maintain the signal-to-noise ratio of the context during multiple rounds of interaction.
- Placing key findings summaries at the beginning of aggregated inputs and organizing detailed results with explicit section headers to mitigate position effects
  - Structure is crucial when the coordinator combines the output of multiple sub-agents into a single prompt. Start with an executive summary or a list of key claims to allow the model to establish a global understanding before going into details; then organize detailed evidence into sections by source, with clear paragraph titles; the most important constraints or judgment criteria can be reiterated at the end. This structure preserves key findings better than piecing everything together linearly.
- Requiring subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis
  - If the sub-agent only returns "35% of users in this market are affected" without providing the data source, collection time, geographical scope and measurement method, the coordinator cannot judge the applicable scope of this number when synthesizing, nor can it identify potential conflicts with other sources. The structured output should include fields such as claim, evidence, source, publication_date, methodology, etc., so that the downstream agent has enough context to use each piece of data correctly.
- Modifying upstream agents to return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when downstream agents have limited context budgets
  - In a multi-agent pipeline, the lengthy inference process of the upstream agent will quickly consume the limited context budget of the downstream agent. Upstream should return refined structured output: key conclusions, evidence citations, fielded metadata, confidence, and gap descriptions; the inference process can be logged, but should not be passed downstream as the main output. This is a key design principle for multi-agent context budget management.
- Using overlapping chunks when splitting long documents to prevent information loss at chunk boundaries
  - When a document is too large to fit in context and must be split into chunks, strict sequential splits create hard boundaries where sentences and logical units spanning the boundary lose their surrounding context. Overlapping chunks solve this by including a shared region (typically 10–20% of chunk size) at the end of one chunk and the beginning of the next. This ensures the model can correctly interpret content near each boundary. When the exam describes "information at chunk boundaries being missed or misinterpreted," overlapping chunks is the correct fix.
- Applying hierarchical summarization to compress large document collections while preserving structure and source traceability
  - Single-pass summarization of a large number of documents discards too much detail. Hierarchical summarization processes in two stages: first, summarize each section or document independently (preserving key claims, evidence, and source references); then, synthesize the section-level summaries into a final high-level output. This achieves high compression while maintaining more structure than a flat summary and keeps conclusions anchored to specific sources. When the exam asks about summarizing large corpora or multi-document collections efficiently, hierarchical summarization is preferred over single-pass compression.

### Task Statement 5.2: Design effective escalation and ambiguity resolution patterns

#### Knowledge of:

- Appropriate escalation triggers: customer requests for a human, policy exceptions/gaps (not just complex cases), and inability to make meaningful progress
  - There are two problems with using "problem complexity" as an upgrade trigger: there is no objective definition of complexity, and in terms of user experience, it is likely that users will wait longer for complex problems before upgrading. The actual upgrade conditions should be programmable judgment rules: the user explicitly requests a human agent, the current request falls outside policy coverage, and the system still cannot advance after multiple attempts. These conditions are clear, testable, and can be demonstrated in few-shot examples.
- The distinction between escalating immediately when a customer explicitly demands it versus offering to resolve when the issue is straightforward
  - The difference between the two situations lies in the clear expression of the user: using statements such as "I want to speak to a real person" and "Please transfer to human customer service" are explicit requests and must be escalated immediately. No matter how simple the problem is, you cannot continue to try to solve it. "I'm very disappointed" and "This is too much trouble" are emotional expressions and are not equal to manual requests. At this time, you can first acknowledge the emotion and provide a solution; if the user explicitly requests a human agent again later, escalate immediately.
- Why sentiment-based escalation and self-reported confidence scores are unreliable proxies for actual case complexity
  - Misjudgment in sentiment analysis - the same expression has different emotional interpretations in different cultures and contexts, and negative emotions do not mean the need for manual intervention. The model's self-assessed confidence ("I am 80% confident about this answer") is not calibrated and cannot be used as a basis for safety thresholds. A reliable upgrade mechanism should be rule-driven: detect specific trigger words ("artificial", "real person"), detect policy gaps, and detect process stagnation (trying the same problem N times without progress).
- How multiple customer matches require clarification (requesting additional identifiers) rather than heuristic selection
  - When the tool returns multiple potentially matching users or orders, it is an error to select one, regardless of which one is more recent, which amount is closer, or which name is more common. Wrong choices can result in exposing information to the wrong user (privacy violation) or operating the wrong account. The only safe way to handle this is to explicitly request additional identifiers from the user (e.g. email suffix, order number, billing address zip code) to confirm identity before continuing.

#### Skills in:

- Adding explicit escalation criteria with few-shot examples to the system prompt demonstrating when to escalate versus resolve autonomously
  - Even if there are literal rules in the system prompt, the model may still make inconsistent judgments when handling edge cases. Adding a few-shot example to the system prompt to show the contrast between "the user explicitly requests a human → escalate immediately and stop trying to resolve" and "the user expressed dissatisfaction but did not request manual work → acknowledge the emotion and provide a solution" can anchor the behavioral boundaries of the model in real conversations. Few-shots serve the same purpose in escalating rules as they do in code review categories: filling in the boundaries of judgment that literal rules cannot express.
- Honoring explicit customer requests for human agents immediately without first attempting investigation
  - Continuing to invoke a tool or attempt to resolve a problem after receiving an explicit human request is a disregard for user intent and can further irritate users and reduce trust in the system. The correct behavior is to immediately stop the tool call sequence, generate an escalation summary (containing the collected case facts and issue status), and hand off to a human agent. The system prompt should make this stopping condition clear and show a few-shot of the correct response pattern when triggered.
- Acknowledging frustration while offering resolution when the issue is within the agent's capability, escalating only if the customer reiterates their preference
  - Escalating based solely on sentiment scores unnecessarily reduces the auto-resolve rate. When a user expresses frustration but the problem is within the capabilities of the system, the correct approach is to first acknowledge the emotion and then provide a solution to avoid repeatedly asking the user to explain a situation that has already been explained. Escalate only if the user explicitly asks for a human agent again after that. This two-step model takes into account both user satisfaction and automation efficiency.
- Escalating when policy is ambiguous or silent on the customer's specific request (e.g., competitor price matching when policy only addresses own-site adjustments)
  - When a customer's request falls outside the literal scope of the system's existing policy (for example, the policy only says "price adjustment on this site" but the customer requires competitor price matching), the model should not be allowed to infer whether the extension is "reasonable" and then handle it on its own. Letting the model interpret policy boundaries on its own creates an inconsistent customer experience and may violate business rules. The correct approach is to detect policy gaps and escalate them, allowing authorized personnel to make exceptions.
- Instructing the agent to ask for additional identifiers when tool results return multiple matches, rather than selecting based on heuristics
  - In customer support scenarios, performing operations under the wrong identity (such as refunding money to the wrong account, viewing other people's order details) may violate privacy regulations and also lead to serious customer disputes. The safety principle of ambiguity resolution is "It is better to ask once more than to guess once." The requested additional identifier should be information that the user can immediately provide (such as email, billing zip code, last four digits of the order number) rather than requiring the user to enter their entire account password or other highly sensitive information.

### Task Statement 5.3: Implement error propagation strategies across multi-agent systems

#### Knowledge of:

- Structured error context (failure type, attempted query, partial results, alternative approaches) as enabling intelligent coordinator recovery decisions
  - Structured error messages are decision input for multi-agent systems, not just logs. `failure_type` distinguishes between timeouts, permission denials, parsing errors and truly empty results; `attempted` records executed queries to prevent the coordinator from repeating the same failed operation; `partial_results` allows the coordinator to judge whether it can continue with incomplete data; `alternatives` gives alternative query terms or sources so that the coordinator has specific solutions to route. These four fields are the minimum feasible structured error format.
- The distinction between access failures (timeouts needing retry decisions) and valid empty results (successful queries with no matches)
  - Access failure (such as timeout, permission error, service unreachable) means that the query was not successfully executed and the result is unknown; an empty result means that the query was successfully executed, but there are no matches in the database. By indicating an empty result as a failure, the coordinator will trigger unnecessary retries, wasting time and resources. By representing access failures as empty results, the coordinator will mistakenly believe that the subject has no data, resulting in incorrect gap descriptions in the composite report. The two should be clearly distinguished in the status field (`success: true, count: 0` vs `status: "timeout"`).
- Why generic error statuses ("search unavailable") hide valuable context from the coordinator
  - Returning a generic error like `"operation failed"` or `"search unavailable"` is equivalent to collapsing all failure types into an indistinguishable signal. The coordinator does not know whether this is a timeout (you can retry), a permissions issue (retry is invalid and you need to change tools), or a parsing failure (prompt needs to be changed). Without a specific failure type, the coordinator can only choose to give up and cannot make intelligent recovery decisions. The cost of structural errors is very low, but the improvement in system reliability is great.
- Why silently suppressing errors (returning empty results as success) or terminating entire workflows on single failures are both anti-patterns
  - "Silently returning empty results as success" can make the coordinator think that the data does not exist, resulting in false gaps or false negatives in the synthetic report. "Single point of failure global termination" will discard the work completed by other sub-agents and force the execution to be repeated. The correct pattern of structured propagation is: each sub-agent reports its own success, partial success or failure status, and the coordinator combines these statuses to decide how to proceed - which one to retry, which one to continue, and which gap to mark.

#### Skills in:

- Returning structured error context including failure type, what was attempted, partial results, and potential alternatives to enable coordinator recovery
  - The diagnosis sheet analogy is very specific: a doctor's diagnosis sheet will record symptoms, checked items, test results and recommended next steps, rather than just "sick". The failure report of the sub-agent should also include: what was done (attempted_queries), what was obtained (partial_results), why it failed (failure_type: timeout/access_denied/parse_error), whether it can be retried (recoverability), and what else can be done (alternatives). These fields allow the coordinator to make specific decisions instead of just giving up.
- Distinguishing access failures from valid empty results in error reporting so the coordinator can make appropriate decisions
  - The recommended return structures are two types: `{"status": "success", "count": 0, "results": []}` is returned on success, indicating that the query is successful but there is no match; `{"status": "failed", "failure_type": "timeout", "recoverability": "retryable"}` is returned on failure, indicating that there is a problem with the execution itself. These two formats allow coordinators to differentiate at the code level without the need to parse text descriptions. The `recoverability` field directly tells the coordinator whether this failure is worth retrying.
- Having subagents implement local recovery for transient failures and only propagate errors they cannot resolve, including what was attempted and partial results
  - The child agent should have its own local error handling layer: if the network times out, it can try 1-2 retries locally. If the parsing fails, it can try different parsing strategies, and try its best to recover itself before reporting to the coordinator. Only when local recovery is not possible, structured failure reports are sent to the coordinator. This layering reduces the number of failure signals that the coordinator needs to handle and reduces unnecessary global retries triggered by transient network jitter.
- Structuring synthesis output with coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources
  - Synthetic reporting should not pretend that all sources are available and of equal quality. When some sub-agents fail or return insufficient coverage, the corresponding conclusion should be marked with `coverage: "partial"` or with the annotation "Source X is not available, this conclusion is only based on sources Y and Z". This allows readers to judge which conclusions are well supported and which require further verification. Presenting all conclusions with the same level of confidence is a dishonest synthesis of multiple sources.

### Task Statement 5.4: Manage context effectively in large codebase exploration

#### Knowledge of:

- Context degradation in extended sessions: models start giving inconsistent answers and referencing "typical patterns" rather than specific classes discovered earlier
  - Context degradation does not appear suddenly, but gradually: the model starts to replace the specific file path found before with "This kind of project usually..."; it starts to say "the general auth module will..." instead of quoting `src/auth/jwt_validator.py` line 45. This shift indicates that early findings have been compressed or lost by the current context. When this signal is detected, scratchpad contents or staged summaries should be injected to replace missing specific findings.
- The role of scratchpad files for persisting key findings across context boundaries
  - Scratchpad files (such as `.claude/scratchpad/findings.json`) are external memories persisted to the file system and are not restricted by the single-session context window. The model cannot "remember" files discovered before the Nth session, but can read from scratchpad. Key findings, entry points, dependencies, backlog items, and risk points should be written to the scratchpad immediately upon discovery, with subsequent tasks reading it first rather than relying on the model's context memory.
- Subagent delegation for isolating verbose exploration output while the main agent coordinates high-level understanding
  - If the main agent is directly responsible for reading a large number of documents one by one, the context will quickly be filled with details, and the main agent's own judgment ability will be reduced due to dilution of attention. The correct division of labor is: the main agent maintains the architecture map (entry points, core modules, dependencies) and task lists, the sub-agent is responsible for answering specific questions ("Find all files that call the payment interface") and return structured summaries, and the main agent merges global understanding based on the summaries. In this way, the main agent always remains at a high level and is not overwhelmed by details.
- Structured state persistence for crash recovery: each agent exports state to a known location, and the coordinator loads a manifest on resume
  - Manifest is the basis for recoverability of long-term tasks. It records completed modules (`completed_modules`), pending modules (`pending_modules`), key discovery file paths (`findings_file`), and current assumptions. After a task crashes or is interrupted, the coordinator loads the manifest, injects state into a new agent prompt, and continues from the pending module instead of rescanning the entire code base from scratch. Long tasks without manifests can only be redone after a crash, which wastes a lot of time.

#### Skills in:

- Spawning subagents to investigate specific questions (e.g., "find all test files," "trace refund flow dependencies") while the main agent preserves high-level coordination
  - The vaguer the sub-agent task description, the wider the scope it explores, the more verbose the content returned, and the lower the value to the main agent. Instructions such as "Analyze the code base" will cause the sub-agent to read a large number of files and return a general summary; "Find all files and calling methods that call `process_refund()`" are the specific executable tasks. Clear boundaries can also prevent sub-agents from over-investing in exploration depth and ensure that each sub-task is completed within a reasonable token budget.
- Having agents maintain scratchpad files recording key findings, referring them for subsequent questions to counteract context degradation
  - The practical operation of this principle is: whenever a child agent discovers an important fact (such as discovering that the auth module is missing a JWT expiration check on line 45), immediately append it to the scratchpad file in a structured format, instead of just mentioning it in the text output of the current session. Another subsequent sub-agent or the restored main agent obtains known facts by reading the scratchpad file instead of searching the chat history. Discovery of external persistence is not restricted by the session context window.
- Summarizing key findings from one exploration phase before spawning sub-agents for the next phase, injecting summaries into initial context
  - The principle of context transfer between stages is "compression but fidelity": neither complete exploration output (too long) nor fuzzy natural language summary (key details are lost), but a structured summary: completed modules, key findings (files, functions, line numbers), discovered problems, unresolved problems, and suggestions for the next stage. This summary serves as the initial context for the sub-agent in the next stage, allowing it to continue based on what is known rather than exploring from scratch.
- Designing crash recovery using structured agent state exports (manifests) that the coordinator loads on resume and injects into agent prompts
  - When implementing recovery logic in the code, the coordinator should first check whether the manifest file exists, and if it exists, load the `completed_modules`, `pending_modules` and `findings_file` paths. After injecting the state, only call the subagent for `pending_modules`, and pass in the content of `findings_file` as a known context. If you fully re-explore without checking the manifest, you will repeat the work you have already done, and you may reach contradictory conclusions when new discoveries are made.
- Using /compact to reduce context usage during extended exploration sessions when context fills with verbose discovery output
  - `/compact` is the context compression command in Claude Code. It is used to compress existing content when the context is close to full load, retain key conclusions and status, and discard redundant exploration process and intermediate step output. In large code base exploration, the detailed output of sub-agents often contains a large number of intermediate steps, and this information is no longer needed after confirming the conclusion. Proactive use of `/compact` prevents premature exhaustion of the context window and allows long exploration sessions to continue.

### Task Statement 5.5: Design human review workflows and confidence calibration

#### Knowledge of:

- The risk that aggregate accuracy metrics (e.g., 97% overall) may mask poor performance on specific document types or fields
  - The extraction system is 99% accurate on standard documents and 68% accurate on handwritten receipts. An overall accuracy rate of 97% would still seem excellent, but the error rate for handwritten receipts is enough to cause real business problems. Hierarchical assessment requires calculating accuracy separately by document type (standard invoice, handwritten receipt, scan, multilingual) and field (total amount, date, tax ID, line item) to identify sub-scores of substandard performance rather than masking them with an overall score.
- Stratified random sampling for measuring error rates in high-confidence extractions and detecting novel error patterns
  - The purpose of continuous sampling is not only to count the proportion of known errors, but also to promptly discover new error patterns that appear after the system goes online. Document format changes, new vendor templates, changes in OCR quality, or prompt drift can cause an otherwise high-confidence field to start showing errors. Stratified random sampling (samples are drawn separately by document type, field, and confidence level) can ensure that the degradation of specific segmentation scenarios will not be missed due to sample bias.
- Field-level confidence scores calibrated using labeled validation sets for routing review attention
  - The specific operation of calibration is: on the validation set with manually labeled correct answers, what are the true accuracy rates in the fields where the statistical model outputs high/medium/low confidence. If the true accuracy corresponding to high confidence is 92%, then when using this level to automatically pass, the expected error rate is 8%. Whether it is acceptable depends on the business risk. Uncalibrated confidence cannot set reasonable thresholds because the range of confidence output varies from model to model, task to task, and field to field.
- The importance of validating accuracy by document type and field segment before automating high-confidence extractions
  - The overall high confidence accuracy reaches the threshold and cannot be used as a basis for full automation. It is necessary to confirm that the confidence-accuracy calibration results meet the set thresholds for each document type and key field. Only after each segmentation dimension has been verified is it safe to turn on automation for high-confidence results for that dimension. Segmentation scenarios that are not fully verified should continue to be subject to manual review to avoid blindly expanding the scope of automation.

#### Skills in:

- Implementing stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection
  - After the system goes online, document distribution will change over time, and new formats, new suppliers, and new data quality issues will appear. If sampling is stopped after automation goes live, drift can accumulate quietly and not be noticed until the error rate increases significantly. The frequency of ongoing spot checks can be lower than during the initial calibration phase, but cannot be zero. New error patterns discovered during sampling should be flowed back into the calibration set as new annotated samples, keeping the calibration data consistent with the current distribution.
- Analyzing accuracy by document type and field to verify consistent performance across all segments before reducing human review
  - The dimension of matrix analysis is document type × field, such as [invoice, tax number], [receipt, total amount], [contract, validity period] and their respective accuracy rates. This matrix can clearly show which dimensions have reached the automation requirements and which still require manual labor. Looking only at the overall score may result in an overall reduction in review when some key fields (such as high-risk amounts, dates) are not up to standard. Each adjustment to the manual review ratio should be based on the latest data from this matrix.
- Having models output field-level confidence scores, then calibrating review thresholds using labeled validation sets
  - Overall confidence ("This document has an extraction confidence of 0.88") doesn't tell you which fields are reliable. Field-level confidence (`invoice_number: high, total_amount: medium, tax_id: low`) can be accurately routed to the appropriate review strategy: if the total amount is medium with confidence and the amount is large, it will be manually reviewed, and if the invoice number is high with confidence and the type has been calibrated, it will be automatically passed. The setting of the threshold requires using real annotated data to estimate the actual accuracy of each gear, rather than directly using the confidence score output by the model.
- Routing extractions with low model confidence or ambiguous/contradictory source documents to human review, prioritizing limited reviewer capacity
  - The manual review queue should be sorted by priority, not FIFO. The highest priorities are: fields with low model confidence, numerical conflicts between multiple sources, fields related to high-risk operations (such as large amounts, contract validity period, permission control), and records with ambiguities or abnormal formats identified by the system. Focusing limited auditor time on the highest risk and most uncertain records maximizes error detection rates at the same labor cost.

### Task Statement 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis

#### Knowledge of:

- How source attribution is lost during summarization steps when findings are compressed without preserving claim-source mappings
  - Source loss during the summarization process is a typical failure mode of multi-source systems: the sub-agent summarizes multiple documents into a paragraph, but the summary does not include "which paragraph of which document this number comes from". The synthetic agent receiving this summary has no way of knowing the source, and the conclusions in the final report cannot be verified. Structured snippets should always preserve claim-to-source mapping, even if text compression is required, preserving the source URL, document name, relevant excerpts, and publication time.
- The importance of structured claim-source mappings that the synthesis agent must preserve and merge when combining findings
  - The correct operation of the synthetic agent is not to restate the existing conclusions in its own language, but to merge and deduplicate the claim-source mappings from multiple sub-agents. If two sub-agents report the same claim but from different sources, the composite result should include both sources, rather than just writing the conclusion once. Rephrasing unsourced conclusions produces "fluent but unverifiable" reports that are extremely unreliable in academic, legal, or business scenarios.
- How to handle conflicting statistics from credible sources: annotating conflicts with source attribution rather than arbitrarily selecting one value
  - Numerical differences between trusted sources may have multiple explanations: different measurement times, different sample ranges, different measurement methods, different variable definitions. When there is insufficient information to determine which is more accurate, the report should juxtapose the two values, their respective sources, dates and methods, and describe possible explanations. Allowing the model to autonomously choose a value that "seems more authoritative" would mistakenly leave the responsibility for methodological judgment to the model.
- Temporal data: requiring publication/collection dates in structured outputs to prevent temporal differences from being misinterpreted as contradictions
  - The same indicator may obtain different results when measured at different time points. This is a normal timing change and not a source conflict. But if neither source has a timestamp, the coordinator cannot tell whether the difference is due to timing or different methods. Enforcing the `publication_date` and `data_collection_date` fields in structured output allows compositing agents to correctly annotate the timeliness of data, and also to prioritize more recent data or explicitly note timing differences when comparing multiple sources.

#### Skills in:

- Requiring subagents to output structured claim-source mappings (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis
  - This is a system design constraint, not just a suggestion: in the output schema of the child agent, each claim object must contain the `claim`, `source_url`, `document_name`, `excerpt`, `publication_date` fields, and the prompts of the coordinator and synthetic agents must make it clear that "the source field must not be omitted". This constraint needs to be strengthened both at the JSON Schema level (required fields) and at the prompt level (few-shot displays the complete mapping structure).
- Structuring reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context
  - "AI improves efficiency" may be a common finding of multiple studies (established), or there may be only one internal study with data (limitations need to be noted), or different studies may have reached opposite conclusions (disputed). The report should clearly distinguish between these three categories of situations rather than presenting all conclusions in the same tone. "Disputed" should also include the source and method differences of the dispute, so that readers can understand the nature of the dispute instead of just knowing "different opinions".
- Completing document analysis with conflicting values included and explicitly annotated, letting the coordinator decide how to reconcile before passing to synthesis
  - Analytical agents process individual documents or data sources without a global view of which source is more trustworthy or which method is more authoritative. When it finds conflicting values ​​within the document (such as two tax amounts that are inconsistent) or differences from known data, both values ​​should be reported to the coordinator, along with their respective locations, context, and possible explanations. The coordinator or final composition agent makes a decision with global information in hand, or presents the conflict transparently to the reader.
- Requiring subagents to include publication or data collection dates in structured outputs to enable correct temporal interpretation
  - There are two types of timestamps: publication date (when the document was published) and data collection date (when the research data was collected). The two may be months or even years apart, and both are important metadata. In the output schema of the sub-agent, these two fields should be set to required (or explicitly allow null and mark the reason). You cannot rely solely on document text descriptions to convey timeliness information. When downstream agents compare multiple sources, the first step is often to align the time dimension, which cannot be done without timestamps.
- Rendering different content types appropriately in synthesis outputs—financial data as tables, news as prose, technical findings as structured lists—rather than converting everything to a uniform format
  - Financial data should be presented in tables (to facilitate alignment and comparison), news events or narrative content should be presented in prose (to facilitate understanding of chronology and cause-and-effect relationships), technical findings should be presented in structured lists (for easy reference and reference), and controversial conclusions should be presented in parallel blocks (with supporters, opponents, and uncertainties clearly marked). Forcing all content into the same format (such as full-text prose or full lists) can reduce the readability of certain content types and sometimes obscure important relationships in the data structure.

---

## Task 5.1: Context management in long interactions

### Key failure modes to understand

1. **"Lost in the middle" effect**: The model handles content at the beginning and end of long inputs more reliably; material in the middle is more likely to be missed.
2. **Tool result accumulation**: Each tool call appends results to the context, consuming tokens even when most fields aren’t relevant to the current task.
3. **Progressive summary risk**: Summarization can silently discard precise values, dates, and explicit user commitments — the details that matter most.

### Case facts block

Extract key facts into a structured persistent block injected at every turn:

```python
case_facts = {
    "customer_id": "CUST-12345",
    "account_status": "active",
    "order_id": "ORD-67890",
    "order_amount": 299.99,
    "order_date": "2024-01-15",
    "issue_type": "missing_item",
    "previous_contacts": 2
}
# Inject at the start of each API call — key facts survive regardless of summarization
```

### Trimming tool results

```python
# Unfiltered: all 40+ fields enter the context
full_result = get_customer(customer_id)

# Filtered: only what the current task actually needs
relevant_fields = {
    "customer_id": full_result["id"],
    "name": full_result["name"],
    "status": full_result["account_status"],
    "refund_eligible": full_result["refund_eligible"]
    # Dropped: address, transaction history, marketing preferences, etc.
}
```

### Context placement strategy

- **Critical information first**: place it at the beginning of the system prompt or message, not buried in the middle
- **Structured sectioning**: separate content with clear headings to counter position effects
- **Separate issue state**: for multi-issue sessions, maintain structured state objects per issue rather than mixing everything together

### What this tests

These questions aren’t about shortening conversations — they’re about compressing context without losing the facts that affect decisions. Exam scenarios typically involve long customer service sessions, multiple orders, multiple tool results, or multiple sub-agent outputs. The correct answer involves a structured fact layer, field trimming, and surfacing key findings at the top, not simply instructing the model to "summarize."

In practice, context should be split into three layers:

1. **Persistent fact layer**: order numbers, amounts, dates, statuses, explicit user expectations, and policy judgments.
2. **Working summary layer**: current issue progress, steps already attempted, and next actions.
3. **Raw evidence layer**: tool outputs, citations, and sub-agent metadata available for review when needed.

This design ensures the model is still reasoning against the same precise facts in turn 30 as it was in turn 1, rather than working from a vague impression left behind by earlier summaries.

### Long-document chunking: overlapping chunks

When a document is too large to fit in context, splitting it into sequential non-overlapping chunks creates a boundary problem: sentences and paragraphs that span a chunk boundary lose their context and may be misinterpreted or missed entirely.

**Overlapping Chunks** solve this by including a small overlap region (typically 10–20% of the chunk size) at both the start and end of each chunk:

```
Chunk 1: [Content A | overlap region]
Chunk 2: [overlap region | Content B | overlap region]
Chunk 3: [overlap region | Content C]
```

This ensures that no sentence or logical unit is split across a hard boundary. When the exam asks about long-document processing or mentions that "information near chunk boundaries is being missed," overlapping chunks is the correct answer — not simply increasing chunk size or reducing overlap to zero.

### Hierarchical summarization for long documents

For very long documents or large numbers of documents, a single-pass summary may lose too much detail. **Hierarchical summarization** processes in two stages:

1. **Section summaries**: summarize each section or document independently, preserving key claims, evidence, and source metadata
2. **Summary of summaries**: combine the section-level summaries into a final high-level synthesis

This achieves high compression ratios while preserving more structure than a flat single-pass summary. It also keeps the summary of each section anchored to the source, making the final synthesis more traceable. When the exam asks about summarizing very large corpora or multi-document collections, hierarchical summarization is preferred over direct single-pass compression.

---

## Task 5.2: Escalation and ambiguity resolution

### Valid escalation triggers

| Trigger | Example |
| -------------------- | ------------------------------- |
| User explicitly requests a human agent | “I want to talk to a real person” |
| Request falls outside policy coverage | Situation the standard policy doesn’t address |
| System cannot make further progress | Multiple attempts have failed to advance the issue |
| Multiple matches can’t be resolved | Several possible customers found, identity unclear |

### Unreliable escalation triggers

| Method | Why it fails |
| ------------------------------ | ------------------ |
| Sentiment analysis (negative score > threshold) | Sentiment doesn’t map to complexity or urgency |
| Model self-reported confidence score | Model self-assessments are uncalibrated |
| Task “feels complex” | Subjective and not programmable |

### The rule you can’t forget

> **When a user explicitly requests a human agent, escalate immediately — do not continue investigating or attempting to resolve the issue.**

```
User expresses frustration for the first time → acknowledge the emotion, offer a resolution
User explicitly asks for a human → escalate immediately, no further persuasion
```

### Handling ambiguous matches

When the tool returns multiple matching results, **request additional identifying information** — never guess:

```python
# Wrong: picking the most likely match heuristically
if len(matches) > 1:
    return matches[0]  # could be the wrong customer

# Correct: surface the ambiguity and ask for clarification
if len(matches) > 1:
    return {
        “status”: “ambiguous”,
        “matches”: matches,
        “request”: “Please provide additional identifying information (email address or order number) to confirm your identity”
    }
```

### What this tests

Common distractors are “continue investigating first,” “escalate based on sentiment score,” and “select the best-matching customer.” The exam tests whether you recognize auditable, rule-based escalation: escalate immediately on an explicit request; escalate when policy doesn’t cover the situation; escalate when meaningful progress has stalled; ask for additional identifiers when matches are ambiguous.

In practice, escalation logic should be a coded decision tree, not a model judgment call:

1. **Explicit human request**: Transfer to a human agent immediately, without trying to talk the user into staying with the bot.
2. **Issue within automated capabilities**: Acknowledge the frustration and attempt resolution; escalate only if the user requests a human again.
3. **Policy gap or exception**: Don’t let the model extend policy boundaries — escalate to a human or higher authority.
4. **Identity or order ambiguity**: Request an additional identifier; never infer from ranking, recency, or similarity.

A reliable system’s goal isn’t to minimize escalations — it’s to ensure escalations happen at exactly the right boundaries.

---

## Task 5.3: Error propagation in multi-agent systems

### Structured error context

When the child agent fails, sufficient information should be returned for the coordinator to make a recovery decision:

```json
{
  "status": "failed",
  "failure_type": "search_timeout",
  "attempted_queries": ["AI in music industry 2024", "music technology trends"],
  "partial_results": [
    { "title": "AI Music Generation Report", "url": "...", "snippet": "..." }
  ],
  "alternatives": [
    "Try searching 'music technology AI impact'",
    "Academic databases may have more comprehensive data"
  ],
  "coverage_gaps": ["Music streaming industry data missing"]
}
```

### Coordinator recovery options

With a structured error in hand, the coordinator can:

- **Retry** (transient failure — network timeout, rate limit)
- **Modify the query** (semantic failure — try alternative terms or sources)
- **Continue with partial results** (coverage is incomplete but sufficient to proceed)
- **Escalate to human review** (unrecoverable failure)

### Error propagation anti-pattern

| Anti-Patterns | Problems |
| ------------------------------------ | -------------------------- |
| Returns a generic error (`"operation failed"`) | The coordinator cannot make a recovery decision |
| Swallow the error silently and return an empty result | The coordinator mistakenly thinks it is successful |
| Single point of failure that terminates the entire process | Loss of all completed work |
| Disguise empty results as errors | The coordinator misjudges and triggers unnecessary retries |

### What this tests

This task tests whether failure information is sufficient for the coordinator to recover intelligently. A sub-agent can't just return `"failed"` or `"search unavailable"`, and it can't silently return an empty array. The coordinator needs to know what type of failure occurred, what was attempted, what partial results were retrieved, and what alternatives exist — so it can decide whether to retry, adjust the query, switch data sources, proceed with partial coverage, or escalate.

In practice, standardize sub-agent outputs into a consistent result envelope:

```json
{
  "status": "success | partial | failed",
  "failure_type": "timeout | access_denied | no_results | parse_error | unknown",
  "attempted": ["query or operation"],
  "partial_results": [],
  "recoverability": "retryable | needs_alternative_source | unrecoverable",
  "coverage_gaps": [],
  "next_options": []
}
```

The most testable distinction here is **valid empty results** versus **access failures**: the former means the query succeeded but found nothing; the latter means the query never completed successfully. The two situations call for completely different coordinator responses.

---

## Task 5.4: Context management in large codebase exploration

### Symptoms of context degradation

> The model starts saying "the usual pattern is..." instead of referencing the specific classes and structures it found earlier

### Scratchpad pattern

Persist key findings externally, across context boundaries:

```python
# agent maintains structured scratchpad
scratchpad = {
    "architecture": {
        "entry_points": ["src/main.py", "src/api/router.py"],
        "key_modules": ["auth", "payment", "notification"]
    },
    "findings": [
        {"file": "auth.py", "issue": "JWT validation missing expiration check", "line": 45},
    ],
    "dependencies": {
        "auth -> user_service": "call directly",
        "payment -> notification": "event driven"
    }
}
```

### Staged summary strategy

```
Exploration phase:
→ Sub-agent investigates a specific question (e.g., "trace refund flow dependencies")
→ Returns a structured discovery summary — not raw exploration output
          ↓
Before the next phase:
→ Compress this phase's findings into a structured state object
→ Inject that summary as the starting context for the next sub-agent
          ↓
Next phase:
→ Continues from known state — no re-exploration of already-covered ground
```

### `/compact` command

During long exploration sessions, proactively use `/compact` when the context fills with verbose discovery output. It compresses intermediate steps while retaining key conclusions.

### Crash recovery design

```python
# Each subagent outputs status to a fixed location
agent_state = {
    "manifest_version": "1.0",
    "completed_modules": ["auth", "payment"],
    "pending_modules": ["notification", "reporting"],
    "findings_file": ".claude/scratchpad/findings.json"
}
# When recovering, the coordinator loads the manifest and continues working
```

### What this tests

This task applies context management principles to codebase exploration. The key insight is that a model should never read the entire repository directly — exploration should be decomposed into bounded questions assigned to sub-agents, with findings written out to external memory rather than accumulating in the main context. The answer "let the main agent retain all exploration output" is almost never correct; scratchpad, sub-agent delegation, staged summaries, manifest, and `/compact` are the right tools.

In practice, large codebase exploration follows this order:

1. **Map first**: identify entry files, core modules, test locations, and key dependencies before reading any code in detail.
2. **Delegate specific questions**: each sub-agent answers one concrete question — call chain trace, test coverage, configuration source — not "analyze the codebase."
3. **Persist discoveries**: write file paths, class names, function names, line numbers, and conclusions to the scratchpad immediately upon finding them.
4. **Compress between phases**: turn verbose exploration output into structured conclusions before starting the next phase; use `/compact` when context fills.
5. **Build for recovery**: the manifest records completed modules, pending modules, the findings file path, and current assumptions — so a crash means resuming, not restarting.

Context degradation has a clear tell: the model starts referencing "common architectural patterns" instead of the specific files and symbols it found in this particular repository.

---

## Task 5.5: Human review workflows and confidence calibration

### The overall accuracy trap

> `97% overall accuracy` can hide specific document types or fields at only 68–71% accuracy

**The correct approach: stratify accuracy analysis by document type and field before drawing any conclusions**

| Document Type | Field | Accuracy |
| -------- | ------ | ------------------ |
| Standard Invoice | Total Amount | 99% |
| Handwritten receipt | Total amount | 68% ← Manual review required |
| All Types | Date | 95% |
| All types | Tax amount | 71% ← Manual review required |

### Confidence calibration process

```
1. Model outputs field-level confidence scores
        ↓
2. Calibrate thresholds against a labeled validation set
   (never use uncalibrated confidence scores as routing gates)
        ↓
3. Set routing thresholds based on measured accuracy:
   - High confidence (>0.95 with verified calibration) → auto-pass
   - Medium confidence (0.7–0.95) → sampling review
   - Low confidence (<0.7) → full manual review
        ↓
4. Ongoing monitoring: stratified random sampling to verify
   true error rates in high-confidence samples over time
```

### What you must verify before reducing human review

Looking at overall accuracy is not enough. You need all three:

1. Accuracy broken down by document type
2. Accuracy broken down by field
3. Continuous sampling on high-confidence output to catch distribution drift

### What this tests

This task tests whether model confidence can serve as a reliable human-review routing signal. Raw model confidence cannot be used directly as an auto-pass gate — it must be calibrated against labeled validation data, and accuracy must be verified by document type and field independently. High aggregate accuracy does not guarantee that every segment is ready for automation.

In practice, structure the human review pipeline into four queues:

1. **Mandatory review**: low confidence, conflicting sources, missing required fields, high-value amounts, or high-risk operations.
2. **Sampling review**: high-confidence output still needs ongoing stratified sampling to detect distribution drift and new error patterns.
3. **Auto-pass**: only after the specific document type and field combination has been calibrated and verified to meet the accuracy threshold.
4. **Rule-based review**: hard rules for specific fields — amount ranges, date format checks, tax consistency validation.

When the exam mentions "97% overall accuracy," the immediate question to ask is: stratified by what? Document type, field, confidence tier, and time period all need to be checked before reducing manual review.

---

## Task 5.6: Source chain preservation in multi-source synthesis

### Preserving claim-source mappings

The comprehensive sub-agent must retain **claim-source mapping**:

```json
{
  "claim": "AI increases music creation efficiency by 35%",
  "confidence": "high",
  "sources": [
    {
      "url": "https://music-tech-report.com/2024",
      "document_name": "Music_AI_Report_2024.pdf",
      "excerpt": "Average creative time reduced from 8 hours to 5.2 hours...",
      "publication_date": "2024-03-15"
    }
  ]
}
```

### Handle conflicting data

When trusted sources give different data:

```
Mistake: Choose one at random or choose the one that looks bigger
Correct approach: keep both values ​​at the same time, indicate the source and date

Example:
{
  "statistic": "AI-assisted creation efficiency improvement",
  "values": [
    {"value": "35%", "source": "MIT Research 2024", "date": "2024-01"},
    {"value": "28%", "source": "Stanford Survey 2023", "date": "2023-11"}
  ],
  "note": "Numerical differences may reflect different methodologies or time periods — not necessarily a factual conflict"
}
```

### Why timestamps matter

Different values from different time points may be **temporal change**, not contradiction — but without publication and data collection dates, the coordinator can't tell the difference. Both dates should be required fields in sub-agent output schemas.

### Synthesis report structure

- **Well-established findings** vs **contested findings** → clearly distinguished sections
- Financial data → tables (facilitates comparison)
- Event narratives → prose (preserves chronology)
- Technical findings → structured lists (easy to reference)

### What this tests

This task tests whether traceability survives the synthesis pipeline. The correct answer preserves claim-source mappings all the way through: each conclusion is bound to its source document, URL, excerpt, and publication date. When sources conflict, the system represents the conflict explicitly with both values and their context — it doesn't let the model pick one. A synthesis report that reads smoothly but can't be verified against its sources is unreliable.

A complete synthesis report has four layers:

1. **Claim layer**: What is being asserted — confirmed, partially supported, or disputed?
2. **Evidence layer**: Source document, excerpt, publication date, and methodological context.
3. **Conflict layer**: Where credible sources disagree, both values with their sources and a note on possible explanations.
4. **Format layer**: Financial data in tables, events in prose, technical findings in structured lists.

The test for any multi-source synthesis: can a reader trace every claim back to a specific source? If the answer is no, the synthesis isn't reliable.

---

## Domain 5 deep dive: context management, reliability, and the human-machine boundary

Domain 5 is the domain candidates most often underestimate. At 15% weight it looks light, but it connects all the others: agent loops generate tool results that bloat context; multi-agent handoffs lose source attribution during summarization; structured extraction produces confidence scores that need calibrated review thresholds; customer support sessions hit ambiguity and escalation decisions; long codebase explorations cause the model to forget early findings. The real question Domain 5 is asking is not “how big is the context window?” — it’s how to preserve key facts, propagate failures faithfully, handle uncertainty honestly, and design reliable human-review mechanisms across extended interactions.

### 1. More context isn’t always better — relevance is

The real danger in long conversations isn’t running out of tokens; it’s having key information buried under low-value content. A single order lookup can return 40+ fields, but a refund decision only needs the order ID, amount, date, status, and return eligibility. Passing the full result through to the context every round makes it progressively harder for the model to find what actually matters. The fix is to trim tool output in code before it ever enters the messages array — keep only the fields the current task requires.

Progressive summarization carries the same risk. “Customer requested a refund of $129.99 before 2024-03-15” can quietly become “customer requested a refund” in a summary, losing the date, the amount, and the explicit commitment. Values, dates, order numbers, and stated user expectations should never live solely in natural language summaries. They belong in a structured case facts block that gets injected into every API call so the model is still working from the same precise constraints in round 30 as it was in round 1.

The “Lost in the Middle” effect compounds both problems: models process the beginning and end of long inputs more reliably than the middle. The solution isn’t just to shorten context — it’s to reorganize it. Put key findings and constraints at the top, divide detailed evidence into clearly labeled sections, and repeat the most critical constraints at the end. When synthesizing multiple sources, lead with an executive summary and claim-source map before presenting the detailed material.

### 2. Escalation should be rule-driven, not sentiment-driven

In customer support, “escalate when the user seems angry” is not a reliable policy. Sentiment analysis misclassifies regularly, varies across cultures, and doesn’t map cleanly to case complexity. Model self-reported confidence (“I’m 80% sure about this”) is similarly uncalibrated and shouldn’t drive escalation decisions. Robust escalation triggers are programmable rules: the user explicitly requests a human agent; the request falls outside existing policy coverage; the system has failed to make meaningful progress across multiple attempts; identity or order matches are ambiguous and can’t be resolved without additional information.

When a customer explicitly asks for a human — statements like “I want to speak to a real person” or “please transfer me” — the agent must escalate immediately and stop trying to resolve the issue. Continuing to investigate or persuade at that point ignores the user’s intent and erodes trust. A user expressing frustration without making that explicit request is a different situation: acknowledge the emotion, offer a resolution, and only escalate if they reiterate the preference for a human.

Ambiguity can’t be resolved by guessing either. When the tool returns multiple customer or order matches, picking the first, most recent, or largest-amount result is never acceptable — it can expose one customer’s data to another or operate on the wrong account. The only safe path is to ask for additional identifying information (email suffix, billing zip code, order number, last four digits of the phone number) before proceeding.

### 3. Give the coordinator enough context to recover from failures

Sub-agent failures are a normal operating condition in multi-agent systems. Search timeouts, permission errors, unavailable sources, and empty result sets all happen. The quality of failure reporting determines whether the coordinator can recover intelligently or is forced to give up. A generic `”search unavailable”` message tells the coordinator nothing: it doesn’t know what query was attempted, how many retries occurred, whether partial results were returned, or what alternative approaches might work. Structured errors should carry `failure_type`, `attempted_queries`, `partial_results`, `alternatives`, and `coverage_gaps` at minimum.

It’s also critical to distinguish a valid empty result from an access failure. `{“status”: “success”, “count”: 0, “results”: []}` means the query executed successfully but found no matches — the coordinator should probably refine the query or note a coverage gap. `{“status”: “failed”, “failure_type”: “timeout”}` means the query never completed — the coordinator should retry or try a different tool. Conflating these two situations causes either unnecessary retries (treating empty results as failures) or false coverage gaps (treating failures as empty results).

A single sub-agent failure should never terminate the entire workflow. The coordinator can proceed with partial results, flag the gap in the output, route around the failed source, or escalate to a human reviewer. The final synthesis should clearly distinguish findings that are well-supported from areas where sources were unavailable — readers deserve to know which conclusions rest on solid evidence and which are provisional.

### 4. Large codebase exploration requires external memory

Context degradation in long exploration sessions has a recognizable symptom: the model starts saying “typically this kind of architecture would have...” instead of referencing the specific files, classes, and functions it discovered earlier. Early findings have been compressed or pushed out of the active context window. The tools for fighting this are scratchpad files, staged summaries, sub-agent isolation, and `/compact`.

A scratchpad file (e.g., `.claude/scratchpad/findings.json`) is external memory that persists beyond any single session’s context boundary. Every significant discovery — entry points, key dependencies, missing validations, open questions — should be written to the scratchpad immediately rather than mentioned once in the current session’s output and then forgotten. Subsequent agents read the scratchpad first rather than trusting the conversation history.

Exploration tasks should be decomposed into specific, bounded questions assigned to separate sub-agents: “find all files that call `process_refund()`,” “list all database migrations from the last six months,” “trace the authentication middleware chain.” Each sub-agent returns a structured summary, and the main agent maintains a high-level architecture map rather than absorbing every raw detail. This also makes crash recovery viable: a manifest recording completed modules, pending modules, findings file paths, and current assumptions means the coordinator can resume after an interruption rather than re-exploring from scratch. `/compact` is the right tool when the context fills with verbose exploration output that has already served its purpose.

### 5. Confidence calibration before reducing human review

An overall accuracy of 97% can mask an 68% accuracy on handwritten receipts or a 71% accuracy on tax amounts — exactly the fields where errors cause real business problems. Before reducing manual review coverage, accuracy needs to be analyzed by document type, by field, and by confidence tier, not just in aggregate. A dimension that hasn’t been verified at the segmentation level shouldn’t be moved to automation no matter how good the overall score looks.

Model-reported confidence must be calibrated against labeled validation data before it can be used as a routing threshold. Calibration means measuring — on a held-out set with known correct answers — what the true accuracy is for fields the model marks as high, medium, and low confidence. Only after that mapping is established can you set a threshold with predictable error rates. “Confidence: high” from an uncalibrated model is not a reliable signal.

Even after automation is live, stratified random sampling must continue. Document distributions shift over time: new vendor templates, new languages, new scan quality, new field layouts. Sampling across document type, field, and confidence tier can catch new error patterns before they accumulate silently. Manual review is a continuous monitoring mechanism, not a one-time setup decision.

### 6. Source chains are the lifeline of multi-source synthesis

The most common failure mode in multi-source synthesis is attribution loss during summarization. A sub-agent compresses three documents into “AI increases creative efficiency by 35%” without retaining which document, which page, which publication date, or which methodology produced that number. Once that context is gone, the synthesizing agent has no way to verify, challenge, or cite the claim. Every sub-agent in a research pipeline should output structured claim-source mappings — each claim bound to `source_url`, `document_name`, `excerpt`, and `publication_date` — and downstream agents must preserve and merge those mappings rather than restating conclusions in their own words.

When credible sources give conflicting statistics, the right response is to surface both values with their context, not to pick one. A 2023 survey reporting 28% and a 2024 experiment reporting 35% may reflect a genuine time trend, different sample populations, or different methodological definitions — not a factual contradiction. The report should present both values, their sources, their dates, their methods, and a brief note on possible explanations. Letting the model choose the “more authoritative” number delegates a methodological judgment that should belong to the reader or a domain expert.

Presentation format matters too. Financial data belongs in tables (facilitates comparison), event narratives belong in prose (preserves chronology and causality), technical findings belong in structured lists (easy to reference), and contested conclusions belong in explicit for/against/uncertain blocks. Flattening everything into one uniform format obscures important differences in the underlying data structure.

### 7. How to approach Domain 5 questions

The pattern-matching table for Domain 5 scenarios: facts forgotten in long conversations → case facts block and structured issue layers; tool results too large → trim to task-relevant fields before they enter context; key findings missed in long documents → place summaries at the top, divide evidence into sections; user explicitly requests a human → escalate immediately, no exceptions; policy doesn’t cover the request → escalate; tool returns multiple matches → request additional identifiers; sub-agent fails → structured error with `failure_type`, `partial_results`, `alternatives`; codebase exploration drifts toward generalities → scratchpad, sub-agent isolation, staged summaries, manifest, `/compact`; considering reducing human review → stratified accuracy and calibrated confidence thresholds first; multi-source conflicts → retain claim-source mappings, timestamps, and explicit conflict annotations.

Wrong-answer patterns are equally consistent: sentiment score as escalation trigger; model self-reported confidence as auto-pass gate; raw tool output passed directly into context; empty list returned on failure; entire workflow terminated on single sub-agent failure; sources discarded during summarization; one conflicting statistic arbitrarily selected; human review reduced based on aggregate accuracy alone. The correct answer preserves facts, sources, errors, and uncertainties — it makes the system traceable, recoverable, and auditable across the full life of a long interaction.

### 8. Layer context instead of letting it grow as one flat record

Production systems shouldn’t maintain a single ever-growing messages array. A more reliable design layers context by purpose: a conversation layer holding the complete interaction, a case facts layer holding structured key facts, an issue state layer tracking per-issue status, a tool summary layer holding trimmed tool outputs, a source map layer holding claim-source relationships, and a handoff layer holding human-ready summaries. Each layer is managed separately rather than mixing everything into one undifferentiated stream of natural language.

This structure solves practical problems directly. When a user raises both a refund issue and a billing dispute, the facts for each issue stay in separate state objects so the model doesn’t apply the wrong amount to the wrong case. When escalating to a human reviewer, you pass the handoff summary — not a dump of raw tool output. When synthesizing from multiple sources, you pass the claim-source map forward — not the full text of every document. The essence of “context management” in Domain 5 is placing information at the layer that matches its purpose, not just keeping everything in one place.

### 9. Reliability is an engineering problem, not a prompting problem

Many wrong answers in this domain assign reliability to the model itself: “tell Claude to pay close attention,” “have Claude self-assess its confidence,” “let Claude decide whether to escalate.” The correct framing is engineering-based: reliability comes from system design, not model instruction. Tool output filtering happens in code. Key fact persistence is handled by the state layer. Error classification is enforced by structured schemas. Escalation rules are implemented as explicit policy logic. Human review thresholds are calibrated against labeled validation sets. The model contributes judgment, but shouldn’t be the sole guarantor of any of these properties.

The practical implication: a customer support agent can understand intent, but identity disambiguation requires a code-level guard that requests additional identifiers — not a model instruction to “be careful.” A document extraction system can output confidence scores, but whether those scores are trustworthy as routing signals depends on calibration data, not the model’s self-assessment. A research synthesis agent can combine findings, but surfacing and labeling source conflicts requires structured schema enforcement, not prompting the model to “be thorough.”

### 10. Context ages as well as grows

Long-running interactions don’t just accumulate content — they accumulate staleness. In codebase exploration, files may have been modified since the first scan. In customer support, order status updates between turns. In research, new publications appear. In CI pipelines, new commits change the code under analysis. When the model reasons against an old version of a fact without knowing it’s stale, the output is wrong in ways that are hard to detect. Key tool results should carry timestamps, file commit hashes, or `retrieved_at` markers so freshness can be assessed when resuming a session.

This connects directly to Domain 1’s session recovery patterns. Resuming an old session doesn’t automatically mean resuming from a valid state — the context needs a freshness audit. Some facts can be treated as stable (e.g., customer ID), some need to be refreshed on each turn (e.g., order status), and some are best treated as historical reference only (e.g., intermediate exploration output from a week ago). When an exam question describes an error caused by “analysis based on outdated files after recovery,” the correct direction is to inject a fresh structured summary and re-examine the changed files — not to blindly resume where things left off.

### 11. Human review queues need priority, not just accumulation

Human-in-the-loop doesn’t mean routing all uncertain output to a single review queue and processing it FIFO. Reviewer capacity is limited, and the highest-risk items should reach reviewers first. The priority ordering should be: low-confidence fields → numerical conflicts between sources → high-risk operations (large amounts, contract terms, permission grants) → ambiguous or unusual document formats → standard high-confidence fields that calibration data shows are reliable (which can be auto-passed or sampled at low frequency).

Review interfaces should show the original text excerpt, the extracted field value, the confidence score, and any detected conflicts — not just the final JSON. This gives reviewers the context to catch errors the model might produce in predictable but subtle ways. Corrections made by reviewers should be logged with error type (OCR failure, field definition mismatch, prompt gap, schema mismatch, missing source), so the system can improve over time. Human review without a feedback loop is just a cost center; with one, it becomes the source of calibration data, new few-shot examples, and schema refinements.

### 12. Traceability is the single throughline for all Domain 5 answers

Every Domain 5 question can ultimately be evaluated against one criterion: can you trace the output back to its source? Can a claimed fact be traced to a user statement or a tool result? Can an error be traced to a specific failure type and what was attempted? Can a conclusion be traced to its source document, excerpt, and collection date? Can an escalation be traced to the policy gap or explicit user request that triggered it? Can a confidence-based routing decision be traced to a calibrated threshold? Wherever that chain of evidence breaks — in summarization, in synthesis, in escalation — the system becomes unreliable.

Traceability doesn’t mean preserving everything verbatim. Full verbatim output wastes context; conclusions-only output loses evidence. The target is structured compression: retain the facts, sources, methods, timestamps, conflicts, and confidence metadata needed for every downstream decision, and discard the rest. When you see “summary” as an answer option on the exam, ask what that summary preserves — a summary that retains metadata supports traceability; one that produces only polished prose does not.

### 13. Connection with other domains

Domain 5 is the reliability layer that sits on top of everything else. Domain 1’s multi-agent workflows require context handoffs between sub-agents and structured error propagation to coordinators. Domain 2’s tool design requires trimmed tool outputs and structured error schemas. Domain 3’s codebase exploration workflows require scratchpad files, manifests, and `/compact`. Domain 4’s structured extraction workflows require calibrated confidence thresholds and human review feedback loops. Domain 5 isn’t a separate chapter — it’s the set of design choices that determines whether any of those systems remains reliable under real operating conditions.

When a question appears to touch multiple domains simultaneously — multi-source reporting, sub-agent failure, and human review all in the same scenario — start from the information-loss angle. What does the system know? Where does that information go? Does the source survive? Does the error survive? Will a human reviewer have enough to act on? That framing is more robust than memorizing patterns for individual topic areas.

### 14. Three scenarios that tie Domain 5 together

The first scenario is a customer support session. A user raises a refund, then adds a billing dispute, the tool returns multiple matching orders, and eventually the user explicitly asks for a human agent. A well-designed system maintains a case facts block, keeps per-issue state objects separate, requests additional identifiers to resolve the order ambiguity, escalates immediately when the human request is made, and passes a structured handoff summary. A poorly designed system collapses everything into a rolling natural language summary, guesses which order to operate on, and either escalates prematurely on sentiment or fails to escalate on an explicit request.

The second scenario is a multi-agent research pipeline. A search sub-agent times out but returns some partial results, a document sub-agent finds two credible sources with conflicting statistics, and a synthesis agent needs to produce a report. A well-designed system has the search agent report its partial results and coverage gaps explicitly, the document agent preserve both conflicting values with their sources and methodological context, and the synthesis agent distinguish confirmed from disputed conclusions in the final output. A poorly designed system swallows the search failure, lets the synthesis agent pick one of the conflicting statistics, and produces a smooth-reading report that can’t be verified.

The third scenario is a large codebase exploration. The main agent needs to understand the refund flow, and four sub-agents investigate the API layer, database, tests, and event pipeline respectively. A well-designed system has each sub-agent write structured findings to a shared scratchpad, the main agent maintain an architecture-level summary rather than absorbing all the raw details, and a manifest track completed and pending modules for crash recovery. A poorly designed system has one agent read files continuously until it starts generalizing from “typical patterns” rather than the specific code in front of it.

Together, these three scenarios cover the core test points of Domain 5: key fact persistence, escalation logic, ambiguity resolution, error propagation, source retention, context degradation, and human review calibration. They’re not isolated topics — they’re all expressions of the same underlying problem: information degrades, goes stale, and loses its provenance over long multi-step processes, and the system has to actively fight that entropy.

### 15. Minimum viable reliability checklist

Before deploying any Claude-powered system that involves long interactions, multiple tools, or multiple agents, work through these ten questions. Which facts must survive for the full duration of the interaction and can never be lost in summarization? Which tool result fields can be safely discarded before they enter context? Which failure types should trigger retries, which should be reported as coverage gaps, and which require escalation? Does the system escalate immediately when a user explicitly requests a human? Does it request additional identifiers rather than guess when tool results return multiple matches? Do sub-agents retain source and timestamp metadata in their outputs? Are long exploration tasks supported by scratchpad files? Are restartable tasks backed by a manifest? Has confidence-based automation been calibrated against labeled data at the document type and field level? Can every claim in the final output be traced to a specific source?

A system that can’t answer these questions will work in demos — where sessions are short, data is clean, and every step succeeds — but will degrade in production, where sessions are long, data is messy, and failures are routine. Domain 5 is the difference between a prototype and a system someone can actually rely on.

Think of it as an information fidelity problem. Fidelity doesn’t mean preserving everything — it means preserving what affects decisions: amounts, dates, identities, statuses, sources, timestamps, errors, conflicts, and the basis for human judgment. A system that can answer “where does this conclusion come from, is it still current, and what happens if it’s wrong?” at the end of any long interaction has the foundation it needs.

A useful self-check: could you come back to this system three days later and still explain clearly why it produced the output it did?

---

## Pre-exam checklist

1. **"Lost in the Middle"** → Put critical information at the beginning or end of long inputs, not buried in the middle
2. **Case facts block** → Extract key values (amounts, dates, order IDs, statuses) into a structured block injected every turn — don’t rely on natural language summaries to preserve them
3. **Trim tool results** → Filter tool output down to task-relevant fields in code before it enters the context
4. **Escalate immediately** → When a user explicitly requests a human agent, stop trying to resolve the issue and escalate — no exceptions
5. **Escalation triggers** → Explicit user request > policy gap > no progress > ambiguous matches; sentiment score alone is not a valid trigger
6. **Structured error propagation** → Sub-agents must return `failure_type`, `attempted_queries`, `partial_results`, and `alternatives` — not just `"operation failed"`
7. **Empty result ≠ access failure** → `{"status": "success", "count": 0}` means no match found; `{"status": "failed", "failure_type": "timeout"}` means the query never completed — treat them differently
8. **Context degradation** → Scratchpad files + staged summaries + sub-agent isolation + `/compact` when context fills up
9. **Overall accuracy trap** → 97% aggregate accuracy can hide 68% accuracy on a specific document type or field — always stratify by document type and field before reducing human review
10. **Conflicting sources** → Preserve both values with their sources and timestamps; do not let the model pick one arbitrarily