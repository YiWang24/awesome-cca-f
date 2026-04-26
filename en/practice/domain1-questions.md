# CCA-F Simulation Questions — Domain 1: Agent Architecture and Orchestration
# 60 questions total | weight 27% | with detailed explanations

---

> **Task Statement Distribution:**  
> Task 1.1 (agent loop): Q1–Q9
> Task 1.2 (Multi-Agent Coordination): Q10–Q18
> Task 1.3 (Subagent Invocation and Context): Q19–Q27
> Task 1.4 (Multi-step workflow enforcement): Q28–Q35
> Task 1.5 (SDK hooks): Q36–Q43
> Task 1.6 (Task Decomposition Strategy): Q44–Q51
> Task 1.7 (Session State and Forking): Q52–Q60

---

## Task 1.1: Agent Loop (Q1–Q9)

---

**Q1.** You are implementing an agent loop using the Claude API. Claude returned a response containing `stop_reason: "tool_use"`. What should your code do?

A) Return the response to the user because Claude has already provided the answer
B) Execute the requested tool, append the results to the conversation history, and call the Claude API again
C) Resend the exact same request because the response is incomplete
D) Increase `max_tokens` and try again as the model may be truncated

> **Answer: B**
> Explanation: `"tool_use"` means Claude requested to call one or more tools. Correct flow: execute the tool → append the result to the history as a `tool_result` message → call the API again. This is the core mechanism of the agent loop. A is incorrect: Claude has not completed the task. C is incorrect: Resending the same request results in an infinite loop. D is incorrect: `max_tokens` has nothing to do with stop_reason of `tool_use`.

---

**Q2.** Which of the following agent loop implementation patterns is an **anti-pattern** (should be avoided)?

A) Continue looping when `stop_reason == "tool_use"` and terminate when `stop_reason == "end_turn"`
B) Append tool results as `tool_result` messages to the conversation history between iterations
C) Check whether the text content of Claude's response contains phrases such as "task completed" or "I have completed" to decide to terminate
D) Update the message history after each tool invocation, allowing the model to infer the next action

> **Answer: C**
> Explanation: Parsing natural language signals to determine loop termination is an anti-pattern. Claude's output is unpredictable natural language, it may not use these specific phrases, or it may use them in intermediate steps. The correct termination condition is `stop_reason == "end_turn"`.

---

**Q3.** A developer implemented the following agent loop:
```python
for i in range(50):
    response = client.messages.create(model=model, tools=tools, messages=messages)
    if response.stop_reason == "end_turn":
        break
    # Execute tools and update history...
```
What is the main issue with this implementation?

A) The upper limit of the number of cycles (50 times) is too low and should be increased to 1000 times
B) Using a `for` loop instead of a `while` loop is a fundamental architectural error
C) Using an arbitrary iteration cap as the primary stopping mechanism is an anti-pattern, but here it is a secondary guard and the primary stopping is still the `end_turn` check, which is acceptable
D) Missing explicit check for `stop_reason == "tool_use"`

> **Answer: C**
> Explanation: This implementation is actually **acceptable** because the main stopping condition is the correct `end_turn` check, and the 50 iteration cap is just a safety guard against infinite loops. The anti-pattern in the question is "use arbitrary iteration caps as the **primary** stopping mechanism" - which is not the case here. D also makes some sense, but does not affect functional correctness, because the code can handle when stop_reason is neither end_turn nor tool_use (like max_tokens).

---

**Q4.** In an agent loop, why do tool results have to be appended to the conversation history between each iteration?

A) Because the Claude API is stateful, it automatically remembers previous calls
B) Because the Claude model has no built-in memory, each API call requires the full context to infer the next action
C) Because the API requires the format of the tool results
D) Because the tool results are used to update the model's weights

> **Answer: B**
> Explanation: Claude API is **stateless** - each call is independent. The model has no built-in memory for previous calls. The complete conversation history, including tool calls and results, must be passed in every request so that the model can infer next steps based on previously discovered information.

---

**Q5.** Your agent calls a database query tool, which returns a large JSON object containing 200 fields. What are the main risks of appending this complete object to the conversation history?

A) Claude cannot parse JSON format
B) Large tool results accumulate in the context, and the tokens consumed are disproportionate to their information value, which may cause the context window to overflow.
C) Claude will ignore tool results above a certain size
D) Database responses must always be converted to natural language

> **Answer: B**
> Explanation: Large tool results (such as 40+ fields returned per query) accumulate in context, consuming far more tokens than actually needed. Best practice is to prune tool results to only relevant fields before appending to history, thus preserving context space for more conversation turns.

---

**Q6.** Which of the following `stop_reason` values indicates that Claude has completed the task and is ready to present the final response to the user?

A) `"tool_use"`
B) `"max_tokens"`
C) `"end_turn"`
D) `"stop_sequence"`

> **Answer: C**
> Explanation: `"end_turn"` means that Claude has completed the current task and generated the final response. This is the normal termination signal for the agent loop. `"tool_use"` indicates that a tool call is required (to continue the loop). `"max_tokens"` indicates that the output is truncated (an error condition). `"stop_sequence"` indicates that a stop sequence has been encountered.

---

**Q7.** What is the main difference between model-driven decision-making (Claude infers which tool to call based on context) versus pre-configured decision trees?

A) Model-driven decision-making is always slower because it requires more inference time
B) Preconfigured decision trees provide a deterministic order of tool invocations, while model-driven decision making allows Claude to dynamically adapt based on intermediate results
C) Model-driven decision-making is only suitable for simple tasks, complex tasks must use pre-configured decision trees
D) Preconfigured decision trees and model-driven decision making are functionally equivalent

