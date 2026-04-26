# CCA-F Simulation Questions — Domain 5: Context Management and Reliability

> There are 60 questions in total, and the recommended completion time is 75 minutes. Single choice for each question.

---

## Task 5.1 Context window limitation and intermediate forgetting effect (Q1–Q12)

**Q1.** In a conversation involving 150,000 tokens, you need to provide Claude with three important documents: a user requirements document (beginning), a technical specification (middle), and a test case (end). According to the intermediate forgetting effect, Claude is most likely to ignore details of which of the following documents?

A) User requirements document
B) Technical specifications
C) Test cases
D) Three documents were ignored

> **Answer: B**
> Explanation: The middle forgetting effect indicates that Claude pays more attention to the content at the beginning and end of the context, and the content in the middle (technical specifications) is most easily ignored. A good architect should place key information at the beginning or end of the conversation, rather than burying it in the middle.

**Q2.** You are designing a long-term code review workflow. The current conversation has accumulated 280,000 tokens, and the user asks: "Please review this newly added critical API verification module." What action should you take?

A) Continue reviewing in the current conversation as the number of tokens is still within Claude's processing range
B) Use the /compact command to compress the conversation history and continue in the same conversation
C) Start a new conversation and pass the previous context summary to the new conversation
D) Have the user re-upload all code files into a new session

> **Answer: B**
> Explanation: The /compact command can compress conversation history without losing important information and maintain context coherence. This is more efficient than starting a new conversation and avoids the forgetting effect. For critical code review tasks, /compact should be used first to manage context.

**Q3.** When designing a multi-turn dialogue system, you need to introduce a new key constraint after round 45 that the user has not yet seen. To maximize the likelihood that Claude will obey this constraint, you should:

A) Insert this constraint in the middle of conversation round 45
B) Add constraints to the very end of the conversation (in the form of new information)
C) Modify the system prompt at the beginning of the conversation, but this will lose the previous context
D) Split into two conversations, the first discussing the existing content and the second starting with the new constraints

> **Answer: B**
> Explanation: Due to the intermediate forgetting effect, new constraints placed at the end of the conversation (the most recent content) will receive higher attention. Claude is more sensitive to the latest information than to earlier information.

**Q4.** Your agent has been running a continuous workflow for 30 hours. The initial system prompt is buried in 500,000+ tokens of conversation history. The agent begins to exhibit behavior inconsistent with the original instructions. What's the most likely cause?

A) Claude’s underlying model capabilities decline
B) Original system prompts become less effective due to intermediate forgetting effects
C) The number of tokens has exceeded Claude's maximum context window
D) There is a bug in the logic code of the agent program

> **Answer: B**
> Explanation: In extremely long conversations, the system prompts at the beginning will gradually lose their impact, and the forgetting effect in the middle will make them less noticeable. This is why you need to start new sessions periodically or use external state management.

**Q5.** When building an architecture that needs to process a large number of contexts, you plan to use an external file (scratchpad) to store intermediate results. Which of the following statements about this strategy is most accurate?

A) External files completely eliminate the need for context management
B) External files can serve as supplementary memory, but they still need to be referenced in the conversation
C) The external file should contain all the details and only the summary should be kept in the conversation
D) External files are not suitable for long-term workflows and should be avoided

> **Answer: B**
> Explanation: Scratchpad files are valid external memory mechanisms, but they must be explicitly referenced and loaded in the session to function. You cannot expect Claude to automatically access the contents of files that are not in the current context.

**Q6.** At hour 18 of a 24-hour data processing pipeline, Claude suddenly ignored a key data format specification that was clearly given at hour 2. According to best practices for context management, this indicates that you should:

A) Increase system resources to handle more tokens
B) Start a new conversation session at a critical checkpoint in the workflow
C) Ask Claude to focus more
D) Accept that this is an inherent limitation of AI systems

> **Answer: B**
> Explanation: Long-term workflows should set session boundaries at critical checkpoints, use fork_session or start new sessions. This avoids early instructions being overwhelmed by intermediate forgetting effects. A single conversation for 24 hours can cause severe context drift.**Q7.** You need to transfer a 50 MB log file in Claude for analysis. Regarding token counting, which of the following is the most accurate understanding?

A) File size (MB) directly corresponds to the number of tokens, 50 MB ≈ 500,000 tokens
B) Token count depends on the actual content and format in the file, files with higher text density consume less tokens
C) The token count is always the same regardless of file format
D) Binary files cannot be counted as tokens and must be converted to text first

> **Answer: B**
> Explanation: Token is counted based on content and encoding method, not a simple file size conversion. The same 50 MB content, if it contains structured data (JSON, CSV), will consume much less tokens than the original log. Understanding this is critical to accurate contextual budget management.

**Q8.** In a multi-round conversation, you give a clear rationale for an architectural decision in round 5, and by round 35, Claude makes a suggestion that conflicts with that decision. The most effective correction methods are:

A) Restate the original rationale in the current round 35
B) Tell Claude to look up Round 5
C) Re-emphasize the decision and its rationale at the end of the conversation (the closest position)
D) Start a new conversation, starting with decision round 5

> **Answer: C**
> Explanation: Taking advantage of the characteristics of the middle forgetting effect, re-emphasis on key decisions at the end of the conversation (the most recent content) will have the greatest impact. This is more effective than simply "referring to round 5" because restatement will re-enter the optimal focus zone.**Q9.** An agent system needs to maintain a dynamic configuration object that is modified 15 times during 50 rounds of dialogue. Which of the following is best regarding the best way to manage this configuration?

A) Each modification fully describes the current status in the dialogue
B) Store the complete configuration in an external file and only reference the file version in the conversation
C) Only track the latest modifications in the conversation and ignore historical changes
D) Require users to provide complete configuration on each visit

> **Answer: B**
> Explanation: For frequently changing states, external file storage is the optimal solution. This avoids stating complete configurations repeatedly in conversations, reducing token consumption while maintaining state accuracy. Only documents need to be cited in the conversation.

**Q10.** When designing a long-term workflow that requires strict compliance with initial constraints, how do you plan to prevent the intermediate forgetting effect from affecting those constraints?

A) List all constraints at once in the system prompt, and I believe Claude will always comply with them
B) Implement a periodic "constraint refresh" mechanism that regularly restates key constraints at the end of the conversation
C) Encode constraints into the agent's action logic instead of just relying on context
D) Accepting that constraints will inevitably be forgotten over time is inevitable

> **Answer: C**
> Explanation: The most reliable way is to transfer constraints from context to system design level. Enforce constraints through the agent's action framework, validation rules, and decision logic, rather than relying solely on statements in the conversation. The effect can be enhanced by supplementing it with periodic constraint refreshes (option B).

**Q11.** Your workflow has reached the context window capacity of 400,000 tokens. The user proposes a new, relatively independent task requirement. The smartest architectural decisions are:

A) Compress current conversation to continue working in the same session
B) Use fork_session to create a new forked session to handle new tasks
C) Tell the user that they must wait for the current task to complete
D) Delete early dialogue turns to make room

> **Answer: B**
> Explanation: fork_session allows the creation of new session branches at critical checkpoints, retaining the complete context of the original session while creating an independent space for new tasks. This is the best way to handle new tasks when the context capacity limit is approached.

**Q12.** While reviewing a 72-hour automated workflow log, you discover that Claude forgot two key naming conventions in the last 12 hours. What architectural problem does this most likely indicate?

A) Claude's model declines in ability over the long run
B) Workflow lacks checkpointing and session management strategies
C) The logging system does not properly capture the naming convention
D) User naming conventions are not clear enough

> **Answer: B**
> Explanation: This is a typical context drift problem caused by a lack of proper session management. Long-term workflows should be checkpointed every 4-8 hours to reconfirm key engagements when starting a new session. A single 72-hour conversation can lead to severe context forgetting.

---

## Task 5.2 Progressive summary and context retention (Q13–Q24)

**Q13.** You are managing a week-long data analysis project that produces a large number of intermediate results each day. Regarding the application of progressive summarization, which of the following methods best balances information preservation and context compression?

A) Manually create a full summary every day, completely replacing the original data
B) Use hierarchical summaries: save daily details in external files and keep only high-level summaries and key metrics in the conversation
C) Don't create any summary, keep all data in the conversation
D) Use the /compact command to completely automate the summarization process without manual intervention

> **Answer: B**
> Explanation: A layered approach combines the advantages of external storage and conversation summarization. Detailed data is saved in the file for easy reference, and enough context is maintained in the conversation to continue working. This avoids the information loss problem of single-level summarization.

**Q14.** Conducted 30 rounds of analysis and revision in a complex code refactoring project. Now we need to summarize the progress. Which of the following is most accurate regarding manual summarization vs /compact command?

A) /compact is always better than manual summarization because it is faster
B) Manual summarization allows finer control and can highlight what the architect considers most important, but takes longer
C) Both methods produce the same results, just at different speeds
D) Manual summarization loses key technical details and should always be used /compact

> **Answer: B**
> Explanation: /compact is a general automation tool, while manual summarization allows architects to selectively highlight key decisions, pitfalls, and learnings. For high-risk decisions, such as security architecture, the controllability of manual summarization is even more valuable.

**Q15.** A multi-stage machine learning pipeline has completed the data preparation and feature engineering stages and now enters the model training stage. The detailed code and experimental notes of the first two phases occupy 200,000 tokens. What should you do with this information?

A) Keep all details in the conversation as you may need to go back and make adjustments in the future
B) Create a detailed summary (containing all key parameters and decisions), save it in an external file, and refer to it in one sentence during the conversation
C) Remove all early details and focus entirely on model training
D) Use the /compact command to delete the details and keep only the execution code

> **Answer: B**
> Explanation: Between different stages, a "handover document" should be created. Detailed information is stored externally and reference is kept concise in conversations. This not only retains the ability to trace, but also frees up context space. If subsequent adjustments to earlier stages are required, external files can be loaded.