> **Answer: B**
> Explanation: The preconfigured decision tree enforces a fixed order of tool calls (deterministic), suitable for scenarios that require strict compliance (such as financial operations that must first verify identity). Model-driven decision-making allows Claude to dynamically decide next steps (adaptability) based on the current context and intermediate results, suitable for tasks with high ambiguity.

---

**Q8.** In the implementation of an agent loop, the developer did not append Claude's `"assistant"` message (including the tool call request) to the history, only the tool results. What problems does this cause?

A) The API will reject the request because the message history format is invalid (assistant message and tool_result must be paired)
B) Claude ignores tool results and generates different responses
C) Performance will be reduced but functionality will not be affected
D) The model will request the execution of more tools to compensate

> **Answer: A**
> Explanation: The Claude API requires that the conversation history be well-formed: the `tool_result` message must be immediately followed by an `assistant` message containing the corresponding `tool_use` block. If the assistant message (containing the tool call request) is not appended, the API will return a format error because `tool_result` has no corresponding `tool_use` to associate with it.

---

**Q9.** When implementing agent loops, which approach best prevents an infinite loop if tool execution fails?

A) Set a very high iteration limit (such as 10,000 times)
B) Handle errors at the tool level and return structured error information to the model, while setting a reasonable maximum number of iterations as an auxiliary protection
C) Terminate the loop immediately when the tool fails and report the error to the user
D) Retry failed tool calls an unlimited number of times until successful

> **Answer: B**
> Explanation: The best practice is dual protection: (1) Structured error response at the tool level (allowing Claude to decide how to handle the failure, such as retrying, changing plans, or reporting to the user); (2) A reasonable maximum number of iterations as a safety protection (to prevent infinite loops in extreme cases). Reliance on either alone is insufficient.

---## Task 1.2: Multi-Agent Coordination (Q10–Q18)

---

**Q10.** In a Hub-and-Spoke multi-agent architecture, what is the main responsibility of the coordinating agent?

A) Directly execute all tool calls, and the sub-agent is only responsible for summarizing the results
B) Manage communication, error handling and information routing among all sub-agents, and is also responsible for task decomposition, delegation and result aggregation
C) Only serves as a messaging layer between users and sub-agents, without making any decisions.
D) Maintain a shared memory pool so that all sub-agents can directly read and write

> **Answer: B**
> Explanation: The coordination agent is the core of the hub-and-spoke architecture and is responsible for: task decomposition, sub-agent delegation, result aggregation, error handling, and information routing. It is the only agent that "knows the whole picture". Subagents run in isolated contexts and do not communicate directly with each other.

---

**Q11.** In a multi-agent research system, you find that the web search subagent correctly finds relevant articles, but the final report only covers a small subset of the research topic. The coordinator's log shows that it breaks down the topic "Impact of climate change on agriculture" into two subtasks: "Impact on wheat yield" and "Impact on soybean yield". What is the root cause?

A) The query of the network search sub-agent is not broad enough
B) The synth agent is not configured correctly
C) The task decomposition of the coordination agent is too narrow - only crop yields are considered, ignoring other important dimensions of climate change on agriculture (such as irrigation water, extreme weather events, planting seasons, agricultural labor, etc.)
D) The subagent uses the wrong information source

> **Answer: C**
> Explanation: This is a classic case of incomplete coverage due to too narrow task decomposition (corresponding to the knowledge points of the official sample question Q7). The sub-agents all performed their assigned tasks correctly. The problem is that the coordinator's decomposition itself is biased. The impact of climate change on agriculture extends far beyond crop yields.

---

**Q12.** In a multi-agent system, why should communication between sub-agents be routed through the coordinating agent instead of directly?

A) Direct communication is technically impossible
B) Ensure observability, consistent error handling and controlled information flow through coordinator routing to avoid sub-agents directly relying on each other's state
C) The coordinator needs to collect all communication records for auditing
D) Direct communication consumes excessive API quota

> **Answer: B**
> Explanation: Three major advantages of routing through the coordinator: (1) **Observability**: All communications pass through a single node, which is easy to monitor and debug; (2) **Consistent error handling**: The coordinator manages all failure conditions uniformly; (3) **Controlled information flow**: The coordinator decides which information is passed to which sub-agents to prevent information overload or leakage.

---

**Q13.** Your multi-agent research system produces a report, but upon review it is discovered that some topics are repeatedly researched by multiple sub-agents, while other topics are not covered by any sub-agent. Which design improvement would best solve this problem?

A) Add more sub-agents to improve coverage
B) Explicitly assign **different sub-topics or source types** to each sub-agent in the coordinator's Task Decomposition Strategy to minimize duplication
C) Let the sub-agents notify each other of the topics they have covered after completion
D) Use a shared database to track which topics have been studied

> **Answer: B**
> Explanation: The fundamental solution to duplication/omission is to explicitly allocate non-overlapping research scopes during the task decomposition phase of the coordinator. For example: sub-agent A is responsible for academic papers, sub-agent B is responsible for news reports, and sub-agent C is responsible for policy documents. This prevents duplication during design.

---

**Q14.** After the coordinating agent received the first output from the synth agent, the evaluation found that the report covered the main points but missed several key statistics. What's the most appropriate next step?

A) Accept output and present it to the user, perfect is the enemy of good
B) Completely discard the synthesis results and restart the entire research process
C) Implement an iterative optimization loop: identify gaps → delegate targeted supplementary queries to search and analysis subagents → re-invoke synthesis with new findings until coverage is sufficient
D) Add missing statistics directly in the coordinator

> **Answer: C**
> Explanation: This is exactly the application scenario of iterative optimization loop. The coordinator should not accept incomplete output, nor waste the cost of a full restart. The correct approach is to identify specific gaps based on existing results, commission targeted complementary studies, and then incorporate new findings into a revised synthesis.

---

**Q15.** In a multi-agent research system, the synthesizer agent started calling the web search API directly (bypassing the coordinator) to verify some claims during its synthesis. What are the main issues with this design decision?