**Q16.** When doing progressive summarization, you face a dilemma: some intermediate results, while not the final answer, contain important reasoning steps and failed attempts. Deleting these will lose the context, and retaining them will waste the token. The most reasonable approach is:

A) Completely remove all failed attempts and keep only successful paths
B) Keep a simplified record of "what failed and why" in an external file, no details are kept in the conversation
C) Keep everything because token is enough
D) Ask the user to decide which steps should be retained

> **Answer: B**
> Explanation: Failed attempts are critical to understanding why a specific decision was made (decision tracing). But the details don't need to be in the conversation, and the summary "why it failed" in the external file provides enough context for subsequent decisions without wasting tokens.

**Q17.** You are designing a workflow that spans multiple Claude sessions. Session A completed the initial analysis, and now it is time to start Session B to drill down further. What are the most important elements when it comes to handover documentation?

A) Full conversation history transcript from Session A
B) Key findings, hypotheses, ruled out options and recommended next steps
C) All raw data and detailed codes
D) A brief summary of the user’s original needs

> **Answer: B**
> Explanation: An effective handoff should include the "why" of the decision (what-ifs and ruled-out options), not the full history. This way session B can quickly understand the context and continue working. The full history will result in wasted tokens, while the original requirement should already be in the system prompt.

**Q18.** In a long-term project dealing with sensitive financial data, multiple progressive summarization cycles were conducted. Which of the following is most accurate regarding the risks of data retention and information loss?

A) Progressive summaries are safe for numerical financial data because the numbers cannot be misinterpreted
B) Each summary increases the risk of information loss, and financial data are particularly prone to errors due to rounding or summarization.
C) No information will be lost as long as /compact is used
D) Financial data should completely avoid any form of summary

> **Answer: B**
> Explanation: This is the key risk of progressive summarization. Each digest cycle is a potential point of information loss. With financial data, even small rounding errors compound. Best practice is to maintain the raw data in an external persistence store and only reference it in the conversation, rather than summarizing it repeatedly.

**Q19.** A project lasting 10 days has accumulated 8 daily summaries. Now you need to do a "summary of summaries" to create a cycle summary. What are the main risks of this operation?

A) There is no risk, information compression is always beneficial
B) Each abstraction level introduces selection bias and information loss, especially for details that may be simplified across multiple abstracts
C) Requiring too many tokens to process the digest of the digest
D) Users will be confused by too many summaries

> **Answer: B**
> Explanation: This is called the "summary compression problem" or "information decay". Context and nuance are lost with each layer of abstraction. Best practice is to keep a version of the original data, with the abstracts as an indexing/navigation tool, rather than linking the abstracts together.

**Q20.** When using a strategy that combines external file storage and summarization, how should you tell when a chunk of information should go to an external file rather than remain in the conversation?

A) All data should be in external files, conversations are for interaction only
B) Only consider external storage after exceeding 10,000 tokens in the conversation
C) If the information is not needed in the current task, or may be referenced in subsequent steps but does not require real-time interaction, it should be stored externally.
D) Never move information from conversations to external files, this creates administrative complexity

> **Answer: C**
> Explanation: The key criterion for external storage is "whether it needs to be actively used in the current session". Configuration, complete historical data, reference materials should be external. Ongoing analysis, latest findings, and real-time decisions should be part of the conversation.

**Q21.** Your workflow has created multiple summary files: daily summary, weekly summary, and a master index. How to ensure that this summary system does not become a maintenance burden?

A) Stop creating summaries and just keep the original data
B) Establish clear update rules: update only at specific points in time (such as every weekend), use automated scripts to generate summaries
C) Require a human to check all levels every time the summary is updated
D) Allow abstracts to evolve freely without version control

> **Answer: B**
> Explanation: A standardized summary strategy (regular update points, automated generation) can prevent the summary system from becoming a maintenance nightmare. Loose, haphazard summary updates can lead to confusion and data inconsistencies.**Q22.** While reviewing a completed long-term project, you discovered that the final summary had slight but important differences from what actually happened (e.g., the reasons for a key decision were simplified). What does this indicate?

A) The abstractor made a mistake
B) Expected information loss during progressive summarization - this is why the original work product should be preserved
C) Summary should be avoided completely
D) The abstract is not detailed enough

> **Answer: B**
> Explanation: This is empirical proof why best practices emphasize retaining the original data and using summaries as navigation tools, not as substitutes. Summaries always lose some nuance, which is problematic for auditing, study, and later reference.

**Q23.** When designing a long-term project workflow that supports multi-person collaboration, what is the most critical thing about shared summaries and context retention?

A) Ensure all participants see the same version of the summary, via frequent synchronization
B) Create a summary of the version control system, clearly marking who created the summary when, and based on what original data
C) Avoid any snippets and ensure everyone has access to the full original conversation history
D) Have each participant create their own version of the summary