A) This will increase API costs
B) Bypassing the coordinator destroys the observability and consistency of the information flow, making the system difficult to monitor, debug, and maintain; the synthetic subagent exceeds the scope of its role
C) Internet search results may duplicate existing research
D) This is technically feasible and there are no substantive issues

> **Answer: B**
> Explanation: All sub-agent communications should pass through the coordinator, which is an architectural principle. Synthetic subagents directly perform network searches: (1) destroying a single point of observation; (2) error handling becomes inconsistent; (3) the subagent crosses the boundaries of its expertise (synthesis should synthesize, not search); (4) debugging and tracing become difficult.

---

**Q16.** Which of the following scenarios is most suitable for adopting a dynamically adaptive multi-agent research architecture (rather than a fixed pipeline)?

A) Generate standardized sales reports for known products
B) Extract customer contact information in a fixed format
C) Research an emerging technology topic where relevant subfields and key sources emerge over the course of the research
D) Generate status reports with the same structure daily

> **Answer: C**
> Explanation: Dynamic adaptive architectures are more suitable when the scope and structure of the task evolve during the research process. The fixed pipeline is suitable for tasks with a clear structure and fixed output format (A, B, D). For open research (C), it is not known in advance which sub-agents and research paths are needed, and dynamic decomposition is more efficient.

---

**Q17.** You find that the coordinating agent always calls all four sub-agents in a fixed order of "Search→Analyze→Synthesis→Report", even for very simple queries. What is the core problem with this design?

A) The number of sub-agents is inappropriate, fewer sub-agents should be used
B) Enforce full pipeline for all queries regardless of query complexity - the coordinator should analyze the query requirements and dynamically decide which sub-agents to call
C) Fixed order makes it difficult for the system to expand new sub-agents
D) There is no problem with this design, a consistent pipeline simplifies system maintenance

> **Answer: B**
> Explanation: The core value of the coordination agent lies in **intelligent choice**. For simple fact queries, you may only need Search → Report; for cached topics, you may only need Synthesis → Report. Forcing all queries to go through a full pipeline wastes resources and is a misuse of the coordinator role.

---

**Q18.** In a multi-agent system, the coordinating agent finds that the coverage of some areas is "not deep enough" after receiving reports from the synthetic sub-agents. The coordinator decides to call the search subagent again to get more information. Which of the following would best make this supplemental search effective?

A) Send exactly the same search task as the first time
B) Instruct the search sub-agent to randomly search for more keywords
C) Provide **specific coverage gaps** (e.g. "missing statistics for 2020-2024", "no data for Asian markets") to the search sub-agent, using targeted queries
D) Increase the max_tokens of the search subagent so that it returns longer results

> **Answer: C**
> Explanation: The effectiveness of an iterative optimization loop depends on the accuracy of re-delegation. Coordinators should convert specific gaps assessed into targeted complementary queries rather than repeating previous searches. This enables sub-agents to fill specific knowledge gaps in a targeted manner.

---

## Task 1.3: Sub-agent calling and context transfer (Q19–Q27)

---

**Q19.** In the Claude Agent SDK, what must be included in the `allowedTools` configuration for the coordinating agent to spawn child agents?

A) `"SubAgent"`
B) `"Task"`
C) `"Spawn"`
D) `"Delegate"`

> **Answer: B**
> Explanation: The `Task` tool is a special mechanism for generating sub-agents in Claude Agent SDK. The coordinating agent's `allowedTools` must contain `"Task"` in order to call the child agent. This is a specific configuration requirement that appears directly in the exam content.

---

**Q20.** Your research coordination agent calls a document analysis subagent and wants it to synthesize findings from a web search. The subagent completed its analysis and returned output based only on the initial document it was assigned, without integrating the web search results. What is the most likely root cause?

A) Subagent selection ignores web search results
B) Network search results are not provided directly in the child agent's prompt - the child agent does not automatically inherit the parent context or share memory between calls
C) The subagent needs access to web search tools to integrate those results
D) The coordinating agent needs to create a shared memory space between the two sub-agents

> **Answer: B**
> Explanation: This is the core issue of sub-agent context isolation. Child agents do not automatically inherit the coordinator's conversation history. Web search results must be explicitly included in the hints passed to the document analysis subagent. If integration is desired, the coordinator must explicitly pass all relevant context.

---

**Q21.** When designing subagent context delivery, which of the following practices best preserves information attribution (knowing where each finding comes from)?

A) Consolidate all findings into a long text paragraph and pass it to the next subagent
B) Use structured data formats to separate content from metadata (source URL, document name, page number, publication date)
C) Pass only the summary of the discovery to save context space
D) Describe the findings in natural language and mention the source names

> **Answer: B**
> Explanation: Structured data formats (such as JSON) ensure that content and source metadata are bound together and do not become separated during composition. Plain text paragraphs (A) make source information difficult for machines to process. Abstracts only (C) may lose precise original source information. Natural language mentions (D) are difficult to reliably extract in subsequent processing.

---

**Q22.** You want to coordinate the agent to launch multiple sub-agents at the same time to shorten the total research time. How to achieve parallel execution?

A) Send Task tool calls separately in multiple API calls and Claude will understand the need for parallel execution
B) Issuing multiple Task tool calls in a single coordinator response - this results in parallel execution rather than sequential execution
C) Use the `parallel: true` parameter to mark Task tool calls
D) Create an independent coordinator instance for each sub-agent

> **Answer: B**
> Explanation: Parallel subagent execution is achieved by issuing multiple Task tool calls in a single coordinator response. If these calls are spread across separate turns, they will be executed sequentially; multiple Task calls within a single turn will be executed in parallel, significantly reducing overall latency.

---

**Q23.** What is the main purpose of `AgentDefinition` configuration?