> **Answer: B**
> Explanation: In a multiplayer environment, version control and provenance tracking of summaries are critical. Participants need to understand what point in time a summary is based on and who created it. This prevents decisions based on outdated summaries.

**Q24.** When applying a progressive summary strategy, how should you deal with "hypotheses" and "open questions" - these are often omitted in summaries but are important for recovery efforts?

A) Hypotheses and open questions are not important in the abstract and should be removed
B) Maintain a dedicated "Assumptions and Tokens" section in each summary to ensure this information is not lost
C) Keep a complete copy of the original conversation to extract this information
D) Mention only confirmed facts in the summary

> **Answer: B**
> Explanation: Hypotheses and open questions are key to decision tracking. If these are omitted from the abstract, subsequent persons may re-make assumptions that have been excluded. A well-structured abstract should clearly list what "we hypothesize" and "what we still need to verify".

---

## Task 5.3 Escalation patterns and human handover protocol (Q25–Q36)

**Q25.** In an automated customer service agent, when is the most critical time to escalate to a human agent?

A) When a user's message contains more than 100 words
B) When there is real ambiguity - the agent is unable to determine the user's intent, or needs to perform a potentially irreversible action
C) When users make any complaints or negative emotions
D) When the agent's confidence score is less than 0.9

> **Answer: B**
> Explanation: Escalation should be based on behavioral risk (reversibility, consequences) and cognitive risk (ambiguity), not just sentiment or confidence scores. Emotions are not triggers for escalation; ambiguity and irreversible actions are.

**Q26.** A data deletion request arrives at your proxy system. The user says "delete all my data" but has multiple different types of data in the account (transaction history, personal information, preferences). Agents should:

A) Delete all categories immediately
B) Escalate to a human agent because there is ambiguity about "all data" and deletion is an irreversible operation
C) Ask the user to confirm again and then execute
D) Delete recently created data and retain historical data

> **Answer: B**
> Explanation: This involves two escalation criteria: (1) ambiguity — "all data" needs to be clarified, (2) irreversibility — data deletion cannot be undone. The combination of the two requires escalation to a human. Automated acknowledgments are not sufficient to handle this high-risk situation.**Q27.** Regarding the "over-upgrade" problem of upgrades, which of the following is most accurate?

A) Over-upgrading is okay because it prioritizes security
B) Frequent excessive upgrades will lead to the "crying wolf" effect, where the manual team begins to ignore upgrades, ultimately weakening the upgrade mechanism.
C) All edge cases should be upgraded to ensure safety
D) The cost of upgrading labor is very low, so upgrades should be frequent.

> **Answer: B**
> Explanation: The upgrade mechanism relies on trust. If agents frequently escalate trivial issues unnecessarily, human teams will suffer burnout and real issues may go unnoticed. Upgrade standards should be precisely calibrated, not excessive.

**Q28.** When designing upgrade criteria, what two different triggers should you distinguish?

A) Speed and accuracy
B) User sentiment and agent confidence
C) Cognitive risk (the agent is unable to determine the correct action) and behavioral risk (the action is irreversible or high-consequence)
D) Simple problems and complex problems

> **Answer: C**
> Explanation: This is the core framework of the upgrade design. Cognitive risk concerns the limits of an agent's ability to understand, and behavioral risk concerns the consequences of actions. An operation may have high cognitive risk but low behavioral risk (upgrade optional), or low cognitive risk but high behavioral risk (upgrade required).

**Q29.** A permissions error occurred: The agent attempted to access the file but was denied. Agents should:

A) Prompt user permission error and try again with different credentials
B) Escalate to manual immediately because the permissions issue cannot be resolved by the agent
C) Retry multiple times to "get past" permission restrictions
D) Ignore the error and continue with other tasks

> **Answer: B**
> Explanation: Permission errors are a clear sign of an upgrade. The agent cannot resolve permission issues; this requires manual intervention (elevated permissions, data owner approval, etc.). Retrying wastes resources.**Q30.** When designing an upgrade protocol, what key information should be included in the upgrade message to support manual takeover?

A) Complete conversation history
B) Reason for upgrade, current status, attempted solutions, and agent's recommended next steps
C) Only user original request
D) Agent’s internal logs and debug information

> **Answer: B**
> Explanation: An effective upgrade protocol provides enough context for humans to get started quickly (why the upgrade is needed, what the current situation is, what has been tried), while not overloading (avoiding a complete history). The agent's suggested next steps help humans make quick decisions.

**Q31.** In a multi-level escalation system (Agent -> Junior Labor -> Senior Labor -> Manager), what are the best practices regarding escalation guidelines?

A) Each level uses different standards, becoming more and more stringent
B) Use the same standards for all levels to maintain consistency
C) No level is set, all upgrades go directly to the administrator
D) Let everyone decide for themselves whether they should upgrade

> **Answer: A**
> Explanation: Tiered upgrade requires clear grading standards. Junior staff handles FAQs and clarifications, senior staff handles technical or policy issues, and managers handle exceptions and policy decisions. The promotion guidelines for each level should reflect this differentiation.