A) Define the training data and fine-tuning parameters of the agent
B) Specify descriptions, system prompts, and tool restrictions for each subagent type
C) Configure the agent’s API rate limit
D) Define the communication protocol between agents

> **Answer: B**
> Explanation: `AgentDefinition` allows you to define different types of sub-agents: (1) Description (helps the coordinator decide when to use it); (2) System prompts (defines the behavior and expertise of the sub-agent); (3) Tool restrictions (scoped tool access, only gives the tools the sub-agent needs).

---

**Q24.** What is the main purpose of fork-based session management?

A) Back up current session state in case of crash
B) Create independent branches from a shared analysis baseline to explore different approaches in parallel (e.g., explore two different refactoring strategies simultaneously based on the same code base analysis)
C) Split large sessions into multiple smaller sessions to improve performance
D) Create an archived copy of the session for auditing purposes

> **Answer: B**
> Explanation: Fork sessions are used for **forked exploration** - starting from a shared analysis starting point and creating independent experimental branches without interfering with each other. Classic usage scenario: You have completed the code base analysis and now want to compare two different refactoring solutions. You can fork two independent exploration branches from the same analysis baseline.

---

**Q25.** When designing the coordination prompts of a sub-agent, which of the following methods best achieves the adaptability of the sub-agent?

A) Provide detailed step-by-step procedural instructions on what specific actions the sub-agent should perform
B) Specify research goals and quality standards, rather than specific operational steps, and let the sub-agent decide how best to achieve the goals
C) Provide minimal context to preserve the token and let the sub-agent judge for itself
D) Copy the coordinator's complete system prompt into the child agent's prompt

> **Answer: B**
> Parse: Goal-oriented prompts ("Research topic The sub-agent can adjust its strategy according to the actual situation encountered.

---

**Q26.** You want to deliver research findings from web searches and document analysis to a synth agent, while retaining the association of each finding with its source. Which of the following is most appropriate?

A) Concatenate all findings into a string with a list of sources at the end
B) Simplify findings into bullet points, mentioning sources in parentheses
C) Use a structured format with each finding containing claim content, evidence excerpts, source URL/document name and publication date
D) Pass only the findings required for synthesis, omitting source information to save tokens

> **Answer: C**
> Explanation: Structured claims-source mapping maintains traceability throughout the composition process. This enables the synth agent to retain citations in the final report, allowing readers to verify claims, and for conflicts between sources to be unambiguously identified.

---

**Q27.** In your multi-agent system design, the coordinating agent encounters context window limitations when passing context to the synthesizer agent. Which of the following strategies best solves this problem?

A) Upgrade to a model with a larger context window
B) Truncate the original output from the subagent and keep only the last few segments
C) Reduce the amount of data passed to the synthetic agent by requiring upstream sub-agents (search, analysis) to return structured data (key facts, citations, relevance scores) rather than detailed content and reasoning chains
D) Increase the number of subagents, each processing smaller fragments

> **Answer: C**
> Explanation: When the downstream agent has a limited context budget, the output format of the upstream agent should be modified. Requiring the return of structured summaries (key facts, quotes, relevance scores) rather than lengthy chains of content and reasoning can significantly reduce the amount of data passed while retaining key information.

---

## Task 1.4: Multi-step workflow enforcement (Q28–Q35)

---

**Q28.** When "deterministic compliance" is required in a workflow, which enforcement mechanism is the right choice?

A) Explain the sequence of operations in great detail in the system prompts
B) Add multiple few-shot examples demonstrating the correct workflow sequence
C) Implement programmatic prerequisites (hooks, prerequisite gates) to prevent downstream operations until prerequisite steps are completed
D) Use more powerful models to improve compliance with prompt instructions

> **Answer: C**
> Explanation: When the workflow requires deterministic compliance (for example, financial operations must first verify identity), the prompt instruction method (A, B) has a non-zero failure rate - unacceptable. Programmatic preconditions provide truly deterministic guarantees: if `get_customer` does not return a verified customer ID, the system prevents `process_refund` from being called, no matter what Claude wants to do.

---

**Q29.** Production logs show that in 12% of cases the customer support agent processed refunds without first calling `get_customer` to verify identity. The operations team requested that this rate be reduced to 0%. Which solution would achieve this goal?

A) Expand the description of the verification steps in the system prompt from two sentences to a full paragraph
B) Add 10 few-shot examples, all showing verifying customer identity before refunding money
C) Implement a Hook to prevent `process_refund` calls at the model level unless the system has confirmed that `get_customer` has been successfully executed and returned a verified customer ID
D) Customers are required to reconfirm their account information before each refund is processed.

> **Answer: C**
> Explanation: Requiring 0% failure rate means that deterministic mechanisms must be used. The failure rate of hint and few-shot methods (A, B) may be low but will never be 0% (LLM is probabilistic). Only programmatic execution can provide a true 0% failure rate guarantee.

---

**Q30.** What is a "structured handoff protocol" and why is it important in a customer support agent?

A) A technical protocol for transferring conversations between different AI models
B) When the agent is upgraded to a human, a summary document containing customer details, root cause analysis, attempted actions, and recommended actions allows the human agent to quickly understand the situation without having to access the full conversation record
C) A security standard for encrypted transmission of customer data
D) SLA agreements that define response times between different customer support tiers

> **Answer: B**
> Explanation: When the agent cannot solve the problem and needs to be upgraded, the human agent usually does not have the time or permission to read the full conversation transcript. Structured handover summary (customer ID, root cause of issue, attempted solutions, recommended actions) makes handoffs seamless and efficient. This is an important component when designing multi-step workflows.

---

**Q31.** A customer sends a message with three questions: a refund request, an account information change, and an inquiry about a product. Which of the following treatments is most appropriate?

A) Only handle the most important issues (refunds) and let the customer resend other issues
B) Address the problems one by one in order of complexity, focusing on one at a time
C) Break down multi-concern requests into distinct actions, investigate each issue in parallel using shared authenticated customer context, and then synthesize a unified response
D) Forward the entire message to a human agent because multi-issue requests are too complex

> **Answer: C**
> Explanation: Correct multi-concern request handling: (1) perform customer verification all at once (shared context); (2) investigate each individual issue in parallel (refund eligibility, account change permissions, product information); (3) synthesize all findings into a single unified response. This is both efficient and provides a complete service.

---

**Q32.** What is the fundamental difference between programmatic prerequisites and prompt-based guidance?

A) Programmed prerequisites are easier to implement and prompt guidance requires more engineering work
B) Programmed preconditions enforce the order of operations at the code level (deterministic), prompting the dependent model to follow instructions (probabilistic, with the possibility of failure)
C) Prompt guidance is more reliable for all workflows, and programmatic preconditions are only used in extreme cases
D) Both are functionally equivalent, and the choice depends on developer preference

> **Answer: B**
> Explanation: This is the core difference in Task 1.4. Programmatic preconditions are implemented at the application code level to enforce constraints regardless of LLM decisions. Tip: Bootstrapping relies on LLM to follow natural language instructions, and even if the instructions are very detailed, there is still a possibility of probabilistic failure.

---

**Q33.** Your customer support system needs to ensure that the customer's identity has been verified by two factors (password + verification code) before processing any account operations. Which implementation is the most reliable?

A) Indicate in the system prompt "Always verify the customer's dual identity before any account operation"
B) Add a few-shot example to show the complete two-step verification process
C) Implement a state machine to track the verification status and prevent all account operation tools from being called at the code level until the password verification and verification code are passed.
D) Add the description "requires two-factor verification" to the description of each account operation tool

> **Answer: C**
> Explanation: Two-factor authentication is a security-critical business rule that requires deterministic guarantees. The state machine tracks the verification status (`password_verified`, `otp_verified`), and the account operation tools are only available at the code level when both states are true. This is the only way to provide a true guarantee.

---

**Q34.** Which of the following elements is most critical when designing handoff summaries for complex workflows?

A) Complete conversation transcript
B) Technical architecture diagram of the system
C) Customer ID, root cause of problem, list of solutions attempted, refund amount (if applicable), recommended follow-up actions - allows the receiving agent to understand the situation without having to access the full conversation
D) Customer’s complete account history

> **Answer: C**
> Explanation: A valid handover summary focuses on the information that the receiving agent needs immediately to continue processing. The full conversation transcript (A) is too long and contains a lot of irrelevant information. Technical architecture diagram (B) has nothing to do with customer support. Full account history (D) can be viewed through the tool and does not need to be included in the summary.

---

**Q35.** When a customer's request requires multiple steps to process, and there are dependencies between some steps (for example, the existence of the order must be confirmed before a refund can be processed), which design pattern is most suitable?

A) Combine all steps into one large tool call
B) Use sequential tool calls, calling Claude once for each step and passing in the results of the previous step.
C) Implement precondition gating - only after verifying that the preconditions are met at the code level (such as order existence confirmation), dependent tools (such as refund processing) are allowed to be called.
D) Execute all steps in parallel, ignoring dependencies to increase speed

> **Answer: C**
> Explanation: Dependent steps require precondition gating. For critical business logic (confirm order existence → then process refund), enforcing dependencies at the code level is more reliable than relying on model judgment. This prevents errors like "Processing refund when order does not exist".

---

## Task 1.5: Agent SDK hook (Q36–Q43)

---

**Q36.** When is the `PostToolUse` hook triggered?

A) After the user sends the message but before Claude responds
B) After tool execution is complete, but before Claude (the model) processes the tool results
C) After Claude generates the response, but before the response is returned to the user
D) Before the tool is called (as a pre-call interception)

> **Answer: B**
> Explanation: The `PostToolUse` hook is triggered after the tool execution is completed and before Claude processes the results. This timing makes it ideal for transformations (such as format normalization) on tool results, ensuring that the model always receives data in a consistent format, regardless of the format returned by the underlying tool.

---

**Q37.** You have three MCP tools, each returning dates in different formats: Unix timestamp, ISO 8601 string, and "YYYY/MM/DD" format. You need to make sure that Claude always handles uniformly formatted dates. What's the best solution?

A) Modify the implementation of all three MCP tools so that they all return the same format
B) Explain in the system prompt "All dates are processed in ISO 8601 format, please convert before use"
C) Implement the `PostToolUse` hook, check the output of each tool and normalize the date into a unified ISO 8601 format, and then pass the result to Claude
D) After each tool call, ask the user to confirm whether the date format is correct

> **Answer: C**
> Explanation: The `PostToolUse` hook is designed for this kind of scenario: transforming tool results before they reach the model. A Works but requires modifications to external tools that may not be under your control. B relies on the model to understand and perform the transformation correctly, which is unreliable. Hooks provide deterministic data normalization and are the right architectural decision.

---

**Q38.** In which of the following situations is it best to use a tool call interception hook (instead of the `PostToolUse` hook)?

A) Format tool results before the model processes them
B) Log all tool calls for auditing
C) Block specific tool calls based on business rules (e.g. amount exceeds a threshold) before the tool call is actually executed
D) Convert tool call results into different data structures

> **Answer: C**
> Explanation: The tool call interception hook is triggered before the tool is executed, which is suitable for scenarios where certain operations need to be blocked. For example, when the model attempts to call `process_refund`, the interception hook checks the amount: if it exceeds $500, blocks the call and redirects it to `escalate_to_human`. A and D are the purposes of `PostToolUse` (result processing). B Both hooks can be implemented.

---

**Q39.** Your business rules require that all refunds over $500 must be manually approved. Which of the following implementations provides the strongest guarantee?