**Q32.** What is the optimal strategy for agents when the upgrade channel is temporarily unavailable (for example, the human team is offline)?

A) Keep trying to solve the problem, even if the stakes are high
B) Tell the user to try again later
C) Implement graceful degradation: explain limitations to users, provide partial self-service or queuing options, collect information for manual processing later
D) Refuse to continue the conversation

> **Answer: C**
> Explanation: Graceful degradation ensures that the system remains useful when the escalation channel is unavailable. This involves transparent communication, providing all possible self-service alternatives, and having data ready to be processed as soon as the team recovers.

**Q33.** When designing an upgrade protocol, which of the following is most accurate regarding "level of authority"?

A) All upgrades should give humans full access
B) When upgrading, permissions should be temporarily elevated until the problem is solved.
C) Escalation should not automatically elevate privileges; privilege escalation itself should be a controlled decision
D) Permissions have nothing to do with upgrades

> **Answer: C**
> Explanation: Escalation and privilege escalation are two different issues. Escalation means moving to manual, but not automatically means elevation of privileges. Humans may need additional permissions to resolve issues, but this should be done as a separate decision, which may require approval.

**Q34.** When designing a manual handover protocol, if the upgrade message contains the user's personal information (such as real name, account ID), you should:

A) Include all information to provide complete context
B) Based on the manual minimum need-to-know principle, only necessary identification information is included, and sensitive data is accessed through secure channels.
C) Never include personal information, only general information
D) Encoding personal information to hide it

> **Answer: B**
> Explanation: The principles of least privilege and least need-to-know also apply to upgrade protocols. Humans need enough information to handle the issue, but not all personal data should be exposed in the upgrade message itself. Secure, controlled access is more appropriate.

**Q35.** After the upgrade is completed and handled by a human, how should agents resume conversations or learning?

A) Agents should no longer be involved and the process is completely taken over by humans
B) After the manual operation is completed, the results should be recorded and returned to the agent system for future reference, but without changing the core behavior of the agent
C) The agent should "learn" and update its rules for handling similar situations
D) The process ends without any feedback loop

> **Answer: B**
> Explanation: The results of the upgrade are valuable for system improvement, but should not be used to automatically change the agent's behavior rules. Instead, results should be recorded in a case library for analysis and potential future model improvements, rather than dynamically modifying upgrade criteria.

**Q36.** Which metric is most representative when evaluating the effectiveness of an upgraded system design?

A) Escalation rate (escalation as a percentage of all interactions)
B) Escalation manual resolution rate (percentage of escalated issues that were successfully resolved manually) and escalation necessity score (estimated percentage of issues that actually should have been escalated but were not)
C) Average upgrade processing time
D) User satisfaction score

> **Answer: B**
> Explanation: The effectiveness of the upgrade system lies in two aspects: high resolution rate (the upgrade does solve the problem) and low false negative rate (necessary upgrades are performed). Escalation rates alone are meaningless - a high escalation rate may indicate excessive escalation, a low escalation rate may indicate missing critical issues.

---

## Task 5.4 Error propagation and retry strategy (Q37–Q48)

**Q37.** An API call returned a 503 Service Unavailable error. According to error classification best practices, this error should be marked as:

A) Validation error - incorrect input
B) Permission error - lack of access rights
C) Retryable transient errors - should be retried using exponential backoff
D) Unrecoverable fatal error

> **Answer: C**
> Explanation: 503 is a typical transient error, indicating that the server is temporarily unavailable. Such errors should be marked with isRetryable=true and retried using exponential backoff (1s, 2s, 4s, 8s...). Validation or permission errors should not be retried.**Q38.** You received an error object `{code: "INVALID_INPUT", isRetryable: false, message: "..."}` while processing an API response. Agents should:

A) Ignore the isRetryable flag and decide whether you should try again.
B) Respect isRetryable=false and do not retry, but instead upgrade or return an error to the user
C) Make a retry and then give up
D) Continue to retry until successful

> **Answer: B**
> Explanation: The isRetryable flag is the API provider's explicit statement about error recoverability. It is best practice to respect this signal rather than overriding it. If the API says retrying is not possible, the agent should handle the failure rather than waste resources.

**Q39.** A multi-step data processing pipeline failed at step 3. Steps 1 and 2 have been successful and produced results. The best approach on how to handle this situation is:

A) Discard the results of steps 1 and 2 and start from scratch
B) Save partial results from steps 1 and 2 and include them when upgrading or reporting to help humans understand what happened
C) Continue to step 4, ignoring the failure of step 3
D) Retry steps 1, 2, 3 even if steps 1 and 2 were not the problem

> **Answer: B**
> Explanation: Some results are valuable for diagnosis and recovery. Saving intermediate results and including them in upgrades or bug reports can help humans quickly identify problem spots and consider alternative paths.

**Q40.** What are the main reasons why you should set a maximum retry delay (e.g. 32 seconds) when implementing an exponential backoff retry strategy?