A) Add to system prompt: "Never automatically process refunds over $500, always escalate to manual approval"
B) Add a few-shot example to demonstrate the upgrade process for large refunds
C) Implement a tool call interception hook, check the amount when `process_refund` is called, and if it exceeds $500, block the call and automatically trigger the `escalate_to_human` tool
D) Ask Claude to confirm the amount in the response before processing any refunds

> **Answer: C**
> Explanation: When business rules require **guaranteed** compliance, programmatic hooks are the only reliable choice. Hints (A) and few-shots (B) are probabilistic - LLM cannot guarantee 100% compliance. The hook is executed at the code level, and refund calls that exceed the threshold will be intercepted regardless of Claude's decision.

---

**Q40.** What are the core trade-offs of using hooks (deterministic) rather than prompt instructions (probabilistic)?

A) Hooks are slower in speed, prompt commands are always faster
B) Hooks provide deterministic guarantees but require engineering implementation; prompt instructions are simple to implement but have non-zero failure rates and are not suitable for high-risk business rules
C) Prompt instructions are always more accurate because they take advantage of Claude's language understanding abilities
D) Hooks only apply to Claude Agent SDK, prompt instructions apply to all Claude products

> **Answer: B**
> Explanation: This is the core trade-off of Task 1.5. Hooks require engineering effort (coding, testing, maintenance) but provide deterministic compliance. Prompt instructions are quick and easy to implement, which is sufficient for non-critical preferences ("keep it simple") but not reliable enough for business-critical rules ("definitely not..."). Basis for selection: How serious are the consequences of rule violation?---

**Q41.** How does the return value of the `PostToolUse` hook affect Claude's subsequent behavior?

A) The return value of the hook will not affect Claude and is only used for logging
B) The (possibly modified) tool results returned by the hook replace the original tool output and become the actual data processed by Claude
C) The hook can only write data to the log and cannot modify the data passed to Claude.
D) After the hook is triggered, Claude will re-execute the original tool call

> **Answer: B**
> Explanation: The power of the `PostToolUse` hook is that it can modify the tool output. The hook receives the raw tool results, performs transformations (such as normalizing date formats, filtering irrelevant fields, adding calculated fields), and returns the modified results. This modified result is what Claude actually sees and infers.

---

**Q42.** You are building the Claude agent for a medical information system. System rules require that any information involving a prescription or diagnosis must be accompanied by a "ask a medical professional" disclaimer. How can I most reliably ensure that this statement always appears?

A) Clearly state in the system prompt that "all medical information responses must include a disclaimer"
B) After the response is generated, implement post-processing logic at the application level to detect medical-related content and attach a disclaimer
C) Use few-shot examples to demonstrate that all medical responses include a disclaimer
D) Ask users to confirm at the beginning of every conversation that they understand the information does not constitute medical advice

> **Answer: B**
> Explanation: For legal/compliance requirements (certain claims must appear), the most reliable approach is to force the addition in application-level post-processing, rather than relying on the model (both hints and few-shots are probabilistic). This is similar to the idea of ​​hooks: executed at the code level without relying on model judgment.

---

**Q43.** In the Agent SDK's hook system, which scenario illustrates why "deterministic guarantees" are necessary in some cases over "probabilistic compliance"?

A) When you want the agent to respond more politely
B) When irreversible operations are involved (such as actual execution of financial transactions, deletion of data), and the error will lead to irreparable consequences
C) When you want to improve the response speed of the agent
D) When you need the agent to support multiple languages

> **Answer: B**
> Explanation: Deterministic guarantees are required when: (1) the operation is irreversible (irreversible financial transactions, data deletion); (2) compliance is a legal requirement; (3) errors are extremely costly (such as sending a refund to the wrong account). For reversible operations or simply preference rules, probabilistic compliance (hints) is often sufficient.

---

## Task 1.6: Task decomposition strategy (Q44–Q51)

---

**Q44.** What is the main difference between Prompt Chaining and Dynamic Adaptive Decomposition?

A) Prompt chaining is only suitable for simple tasks, and dynamic decomposition is suitable for complex tasks.
B) Prompt chains use a fixed sequence of predefined steps; dynamic adaptive decomposition generates subtasks based on intermediate findings, allowing the plan to adjust with new information
C) Cue chaining is more expensive, dynamic decomposition is cheaper
D) The two are functionally equivalent, but they are implemented in different ways

> **Answer: B**
> Explanation: Hint chaining is suitable for **predictable** tasks (the review always contains the same aspects); dynamic decomposition is suitable for **open-ended** tasks (you don't know what steps are required before exploring). For example, code review (fixed: security, performance, testing, documentation) vs legacy codebase analysis (dynamic: no idea what architectural issues will be found).

---

**Q45.** You need to review a PR that contains 20 files. Which of the following task decomposition strategies best avoids the "attention dilution" problem?

A) Provide all 20 documents at once and request a full review
B) Consolidate the file contents and serve them so that Claude can process them faster
C) Divide the review into two rounds: first local analysis on a file-by-file basis (looking at only local issues in one file at a time), and then integrated analysis across files (focusing on inter-file data flow and compatibility)
D) Randomly select 5 documents for in-depth review

> **Answer: C**
> Explanation: Processing too many files in a single large prompt can lead to diluted attention - some files are analyzed in detail, others have only superficial comments, and even contradictory judgments appear between different files. It is divided into two rounds of file-by-file local analysis + cross-file integration analysis to ensure that each file receives consistent and in-depth attention while not missing integration issues between files.

---

**Q46.** "Add comprehensive testing to legacy code base" is a typical open-ended task. Which decomposition method is most appropriate?

A) Start generating tests for each file immediately, working on a file-by-file basis
B) First map the code base structure and dependencies, identify high-impact areas, create a prioritized plan, and then dynamically adjust as dependencies are discovered
C) Use prompt chain to decompose the task into three fixed stages: unit testing → integration testing → end-to-end testing
D) First ask the customer which files they want the test to cover and then process them in order

> **Answer: B**
> Explanation: The characteristic of open-ended tasks is "not knowing what the most important focus is before starting." Dynamic decomposition (first explore → discover → plan → execute) is more effective than fixed decomposition because it allows prioritizing the really important parts (high impact areas, core business logic, code with complex dependencies).

---

**Q47.** When should you choose **Prompt Chaining** (fixed sequence) over **Dynamic Decomposition** for code review of a code base?

A) When the code base is very large
B) When the review needs to focus on fixed, predictable aspects (e.g. security, performance, test coverage, documentation) - each document needs to be examined from these fixed perspectives
C) When there is only one developer
D) Cue chaining is always better than dynamic decomposition

> **Answer: B**
> Explanation: Prompt chains are suitable for **structured, predictable and multi-faceted tasks**. When you know that reviews always need to check security vulnerabilities → performance issues → test coverage → documentation quality, these steps are fixed and the prompt chain can provide consistency and predictability. Dynamic decomposition is great for open-ended exploration when you don’t know what you’re going to find.---

**Q48.** Your CI/CD pipeline runs code reviews on PRs. Historical data shows that PRs with more than 50 files produce inconsistent review results. What is the most effective solution?

A) Ask developers to split large PRs into smaller PRs
B) Upgrade to the Claude model with a larger context window
C) Divide the review into: (1) independent analysis of local issues for each document; (2) integrated analysis across documents for groups of related documents
D) Skip automatic review for large PRs and use manual review instead

> **Answer: C**
> Explanation: Large context windows do not solve the attention dilution problem - models still tend to focus on some parts and ignore others when processing large amounts of information. The right solution is task decomposition: breaking down large reviews into manageable units, focusing on a limited scope at a time.

---

**Q49.** Which of the following approaches is most effective when designing a decomposition strategy for a large code migration task such as migrating a Java application from Spring Boot 2 to Spring Boot 3?

A) Start the migration directly and adjust when you encounter problems
B) First do a full codebase analysis (Glob/Grep identifies all patterns that need to be changed), then migrate by module/package grouping, and finally do integration testing
C) Provide all files to Claude at once and require full migration
D) Randomly migrate files until the entire code base is covered

> **Answer: B**
> Explanation: Large-scale migration tasks require: (1) **Explore first** (understand the migration scope and complexity); (2) **Group by module** (prevent inconsistencies caused by cross-file dependencies); (3) **Verification points** (verify the functionality of each module after migration). Processing it all at once can lead to dilution and omission of attention.

---

**Q50.** In which of the following situations should a large task** be split into independent local analysis passes + cross-file integration passes** (rather than a single large analysis)?

A) When the task involves only one file
B) When the review needs to identify inconsistencies between files (file A defines a certain function, file B uses it in a different way) AND when a deep local analysis of each file is required
C) When developers want to save API call costs
D) When the time budget of the task is limited

> **Answer: B**
> Explanation: The two-pass method is particularly valuable in the following situations: a combination of depth + breadth is required - each file requires in-depth local analysis (suitable for per-file pass), and at the same time inconsistencies or interface issues between files need to be identified (suitable for cross-file integration pass). A single large analysis would trade off these two goals, resulting in neither being good enough.

---

**Q51.** What is the main purpose of Explore subagent?

A) Replace manual code review
B) Isolate verbose discovery output (detailed code exploration, large read operations) and return the summary to the main session to prevent the main session's context window from filling up with discovery output
C) Process multiple search queries in parallel
D) Generate code documentation

> **Answer: B**
> Explanation: The Explore sub-agent is used to isolate the **lengthy discovery phase**. For example, exploring a large code base requires reading hundreds of files, which produces a lot of output. The Explore subagent handles this detailed exploration and returns only the summary to the main session, keeping the main session's context window clean and available for subsequent implementation steps.

---

## Task 1.7: Session state and forking (Q52–Q60)

---

**Q52.** What is the main difference between `--resume <session-name>` and `fork_session` in Claude Code?

A) `--resume` is used to restore a crashed session, `fork_session` is used for normal work switching
B) `--resume` continues the same conversation thread (continues where you left off); `fork_session` creates an independent branch from a shared baseline, allowing different methods to be explored without affecting the original session
C) `--resume` works in all cases, `fork_session` is experimental
D) Both functions have the same function, but the command syntax is different

> **Answer: B**
> Explanation: `--resume` is linear - continues the same conversation where it left off. `fork_session` is forking - creating two (or more) independent branches from the same starting point, each branch can explore different methods, and the branches do not affect each other.

---

**Q53.** You analyzed a large code base in yesterday's Claude Code session, which had the results of many Grep and Read tool calls. You modified 3 files today and now you want to continue analyzing them. Which of the following methods is most appropriate?

A) Restore yesterday's session without mentioning any new changes - the previous analysis is still valid
B) Start a fresh session and reanalyze the entire codebase from scratch
C) Restore yesterday's session, clearly inform Claude which 3 files have been modified, and request targeted re-analysis of these specific files
D) Use `fork_session` to fork from yesterday's session and process changes in the new branch

> **Answer: C**
> Resolution: When resuming a session, the previous tool results (file read results) may be out of date because the code has changed. But most of the analysis context (architecture understanding, analysis of unmodified files) is still valid. The correct approach is to restore the session + notify specific changes and let Claude perform targeted update analysis instead of wasting resources and re-analyzing the entire process.

---

**Q54.** Under what circumstances is it more appropriate to start a completely new session with structured snippet injection than to resume a previous session?