A) Save network bandwidth
B) Prevent retry delays from becoming so long that users will assume a complete system crash while giving the service ample time to recover
C) Because retrying after 32 seconds is invalid
D) Required by API documentation> **Answer: B**
> Explanation: The goal of exponential backoff is to give failed systems time to recover, but unlimited growth can lead to user experience issues and unnecessary delays. Maximum delay (usually 30-60 seconds) provides a balance between the two.

**Q41.** Your system uses a dead letter queue (DLQ) to capture repeated failed tasks. Which of the following is most accurate regarding the purpose of DLQ?

A) DLQ is where permanently failed tasks are stored and no longer need to be processed.
B) DLQ is an observation window used to identify system problem patterns and candidate tasks for human review
C) DLQ will automatically resolve the issue
D) DLQ should be avoided and any failure should cause an immediate system crash

> **Answer: B**
> Explanation: DLQ is an observability and failure recovery tool. It doesn't automatically resolve any issues, but it allows the system to continue running while providing a human with a list to review. Patterns in the DLQ can indicate system problems that need to be fixed.

**Q42.** In a complex workflow that relies on multiple external APIs, the first API call succeeds, but the second API call fails due to rate limiting (429 error). Regarding handling this error, the best practice is:

A) Immediately retry the second API call
B) Retry the second API using exponential backoff, but not the first; also consider implementing rate limit-aware backoff (if the 429 response header Retry-After is provided)
C) Both APIs retry
D) Downgrade to calling alternative API

> **Answer: B**
> Explanation: Rate limiting (429) requires special handling. If there is a Retry-After in the response header, it should be respected instead of using the standard exponential backoff. Also, the first API that has succeeded should not be retried. This demonstrates refined error handling.

**Q43.** In the case of a cascading failure (A calls B, B calls C, C fails), how should errors be propagated up the call chain?

A) Abort immediately at level A with original error message
B) Add context information at each level ("C failed because X, B was unable to complete Y, and the operation of A failed") to form an error chain or nested error object
C) Completely hide the underlying error and only report "Operation failed"
D) Attempt local fixes at each level without propagating upwards

> **Answer: B**
> Explanation: Error chains preserve the complete causal path, which is critical for diagnosis and repair. Each level adds its own context (what was tried, why it failed), forming a traceable error history. This is a modern best practice for error handling.

**Q44.** A task fails on the first retry and succeeds on the second retry. This is called "fleeting failure". How should such failures be handled in long-term system design?

A) Consider them a success - the problem fixed itself
B) Record them in the log but take no further action
C) Log them as "Exceptions requiring investigation" as frequent fleeting failures may indicate underlying resource issues or race conditions
D) ignore them completely

> **Answer: C**
> Explanation: Single fleeting failures may be harmless, but their pattern may indicate a larger problem. If your API frequently succeeds on retries, you may have intermittent failures, resource contention, or load balancing issues. Monitoring these patterns is important for preventive maintenance.**Q45.** When designing a retry strategy, which of the following recommendations is the most reasonable regarding an upper limit on the number of retries?

A) No limit, try again until successful
B) Fixed number of retries (e.g. 3 times) for all error types
C) Different limits based on error type: Transient errors more retries (5-7 times), permission errors 0 times, rate limit 2-3 times plus delay
D) Retry once, then give up

> **Answer: C**
> Explanation: Different error types merit different levels of retry investment. Transient errors may require multiple retries to succeed, whereas permission errors never change across retries. This differentiated strategy optimizes resource usage and recovery speed.

**Q46.** When handling errors, what key information should be included in error reports to support debugging and recovery?

A) Just an error message
B) Error message, error code, timestamp, context of operation (what to do when it failed), number of retries attempted and whether an upgrade should be made
C) Full stack trace
D) User-friendly messages

> **Answer: B**
> Explanation: Actionable error reports include diagnostic information (code, timestamp, number of retries) and context (what operation failed), but do not include technically unnecessary information (full stack trace). This allows engineers and automation systems to respond quickly.

**Q47.** When an operation supports idempotence (multiple executions with the same result), how does this change the retry strategy?

A) Idempotence does not affect retries
B) Idempotent operations allow for more aggressive retries because repeated executions do not change the state and only waste resources
C) Non-idempotent operations should avoid retrying, while idempotent operations can be retried with confidence to ensure success.
D) Idempotent operations should never be retried

> **Answer: C**
> Explanation: Idempotence is a key prerequisite for retrying. For idempotent operations, it is safe to retry - even repeated executions will produce the same results, without causing double billing, double creation, or other side effects. Retries of non-idempotent operations require more caution.

**Q48.** When designing long-running workflows, how should "retry fatigue" be dealt with - even if the user experience degrades due to constant failures and retries?

A) Keep retrying regardless of user experience
B) Implement a "polite failure" strategy: After a limited number of retries, notify the user of the situation, provide the option of manual intervention, or queue to wait, rather than retrying indefinitely
C) Give up immediately on first failure
D) Hide evidence of retries to make users think the system is running