A) Always - a new session is always more reliable than a restore
B) Never - it is always better to restore the session as it retains more context
C) When a large number of tool results (file reading results, search results) from previous sessions are out of date (the code base has undergone significant changes), the presence of previous context can lead to incorrect inferences based on old information
D) Only if the previous session is more than 24 hours old

> **Answer: C**
> Explanation: Restoring a session can be detrimental when previous tool results (cached file contents, search results) are largely stale - Claude may be making incorrect inferences based on stale information and not be aware of the problem. At this time, it is more reliable to distill the key (still valid) findings into a structured summary and inject it into a new session.

---

**Q55.** You want to explore two different refactoring options at the same time after completing the code base analysis (Option A: Split a large class into multiple small classes; Solution B: Extract shared logic into independent modules). Which Claude Code feature is best suited for this scenario?

A) `--resume` twice, one for each scenario
B) `fork_session`: Fork two independent branches from the baseline of completing the code base analysis to explore plan A and plan B respectively.
C) Explore both options sequentially in the same session
D) Create two new independent sessions and re-analyze the codebase in each session

> **Answer: B**
> Explanation: `fork_session` is designed for this scenario: there is a shared baseline analysis, and multiple different directions need to be explored starting from this baseline, without the hope that these explorations will affect each other. Unlike option D, `fork_session` avoids the cost of repeated code base analysis.

---

**Q56.** After resuming a session, Claude starts giving answers that are inconsistent with the original analysis and starts referring to "typical design patterns" rather than specific implementations in your codebase. What is this most likely problem?

A) API connection issues
B) Context degradation - as the session lengthens, the specific details of earlier analysis lose weight in the context, and Claude begins to rely on general knowledge rather than the specific code base information previously discovered.
C) Model version updates lead to behavioral changes
D) The restored session uses the wrong system prompt

> **Answer: B**
> Explanation: This is a classic symptom of context degradation in an extended session. The model's attention is distributed unevenly across long contexts, and specific findings from earlier sessions may be "diluted" by new information. Solution: Use scratchpad files to record key findings and reference these files explicitly in subsequent questions.

---

**Q57.** In a Claude Code session, when the context window starts to fill with a lot of discovery output, which command can help reduce context usage?

A) `/clear`
B) `/reset`
C) `/compact`
D) `/summarize`

> **Answer: C**
> Explanation: The `/compact` command is used to reduce context usage in an extended exploration session. It compresses existing context, retaining key findings but reducing redundant exploration output, thereby freeing up context space for subsequent code generation or further analysis.

---

**Q58.** When you resume a session using `--resume`, you want Claude to reanalyze modified files. Which method is the most efficient?

A) Tell Claude "Please reanalyze the entire codebase, everything has changed"
B) Don’t tell Claude any changes and let him discover them on his own
C) Explicitly state which specific files were modified (e.g. "auth.py and user_service.py modified") and request targeted reanalysis of those specific files
D) End the session and start from the beginning

> **Answer: C**
> Explanation: Accurate notification of the scope of changes is the most efficient strategy. Asking Claude to reanalyze the entire codebase (A) wastes resources - most files remain unchanged. Failure to notify of changes (B) may cause Claude to make incorrect inferences based on outdated cached information. Targeted notifications + targeted reanalysis is the best balance.

---

**Q59.** In a multi-agent research system, when recovery is required after a system crash, which design best achieves reliable crash recovery?

A) Restart the entire research process from scratch
B) Rely on Claude's built-in memory to automatically restore state
C) Each agent exports state to a known location (such as a JSON manifest file) at key checkpoints, and the coordinator loads these manifests on recovery and injects the state into the agent prompts
D) Use the database to store all intermediate results and query the database during recovery

> **Answer: C**
> Explanation: Structured state persistence is a reliable crash recovery mechanism. Claude has no built-in memory (B is wrong). Each agent exports status to a file when completing key steps. The coordinator maintains a "progress list". After a crash, the coordinator loads the list to understand which steps have been completed and resumes from the interruption point instead of starting over.

---

**Q60.** Which of the following scenarios is a best practice use case for using named sessions (named sessions + `--resume`)?

A) One-time tasks that do not require subsequent continuation
B) Continuous daily investigation of the code base, which requires accumulating an understanding of the system over multiple working days, continuing where we left off each day
C) Single bug fix with clear stack trace
D) Quick code formatting tasks

> **Answer: B**
> Explanation: Named sessions are best suited for ongoing investigative tasks that require **accumulation of understanding across multiple work sessions**. By resuming the same investigation session every day with `--resume <session-name>`, Claude can gradually build a deep understanding of the system and avoid re-exploring what he already knows every day. One-time tasks (A, C, D) do not require session persistence.

---

## Answer Analysis Guide

### Score comparison of each Task Statement| Task | Title | Key Concepts |
|------|------|---------|
| 1.1 | Q1–Q9 | `stop_reason`, dialogue history appending, anti-pattern |
| 1.2 | Q10–Q18 | Hub and spoke architecture, task decomposition, iterative optimization |
| 1.3 | Q19–Q27 | Task tools, context isolation, parallel generation |
| 1.4 | Q28–Q35 | Programmatic vs Prompt, Deterministic Compliance, Handover Summary |
| 1.5 | Q36–Q43 | PostToolUse hook, interception hook, deterministic guarantee |
| 1.6 | Q44–Q51 | Prompt chain vs. dynamic decomposition, attention dilution, Explore sub-agent |
| 1.7 | Q52–Q60 | --resume, fork_session, context degradation, crash recovery |

### Common Pitfalls

- Confusing the triggering timing of `PostToolUse` (after tool result) and tool call interception (before tool call)
- Believe that prompt instructions can provide "deterministic" guarantees (only programmatic mechanisms can)
- Consider that child agents automatically inherit the coordinator's context
- Confusing `--resume` (continue) and `fork_session` (fork)
- Consider iteration caps themselves an anti-pattern (acceptable as secondary protection)