> **Answer: B**
> Explanation: The balance between user experience and system reliability is critical. Infinite retries frustrate users; giving up immediately hurts usability. The middle path is to retry with limited effort and then transparently notify the user and provide alternative options.

---

## Task 5.5 Reliability design of long-running workflows (Q49–Q60)

**Q49.** A 48-hour data processing workflow suddenly crashes (e.g. process terminates unexpectedly) after 36 hours of running. Without checkpoints, an entire 36 hours of work is lost. Which of the following is most accurate regarding the checkpoint mechanism?

A) Checkpointing mechanism is unnecessary and should rely on the built-in reliability of the service
B) The status of the workflow, including completed steps, intermediate results, and current position, should be saved periodically (e.g., every 2-4 hours) to support recovery
C) Checkpoints should only be created when the workflow is complete
D) Checkpointing is only suitable for short-term workflows

> **Answer: B**
> Explanation: Checkpoints (or manifests) are the foundation for long-term workflow reliability. By periodically saving state, the system can recover to the last known good state after a failure without having to repeat all the work. The checkpoint interval should be chosen based on the cost of the job and the recovery objectives.

**Q50.** When implementing checkpoints, which of the following is the most complete regarding the state information that should be saved?

A) Save only the final result
B) Save the current workflow position, all completed intermediate results, configuration/parameters required for recovery, and any dynamically generated decisions (such as selected algorithm variants)
C) Save complete conversation history
D) No need to save anything, should be able to recalculate state

> **Answer: B**
> Explanation: A valid checkpoint contains everything needed for recovery. This includes progress information (where it stopped), non-recomputable results (such as the results of expensive API calls), and parameterized decisions. The complete conversation history is too large and recalculation may change the results (non-deterministically).

**Q51.** After the workflow is restored from a checkpoint, it is found that the saved intermediate results may no longer be valid due to code or dependency version changes. Regarding the best way to handle this situation:

A) Trust the saved result and continue
B) Implement version tracking: record code version, library version and other important dependencies in checkpoints, and trigger verification or recalculate the path if the version has changed during recovery
C) Always discard saved results and recalculate
D) Continue using the results but add a warning to the log

> **Answer: B**
> Explanation: Version compatibility is a key issue in checkpoint recovery. By tracking versions in checkpoints, the system can detect potential incompatibilities upon recovery and take action (validate results, mark for review, or recalculate). This prevents continuation of work based on obsolescence.**Q52.** Regarding the --resume flag of Claude Code, what is its function?

A) Resume the last interrupted agent session
B) Allow workflow to be restarted from the previous checkpoint, preserving the saved state
C) Clear all previous state and start from scratch
D) Pause the workflow and prevent it from continuing

> **Answer: B**
> Explanation: The --resume flag is the implementation of the checkpoint/recovery mechanism. It tells Claude Code to continue from the last save point rather than start as a new, independent run.

**Q53.** When using fork_session to create a checkpoint, which of the following best describes the operation?

A) fork_session creates a new independent branch of the workflow, the original session continues to run, and the new branch can be used for experiments
B) fork_session ends the current session and starts a new session
C) fork_session cannot be used to create checkpoints
D) fork_session only copies files, not the conversation context

> **Answer: A**
> Explanation: fork_session allows forking of sessions at critical points, which is useful for "should I try this risky operation?" situations. Branch sessions can conduct experiments independently, and if successful, their results can be merged back; if failed, the main thread is not affected.

**Q54.** The configuration parameters of a long-lived workflow changed during operation (e.g. due to external events). Regarding state management, it should be:

A) Ignore the parameter changes and continue to use the original parameters
B) Stop immediately and start from the beginning, using new parameters
C) Record parameter changes at the next checkpoint, use the new parameters when recovering, but mark which work was done with the old parameters
D) Continue running and manually adjust the results later> **Answer: C**
> Explanation: This involves workflow integrity and auditing. If parameters change, the point of change and the reason should be documented and the new parameters applied upon recovery. However, it should be clearly marked which results were generated under which parameters to support later verification.

**Q55.** When designing idempotent operations (safe to retry), which of the following statements is most accurate?

A) All operations are idempotent
B) Only read operations are idempotent
C) Idempotent operations mean that executing them multiple times has the same end result as executing them once, such as "set this file to content X" (as opposed to "append to file")
D) Idempotence cannot be achieved and retries should be avoided

> **Answer: C**
> Explanation: Classic examples of idempotent operations include "set state", "create a resource with a specific ID" (return if it already exists). In contrast, "append" or "increment counter" are non-idempotent. Designing idempotent operations is key to long-term workflow reliability.

**Q56.** A multi-step workflow consists of an expensive but idempotent step (e.g., calling a generative model for analysis) and a subsequent non-idempotent step (e.g., appending the results to a log). What should be done during recovery?

A) Re-execute all steps
B) Skip expensive steps and continue from last saved result
C) Re-execute only non-idempotent steps
D) Use randomness to decide whether to re-execute

> **Answer: B**
> Explanation: This is the optimal way to utilize checkpoints. Expensive but idempotent steps should be skipped (results already in the checkpoint) and revert directly to non-idempotent steps. This saves computing resources.

**Q57.** When designing an external state store (such as a database) to track the progress of a long-term workflow, which of the following is most important?

A) Size of state storage
B) Atomicity and consistency: Status updates should be atomic operations to prevent partial updates from causing inconsistent states
C) Storage speed
D) Compression rate

> **Answer: B**
> Explanation: If state updates are not atomic, the workflow may crash while the state store is in an intermediate state, resulting in inconsistent data on recovery. Database transactions, atomic file operations, or other atomic mechanisms are critical to ensuring data integrity upon recovery.

**Q58.** Regarding long-term workflow monitoring, which of the following best indicates workflow reliability issues?

A) CPU usage
B) Checkpoint failure rate, recovery success rate and average time between checkpoints
C) Memory consumption
D) Network delay

> **Answer: B**
> Explanation: These indicators directly reflect the resilience of the workflow. A high checkpoint failure rate means that progress cannot be saved; a low recovery success rate means that even if it is saved, it cannot be restarted; and a long checkpoint interval means that a single failure has a large impact.

**Q59.** When performing fault injection testing (chaos engineering) to verify the reliability of long-term workflows, what scenarios should be tested?

A) Only test network failures
B) Inject failures at various stages (process crash, database connection failure, file system failure) and verify that the system can be recovered without losing progress or creating an inconsistent state
C) No fault testing, relying on the production environment for learning
D) Only test for failures in the last step

> **Answer: B**
> Explanation: Fault injection testing should cover different types of faults and different time points. By intentionally injecting faults into a test environment, problems in checkpointing strategies, recovery logic, or state management can be discovered and fixed before they appear in production.

**Q60.** When designing a workflow that needs to run reliably for 30 days, which of the following is the most comprehensive best practice for checkpointing strategies?

A) Create a checkpoint every hour
B) Create multiple levels of checkpoints based on the cost of the step and the importance of recovery: micro checkpoints (every 15 minutes) for high-cost operations, macro checkpoints (every 4 hours) for overall progress, plus a clear state versioning and recovery testing plan
C) Create a checkpoint every day
D) No checkpoints required, 30 days of work should be 100% reliability

> **Answer: B**
> Explanation: This reflects actual, mature workflow design. Multi-level checkpointing optimizes cost/recovery trade-offs for different priorities. Micro checkpoints every 15 minutes protect expensive operations, and macro checkpoints every 4 hours prevent long-term drift. Version control and testing ensure that recovery mechanisms actually work. Such a strategy is necessary to prevent large-scale losses during the 30-day run.

---

## Task 5.1 (Supplemental) Chunking and compression techniques (Q61–Q62)

**Q61.** You are building a pipeline that processes legal contracts too large to fit in a single context window. You split each contract into sequential non-overlapping 4,000-token chunks. During testing, you notice that conclusions that span a chunk boundary are consistently misattributed or missed. What is the most appropriate fix?

A) Increase the model's context window by switching to a larger model
B) Reduce chunk size so that fewer conclusions span boundaries
C) Use overlapping chunks, where each chunk shares a portion of tokens with the adjacent chunk, so boundary content appears in both
D) Post-process results to merge conclusions across chunk outputs

> **Answer: C**
> Explanation: Overlapping chunks (typically 10–20% overlap) solve the boundary problem directly. When content that spans a split point is included in both the preceding and following chunks, the model has full context to interpret it correctly. Reducing chunk size (B) just shifts the boundary problem; it does not eliminate it. Post-processing (D) is a downstream workaround that still depends on boundary content being intact in at least one chunk.

**Q62.** A research team asks you to summarize a collection of 200 academic papers into a single executive brief. Attempting a one-pass summary of all 200 papers concatenated exceeds the context window and loses critical nuance. What is the most appropriate architectural approach?

A) Randomly sample 20 papers and summarize only those
B) Use hierarchical summarization: summarize each paper individually, then combine the per-paper summaries into a final synthesis
C) Ask the model to skip methodology sections to fit more papers in one pass
D) Use the /compact command to compress all 200 papers before summarizing

> **Answer: B**
> Explanation: Hierarchical summarization processes in two stages: first produce a focused summary of each paper (preserving key claims, evidence, and source attribution), then combine those section-level summaries into a final high-level synthesis. This approach retains more structure and traceability than a single-pass compression, and each conclusion remains traceable to its source paper. Random sampling (A) introduces selection bias. Skipping sections (C) discards potentially critical content. /compact (D) is a session-management tool, not a document processing architecture.

---

**Answer sheet suggestions**:
- Completion time target: 75 minutes (1.25 minutes per question)
- Scoring criteria: 56-60 correct questions = outstanding; 48-55 questions = excellent; 40-47 questions = passing; less than 40 questions = recommended to review domain knowledge