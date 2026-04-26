# CCA-F Simulation Questions - Domain 4: Prompt Engineering and Structured Output

> There are 60 questions total, and the recommended completion time is 75 minutes. single-choice questions.

---

## Task 4.1 Few sample prompts and clear standards (Q1–Q12)

**Q1.** Your business needs Claude to summarize lengthy technical documents into concise executive summaries. You provided an unclear instruction: "write a good summary". What actions should you take to ensure consistent results?

A) Add the temperature parameter called by Claude API to make the model more creative
B) Provide 3-5 examples of excellent abstracts of varying styles and lengths that illustrate your desired format and depth of content
C) Ask Claude to mark each summary paragraph with a symbol
D) Use different model versions and take turns testing which one performs best

> **Answer: B**
> Explanation: Few-shot hints help Claude understand implicit preferences by providing concrete examples. Giving 3-5 examples that demonstrate expectations for length, depth, format, etc. works much better than vague "good summary" directives. This is the most direct way to establish clear expectations.

---

**Q2.** You are teaching Claude how to perform sentiment analysis while using few-shot prompts. Which of the following methods is most efficient at displaying your unwanted output?

A) Add a note at the end of the prompt: "Do not output irrelevant text"
B) Provide negative examples of erroneous output, illustrating how it should not be classified (e.g. incorrectly labeling a neutral opinion as positive)
C) Repeat the correct output examples multiple times until you reach 20 examples
D) Use regular expressions to limit output formats

> **Answer: B**
> Explanation: Explicit negative examples (negative teaching) can help Claude understand what you don't want, which is just as important as providing only positive examples. By showing common misclassification errors, you can effectively reduce the model's hypothesis space and improve classification accuracy.

---

**Q3.** You are developing a content moderation system. Your original instruction was: "Check text for inappropriate content." Which of the following is best to translate this into clear criteria?

A) "Flag text as inappropriate if it contains profanity or hate speech; flag as violent if it contains depictions of violence; flag it as adult if it contains explicit depictions of adult content. Do not flag constructive political discussion."
B) "Check the text for any controversial words"
C) "Use your best judgment to determine what is inappropriate"
D) "Judge this text against community standards"

> **Answer: A**
> Explanation: Clear criteria should enumerate specific classification categories, inclusion/exclusion definitions, and boundary cases. Option A provides clear classification definitions and counterexamples, making review decisions repeatable and consistent, which is the core of clear standards.

---

**Q4.** You have a task: classify customer reviews as "Product Issues", "Logistics Issues" or "Other". Your few-shot prompt contains 2 examples per category. A customer submitted a new review: "My package was left outside the door in the rain and now the box is wet and the items inside may be damaged." Claude gave the wrong classification. To improve accuracy, what should you do?

A) Add more examples, ensuring that examples involving mixed problems involving environmental conditions and product damage are included
B) Ask Claude to re-evaluate with a higher level of confidence
C) Use binary classification instead (product issues vs other)
D) Add a system prompt: Never classify as "Other"

> **Answer: A**
> Explanation: This comment relates to a cross-border situation (logistics and product damage). Adding few-shot examples that cover edge cases is the most effective way to improve accuracy. Option B misunderstands the role of confidence, and options C and D are inappropriate system-level changes.

---

**Q5.** When using the Chain-of-Thought prompt, what should you include in the prompt to maximize its effectiveness?

A) More examples, each with 100+ steps of reasoning
B) Ask Claude to "think about it" and then request the answer at the end while showing the complete reasoning steps in the example
C) Provide only answers and let Claude deduce the reasoning process on his own
D) Use a forced, step-by-step numerical list format with a limit of 10 words per step

> **Answer: B**
> Explanation: Chain thinking improves accuracy by allowing Claude to show his reasoning process before giving his final answer. What works is to demonstrate the complete inference steps on a few-shot example and then ask Claude to do the same in a real task. This can significantly reduce error rates in complex tasks.

---

**Q6.** You are writing an XML structured prompt for a data annotation task. What are the best practices?

A) Put everything in the <instructions> tag without any splitting
B) Use a clear hierarchical structure: <instructions>, <examples>, <output_format>, <edge_cases> to keep different content types separated and easy to parse
C) Only use JSON as XML is obsolete
D) Mix XML tags into natural language to make prompts more organic

> **Answer: B**
> Explanation: The key advantage of XML tags is clear layering and structural separation. Using named containers (such as <instructions>, <examples>, <output_format>) helps Claude understand the purpose of different parts, thereby improving compliance and output quality.

---

**Q7.** You are writing a prompt for a legal document review assignment. What should the system prompt contain and what should the user message contain?

A) System prompts should include specific review criteria; user messages should include files to be reviewed
B) System prompts should contain general legal knowledge; user messages should contain review standards and documents
C) All content should be in the system prompt, user messages should only contain the file content
D) All content should be in the user message and the system prompt should be empty

> **Answer: A**
> Explanation: System prompts should define roles, rules, and criteria (relatively static, consistent content across requests); user messages should contain task-specific data (documents that are reviewed). This separation ensures reusability while maintaining flexibility in handling different inputs.

---

**Q8.** In the few-shot prompt, you realize that the examples are formatted inconsistently (some in JSON, some in Markdown). What impact will this have?

A) No impact on Claude's performance, as long as the content is correct
B) May reduce Claude's ability to understand the example, resulting in inconsistent output formats because Claude replicates the inconsistency
C) Will force Claude to learn multiple formats and increase his flexibility
D) Will cause API errors

> **Answer: B**
> Explanation: Claude understands the expected output by learning patterns from examples. Inconsistent formatting confuses the model's expectations about the form of the target output, leading to confusing formatting in the actual output. All few-shot examples should use a consistent structure and format.

---

**Q9.** Your task is to extract structured data (company name, year of establishment, industry) from news articles. Your initial prompt lacks few samples. 30% of the fields in the results are empty or malformed. What type of few samples would be most likely to be added to solve this problem?

A) Add 10 successful data extraction examples showing how to handle missing fields, date variations, and industry classifications
B) Add a generic news article and ask Claude to extract only the available fields
C) Add a hint saying "If field doesn't exist, skip it"
D) Use regular expressions for extraction instead

> **Answer: A**
> Explanation: 30% of fields are missing or malformed indicating that Claude doesn't know how to handle edge cases. Adding few-shot examples covering missing field handling, multiple date formats, and classification variations will directly address these issues. Options B and C worsen the problem.

---

**Q10.** When writing few examples for customer feedback classification prompts, how many examples should you include?

A) At least 50 to cover all possible variations
B) 2-5 carefully selected examples per target category showing typical and edge cases
C) Only 1 example is enough because Claude is smart enough
D) 100+ examples to ensure absolute accuracy

> **Answer: B**
> Explanation: Research shows that 2-5 high-quality examples per category are often sufficient for effective few-shot learning. The key is quality, not quantity—choose examples that represent typical and edge cases for the category. Too many examples can increase noise and confusion.

---

**Q11.** You are writing prompts for paper abstract generation. You included 5 good summary examples, but the result is still too lengthy. What are you most likely missing?

A) Explicitly state length constraints in examples (e.g. ~150 words per abstract)
B) Ask Claude to use a higher temperature value
C) More examples
D) A more powerful system prompt

> **Answer: A**
> Explanation: Although the example is useful, Claude may not have inferred the exact length constraints from the example. Clear criteria (e.g. "about 150 words", "no more than 3 paragraphs") should be clearly stated both in the instructions and in the examples. Few-shot examples should obey this constraint consistently.

---

**Q12.** You are designing few-shot prompts for a medical term recognition task. The prompt contains 4 examples, each from the same specialty area (cardiology). New input comes from the field of oncology and accuracy decreases. What is the most likely cause and what should be done?

A) Claude does not support medical tasks; use a different API instead
B) Examples are too narrow in scope; add examples from multiple medical specialties, including oncology
C) More cardiology examples are needed
D) The task of medical terminology is inherently unstable

> **Answer: B**
> Explanation: Few-shot examples should be representative of the data distribution that the task will process. If all examples come from one domain of expertise, the model will overfit to patterns in that domain. Adding examples from multiple medical fields, including oncology, will improve generalization across fields.

---

## Task 4.2 Reduce the false positive rate (Q13–Q24)

**Q13.** In a classification task, what is the difference between "false positives" (false positives) and "false negatives" (false negatives)?

A) False positive example = the model says it is, but it is actually not; False negative example = the model says it is not, but it is actually not
B) False positive examples = fake data; False negative examples = real data
C) The two are synonyms and can be used interchangeably
D) False positive examples are used for precision, and false negative examples are used for recall.

> **Answer: A**
> Explanation: Correct definition is crucial to selecting optimization indicators. False positives (where the model incorrectly predicts a positive) lead to false positives; false negatives (where the model incorrectly predicts a negative) lead to false negatives. This trade-off needs to be understood when choosing model thresholds and design cues.

---

**Q14.** You are building prompts for spam detection. The business requirement is to minimize false positives (legitimate emails being marked as spam). What should you adjust to achieve this?

A) Increase model temperature for more creative classification
B) Clearly require high accuracy in the prompt and lower the threshold for classifying spam (stronger evidence is needed to mark it as spam)
C) Add more spam examples
D) Use lower top-p parameter

> **Answer: B**
> Explanation: Minimizing false positives means optimizing accuracy (true examples/all predicted positives). This requires the model to require stronger evidence to make a positive classification by raising the threshold. On the contrary, if you want to minimize false negatives, you need to lower the threshold and increase the recall rate.

---

**Q15.** You are developing a classifier for content moderation. The model is over-triggered (flagging a lot of legitimate content as inappropriate). What is the most likely cause and how to fix it?

A) The model is not smart enough; change to a more powerful model
B) Lack of negative examples (examples of legal content); add multiple examples of legal content to show content that should not be flagged as inappropriate
C) requires higher temperatures
D) Use stricter regex filters

> **Answer: B**
> Explanation: Over-triggering (high false positive rate) is usually because the model does not have enough negative teaching. Adding negative examples (examples of content clearly marked as inappropriate) helps Claude understand classification boundaries and reduce false positives.

---

**Q16.** When weighing the trade-off between precision and recall, in which of the following situations should high precision be given priority?

A) Patient risk screening: you want to capture all potential risk cases
B) Fraud detection: you want to minimize false fraud alerts because each alert is expensive
C) Medical Diagnostic Aids: You want to make sure every diagnosis is accurate
D) Recommendation system: you want to show the user as many relevant items as possible

> **Answer: B**
> Explanation: In fraud detection, false positives (legitimate transactions marked as fraud) are very costly (user frustration, manual review costs), so accuracy should be prioritized. In medical screening, the cost of missing a real risk (false negative case) is higher, so recall rate should be prioritized. Understanding business costs is critical.

---

**Q17.** You are building a customer support ticket prioritization classifier. The problem now is that most tickets are marked as low priority and truly urgent issues are missed. What problem does this indicate?

A) Need more low priority examples
B) High false negative rate (urgent tickets are not recognized); adding more positive examples of urgent issues may lower the classification threshold
C) The model is not suitable for the task
D) Need to delete low priority samples

> **Answer: B**
> Explanation: Missing the real urgent problem is a false negative example (low recall rate). To improve detection of emergent issues, add positive examples of emergent issues and emphasize the importance of identifying emergent issues in your prompts. This increases recall (although may decrease precision).

---

**Q18.** In multi-classification tasks (eg: classifying reviews into 5 sentiments), how to reduce misclassification of minority categories?

A) Remove minority categories entirely
B) Provide more few-shot examples for minority categories to ensure adequate category representation and emphasize their definitions in the hints
C) Ignore minority categories and focus on majority categories
D) Use random classification to balance the results

> **Answer: B**
> Explanation: In multi-classification tasks, minority categories are often underestimated. By providing more examples and clear definitions for these categories, the recognition ability of the model can be improved. Unbalanced example representation leads to a high false negative rate for the minority class.

---

**Q19.** You are testing a classification prompt. You get 100% accuracy on 10 examples, but only 75% on 100 more diverse test samples. What does this indicate?

A) This is normal, don’t worry
B) Your model is overfitting to few examples; need to test more edge cases and add examples that cover them
C) There must be something wrong with the test data
D) Need to increase the number of examples to 100

> **Answer: B**
> Explanation: The gap between high training accuracy and low test accuracy indicates overfitting. The initial 10 examples may not represent the diversity of the real data. Adding and testing edge cases, anomalies, and various data variations will improve generalization performance.

---

**Q20.** You included a symmetrical number of categories in the classification prompt (example: 5 positive examples per category). Why might this not be good enough for some tasks?

A) Symmetrical examples are always best practice
B) Some categories may be inherently more difficult to classify or confusing; provide additional examples and more detailed definitions for difficult categories
C) Symmetry has no effect
D) 100+ examples should be provided for all categories

> **Answer: B**
> Explanation: Not all categories are equally easy to learn. Some categories may be more similar (easily confusing) to other categories, or may be inherently more subtle. By analyzing classification errors, difficult categories are identified and then provided with additional examples and more refined definitions for these categories.

---

**Q21.** To test the robustness of classification cues to adversarial or unusual inputs, what should you do?

A) Only test typical examples because that's what the model will encounter
B) Create adversarial test cases (typos, sarcasms, negatives, etc.) explicitly designed to fool the model and evaluate error rates
C) Avoid testing any "fake" examples
D) Assume that if the model performs well on standard examples, it will handle all cases

> **Answer: B**
> Explanation: Adversarial testing is key to identifying weaknesses in classifiers. Testing for typos, context inversions, ambiguous edge cases, etc. will reveal failure patterns that may occur in real data. This allows you to improve the prompt by adding examples that cover these situations.

---

**Q22.** You are developing prompts for product category classification. A specific product (for example: an item that can be used as a chair or a step stool) is misclassified 50% of the time. Why is this so difficult to solve and what is the correct approach?

A) This task is impossible; give up
B) This is an inherent label ambiguity issue; explain in the tip how to handle edge cases (e.g.: "Prioritize primary use") and include examples of such products
C) Increase the temperature of the model
D) Ask Claude to give a single, clear answer

> **Answer: B**
> Explanation: Real data often contains label ambiguity (items belonging to multiple categories). The solution is to make it clear in the prompt how to handle these situations (priority rules, definitions) and use examples to illustrate your preferences. This is more important than improving accuracy.

---

**Q23.** Why shouldn't you rely solely on overall accuracy when evaluating classification performance?

A) Accuracy is the only metric required
B) Accuracy can be misleading when classes are imbalanced; consider precision and recall for each class, or use F1 score
C) Accuracy is too easy to understand
D) Classification tasks should not be evaluated

> **Answer: B**
> Explanation: If 95% of the data is class A and 5% is class B, a model that always predicts A will get 95% accuracy, but fail completely at identifying class B. Precision, recall, and F1 scores for each category provide a more accurate picture of performance.

---

**Q24.** You noticed that classification hints perform well for some categories and poorly for others. Which of the following methods is most diagnostic?

A) Assuming poor performance is random, rerun the test
B) Analyze the confusion matrix (which class is misclassified as which), identify confused class pairs, and then add discriminating examples for these class pairs
C) Increase the number of all examples
D) Use different models for poorly performing categories

> **Answer: B**
> Explanation: The confusion matrix identifies which pairs of categories are most commonly confused. This guides you on where to add improving examples. For example, if "like" is often mistaken for "neutral," add examples that show how these emotions differ. This targeted approach is more effective than blindly adding examples.

---

## Task 4.3 Structured output design (Q25–Q36)

**Q25.** When should tool_use be used to force structured output, rather than requiring JSON in the prompt?

A) Always requires JSON, tool_use has no advantage
B) When you need to ensure that Claude outputs 100% valid structured data, use tool_use; JSON format requests may cause parsing errors or inconsistent formats
C) tool_use should never be used on structured output
D) Both methods are exactly the same, the choice is not important

> **Answer: B**
> Explanation: tool_use forces Claude to return data according to the precise structure defined by JSON Schema. In contrast, requiring JSON in a prompt may result in format variations, missing fields, or invalid JSON. When accurate structure is critical, tool_use is the more reliable choice.

---

**Q26.** You are defining a JSON Schema for customer information extraction. A customer may have one or more phone numbers. How should this field be represented in Schema?

A) "phoneNumber": {"type": "string"} - only one phone number is allowed
B) "phoneNumbers": {"type": "array", "items": {"type": "string"}} - allows an array of multiple phone numbers
C) "phoneNumber": {"type": "object"} - Use objects to store multiple values
D) Use two separate fields: phoneNumber1, phoneNumber2, etc.

> **Answer: B**
> Explanation: Arrays are the standard way to represent one-to-many relationships in JSON. The "phoneNumbers" array allows zero, one or more phone numbers. This is more flexible and scalable than using multiple named fields (phoneNumber1, phoneNumber2).

---

**Q27.** What is the purpose of the "required" field in your JSON Schema?

A) List the fields you expect, but don't force Claude to include them
B) Define which fields must be included in the response by Claude; omitting fields that Claude can skip if they are not applicable
C) Limit the size of JSON
D) Duplicate with "type" field, unnecessary

> **Answer: B**
> Explanation: The "required" array in Schema specifies which fields are mandatory. For example, "required": ["id", "name"] means that Claude must always return these fields. Optional fields should be excluded from the required array, allowing Claude to omit them if they are not relevant.

---

**Q28.** You are designing a schema for product information extraction. The product has a name (required), optional description, and possibly empty SKU. What is the best Schema definition for the SKU field?

A) "sku": {"type": "string"}
B) "sku": {"type": "string", "nullable": true}
C) "sku": {"type": ["string", "null"]}
D) "sku": {"type": "string"} and exclude it from the required array

> **Answer: B**
> Explanation: Use "nullable": true to indicate that the field can be explicitly null. This is different from the field not appearing. Option D (exclude required) is for a field that might be missing entirely; Option B is for a field that might be present but have a null value. The semantics of the two are different.

---

**Q29.** Your tool_use Schema contains a nested object: Address has street, city, state, and zip code. How to define this in Schema?

A) "address": {"type": "string"} - treat the entire address as a single string
B) "address": {"type": "object", "properties": {"street": {"type": "string"}, "city": {"type": "string"}, ...}, "required": ["city", "state"]}
C) Use 4 separate fields: addressStreet, addressCity, etc.
D) Nested objects are not supported in Schema

> **Answer: B**
> Explanation: JSON Schema supports nested objects. Use "type": "object" and "properties" to define nested structures. In nested objects, the "required" array can also specify which nested fields are required. This is cleaner and more maintainable than flat field names.

---

**Q30.** You want to use tool_use, but need to ensure that Claude always calls a specific tool (for example: extract_data). How should you configure it?

A) "tool_choice": "auto" - Let Claude choose freely
B) "tool_choice": {"type": "tool", "name": "extract_data"} - forces Claude to always call this tool
C) The system prompt says "extract_data must be used"
D) tool_choice parameter does not exist

> **Answer: B**
> Explanation: The tool_choice parameter controls how Claude selects tools. "auto" allows free choice (possibly not calling the tool at all); {"type": "tool", "name": "extract_data"} forces calling of the specified tool. This is critical when you need to ensure structured output.

---

**Q31.** Your Schema defines an enumeration field: "status": {"type": "string", "enum": ["active", "inactive", "pending"]}. If the status is not explicitly stated in the text, what should Claude do?

A) Guess a value
B) Return a value not in enum
C) Clearly state in the prompt how to handle uncertain situations (e.g. "If status is unclear, default to 'pending'")
D) Omit this field if unsure

> **Answer: C**
> Explanation: Enum limits the possible values, but does not solve the problem of how to handle ambiguity. Enumerations defined in the Schema should have clear guidance in the hint: such as default values, confidence thresholds, or how to handle missing information. This ensures consistent, predictable behavior.

---

**Q32.** You are using tool_use for classification tasks. The data validator detected that some outputs had semantic errors (eg: classified as "Urgent" but the reason given was completely irrelevant). What's the problem?

A) JSON Schema validation failed
B) Semantic error (violates business logic, but JSON is syntactically valid); validation rules and few samples need to be added to the prompt to demonstrate consistent classification reasons
C) Claude cannot be used for this task
D) Post-processing is required to clean the data

> **Answer: B**
> Explanation: JSON Schema verifies syntax (type, structure); it cannot check semantic consistency (whether the classification is consistent with the reason). By adding clear rules ("classification must be consistent with justification") and few-shot examples in the hints, you are able to reduce semantic errors at generation time.

---

**Q33.** You are designing a tool schema for an API that handles variable-length lists. A product can have 0 to 100+ tags. What should be defined in the array's Schema to handle this?

A) "tags": {"type": "array"} - no size limit
B) "tags": {"type": "array", "items": {"type": "string"}, "minItems": 0, "maxItems": 100}
C) Instead of using arrays, use separate fields
D) If there are more than 10 projects, split them into multiple tool calls

> **Answer: B**
> Explanation: JSON Schema supports "minItems" and "maxItems" constraints. These ensure that the array size is within the defined range. Setting reasonable limits (minItems: 0 allows empty lists, maxItems: 100 prevents overbloat) can help control output size and processing time.

---

**Q34.** Under what circumstances might it be reasonable to ask Claude to return JSON directly in text (without using tool_use)?

A) You should never do this; always use tool_use
B) For simple, one-time tasks where the output does not need to be parsed into structured data, or in API clients that do not support tool_use
C) tool_use is always better
D) This method has no disadvantages

> **Answer: B**
> Explanation: While tool_use is more reliable, for simple use cases it may be simpler to return JSON in text. The trade-offs are: tool_use provides guarantees and enforced structure but adds complexity; textual JSON is simpler but requires user-side parsing and error handling.

---

**Q35.** You are designing a schema where some fields only make sense if other fields have specific values (for example: trackingNumber is required if orderStatus == "shipped"). How does JSON Schema handle this conditional logic?

A) Use a simple required array (this does not support conditions)
B) Use JSON Schema's "if/then" constraints or explicitly state conditional rules in the prompt
C) Conditional logic is not possible in Schema
D) Create two separate Schemas: one for each case

> **Answer: B**
> Explanation: Standard JSON Schema has limited conditional support (if/then). For complex conditions, the most reliable approach is to state the rule explicitly in the prompt: e.g. "if orderStatus is 'shipped', trackingNumber must exist". Combined with few examples, this ensures that Claude understands the conditions.

---

**Q36.** You have an API that accepts up to 10 tool_use tool definitions. You need to extract 15 different data types. What is the best design approach?

A) Create 2 separate API calls, the first with 10 tools and the second with 5 tools
B) Create a comprehensive tool whose Schema contains 15 different object types as branches, and the user specifies which type to extract
C) Abandon tool_use and use textual JSON instead
D) Only supports 10 data types

> **Answer: B**
> Explanation: Rather than multiple sequential calls, a better design is to create a single, flexible tool whose Schema allows parameters to be used to select which type of data to extract. This is achieved by using a "dataType" enumeration parameter and a "data" object, whose structure changes according to the dataType type.

---

## Task 4.4 Multi-round review structure (Q37–Q48)

**Q37.** In a multi-round review architecture, what system prompts should be used for the first and second rounds?

A) Use the same system prompt in both rounds
B) Round 1: Generate system prompts (promote creativity and content generation); Round 2: Review system prompts (critical thinking, problem identification)
C) There is no system prompt in the first round, but there is a review prompt in the second round.
D) The system prompt has nothing to do with this architecture

> **Answer: B**
> Explanation: The key to multiple rounds of reviews is for both roles to optimize their goals. Generators should be optimized for producing quality content; reviewers should be optimized for criticism and suggestions for improvement. Different system prompts will allow for better performance each round, rather than having the same prompt try to do two things.

---

**Q38.** You are implementing multiple rounds of review for an article generation task. The output of the first round is: "The potential of artificial intelligence is unlimited." What issues should be identified in the second round?

A) The article is too short
B) The statement is vague and lacks support; reviewers should flag it as requiring specific evidence, examples, and more refined arguments
C) Artificial Intelligence is not a valid topic
D) No review required, the article is already good

> **Answer: B**
> Explanation: An effective review does more than just count metrics; it identifies specific flaws in the content. The statement "potential is unlimited" is vague, unproven, and uses absolute language. Reviewers should note that this requires supporting evidence, qualifications, and specifiable arguments.

---

**Q39.** In a multi-round architecture, what should a reviewer do when they find many issues?

A) Create a detailed feedback list asking the generator to fix all issues at once
B) Rank issues by priority or impact, ask the generator the most critical 2-3 questions for a second iteration, and continue as needed
C) Abandon the project, the quality is too poor
D) Let reviewers fix any problems themselves

> **Answer: B**
> Explanation: Giving too much feedback at once can overwhelm the model, leading to inconsistent improvements. A better approach is to improve iteratively: prioritize feedback, focusing on the most impactful issues each time, and then conduct a new review cycle. This results in incremental improvements and better final quality.

---

**Q40.** You implemented a multi-round architecture for code reviews. The generator generated the code and the reviewer identified a security issue. What should be the next step?

A) Flag the issue and forward it to a human engineer
B) Tell the generator explicitly what to fix (eg: "Add input validation to prevent SQL injection"), let the generator fix it, and then review it again
C) Assume the question is not true and proceed
D) Let Claude edit the code himself to fix the problem

> **Answer: B**
> Explanation: The purpose of multi-round architecture is to improve through feedback loops. When reviewers identify issues, provide clear, actionable feedback to the generator, allowing the generator to regenerate improved code. Then, have the reviewer verify that the fix actually solved the problem and didn't introduce new problems.

---

**Q41.** In multiple rounds of reviews, how to prevent cycles (reviewers keep finding the same issues and the generator cannot improve)?

A) Only one round of review
B) Define convergence criteria: e.g. "If the problem list does not reduce by more than 20% between two consecutive rounds, stop the iteration"; if the stopping condition is reached, escalate to a human or use a different strategy
C) Continue iteration infinitely until perfection
D) Increase the temperature of the generator each round to make it more creative

> **Answer: B**
> Explanation: Multiple rounds of iteration can get stuck in a loop, especially if the problem is fundamentally related to the capabilities of the model. It is important to have well-defined stopping conditions: for example, if improvements plateau or the number of issues does not decrease, stop the iteration and consider escalating to humans or adjusting the strategy.

---

**Q42.** You are implementing multiple rounds of reviews for translation quality. The first round: translating the first draft; the second round: identifying problems. What should be done in the third round?

A) Stop at the second round, two rounds are enough
B) Round 3: Let the generator (or a different model) improve the translation based on review feedback, then optionally review it again in Round 4
C) Repeat the second round
D) Let a human translator do the third round

> **Answer: B**
> Explanation: Multi-round architecture can be extended to 3+ rounds: Build → Review → Improve → Re-review (if needed). The key is to have a clear workflow: alternating production and review roles. This way you can achieve incremental improvements without involving humans at every step.

---

**Q43.** What kind of reminder engineering is most critical for effective multi-round review?

A) Generators and reviewers should be as similar as possible
B) The inspector should have a clear, detailed checklist that tells it what to look for; the generator should have a clear goal statement that tells it what to generate.
C) Prompt project has nothing to do with multiple rounds of review
D) The censor should be given as little guidance as possible, leaving it free to interpret

> **Answer: B**
> Explanation: The success of multiple rounds of reviews depends on each role having clear, actionable guidance. The reviewer needs to know what to check (the checklist); the generator needs to know what the goal is. Vague guidance can lead to inconsistency and low-quality feedback loops.

---

**Q44.** You are using a multi-round architecture for data quality improvement. The generator outputs a data set and the reviewer examines it. After reviewer feedback, you notice that the second version of the generator contains completely different data. What is the most likely cause and what should be done?

A) Claude is not suitable for the task.
B) Generator misunderstood review feedback or was not clear enough; rephrase feedback to be more specific, e.g.: "Keep existing 500 records, add missing fields" instead of "Improve data"
C) This is expected, data always changes
D) Data generation should not use a multi-round architecture

> **Answer: B**
> Explanation: When feedback is too vague or too high-level, the model may overinterpret and make irrelevant changes. The way to improve is to provide specific, clearly scoped feedback: specifying what should be retained, what should be improved, and how. This prevents the model from making unwanted large-scale changes.

---

**Q45.** In multiple rounds of review, should multiple different reviewers independently review the same output?

A) No, it would be a waste of time and cost
B) Can be considered for high risk or mission critical; multiple reviewers may find different issues and their consensus increases confidence, but costs double the API calls
C) Always use multiple reviewers
D) A single reviewer is always sufficient

> **Answer: B**
> Explanation: Multiple independent reviewers can increase problem detection (one reviewer may miss something discovered by another reviewer). For critical applications (medical, legal, financial) this may be worthwhile, albeit at a higher cost. For low-risk tasks, a single reviewer may be sufficient. It's a cost versus quality trade-off.

---

**Q46.** You are implementing multiple rounds of review for creative writing. Reviewer Feedback: "The story is good, no issues". Why is this kind of feedback problematic in multi-round architectures?

A) This is perfect feedback
B) It is not specific or constructive enough; even if there are no errors, the reviewer should suggest improvements, deepening of characters, improved pacing, etc. to help the story develop further
C) Creative writing should not be censored
D) Feedback is too long

> **Answer: B**
> Explanation: In multiple rounds of reviews, the reviewer's goal is not only to find errors, but also to guide improvements. For creative writing, "no problem" doesn't give the generator any direction for iterative improvement. More useful feedback will suggest specific improvements: strengthening dialogue, deepening character development, improving ebbs and flows, etc.

---

**Q47.** In multiple rounds of reviews, should the generator be able to see review feedback from previous rounds?

A) No, this would put the generator in a loop
B) Yes, the generator should see the full history of feedback so it can understand the evolution of the problem and the success or failure of its improvements
C) Only the latest feedback
D) Review feedback should be kept confidential

> **Answer: B**
> Explanation: A complete history provides context to help the generator understand the evolution of the problem. For example, if the reviewer says "this is still lacking detail" (after two rounds), the generator knows it needs to be more aggressive in increasing detail. Complete context leads to better improvements.

---

**Q48.** How should you handle all conversations and feedback when implementing a multi-round review architecture?

A) Nothing is stored, each round is independent
B) Maintain a complete conversation history, including all generation, review and feedback rounds; this allows for post-processing, quality analysis and continuous improvement
C) Only store the final result
D) Dialogue is irrelevant to the architecture

> **Answer: B**
> Explanation: Maintaining a complete history is important for several reasons: 1) for debugging and improving processes; 2) for monitoring quality trends (has it improved?); 3) for auditing and compliance; 4) for learning how to improve tips. These insights are lost by not storing the data.

---

## Task 4.5 batch processing and Message Batches API (Q49–Q60)

**Q49.** What are the main advantages of the Message Batches API over regular API calls?

A) Ability to use more advanced features such as tool_use
B) 50% cost savings (at the expense of latency); supports processing windows of up to 24 hours
C) Faster response time
D) Support real-time interaction

> **Answer: B**
> Explanation: The main selling points of the Batches API are cost and scale. By processing requests in a non-real-time manner, Anthropic can optimize infrastructure usage and reduce costs by 50%. The downside is that there is no latency SLA and it can take up to 24 hours.

---

**Q50.** Which of the following use cases is best suited for using the Message Batches API?

A) A real-time chat application that needs to respond to the user immediately
B) Process 10,000 customer support tickets per night to triage and generate response suggestions
C) An interactive conversational agent
D) An API that needs to return results within 5 seconds

> **Answer: B**
> Explanation: Batches API is very suitable for large-scale, non-real-time, offline processing tasks. Processing 10,000 tickets per night is the perfect use case: the data is ready before work starts, processing can take place during off-hours, and the results are ready the next morning. Options A, C, and D all require real-time response.

---

**Q51.** In the Message Batches API, what is the purpose of the "custom_id" field?

A) This is the internal ID assigned by the API for each request
B) It is a user-supplied, unique identifier used to associate the results in a batch request with the original input
C) This field is optional and does not affect processing
D) It is used for authentication

> **Answer: B**
> Explanation: custom_id is an identifier you provide that is meaningful to your system. When the batch returns, the results include the custom_id, allowing you to match the results to the original data. For example, custom_id can be a customer ID, line number, or document identifier.

---

**Q52.** What are the important limitations in the Message Batches API?

A) Support tool_use and multi-turn dialogue
B) tool_use is not supported; each request must be a single round (no multi-round conversations); requests can only use the messages API, no streaming
C) No restrictions
D) Only supports text input

> **Answer: B**
> Explanation: The key limitation of the Batches API is that it is designed for simple, scalable batch processing. No support for tool usage or multiple rounds of conversations. Each request is independent and single-round. This simplifies processing, but means that some advanced use cases are not suitable for the Batches API.

---

**Q53.** You are submitting a batch of 50,000 text classification requests. What is the most efficient way to do this?

A) Submit all 50,000 requests at once
B) Group requests into batches of up to 10,000 requests and submit them sequentially; this avoids super batch processing while still achieving cost effectiveness
C) Submit 1 request at a time, wait for completion, then submit the next one
D) Use regular API instead of Batches

> **Answer: B**
> Explanation: While the Batches API can handle large numbers of requests, splitting into reasonably sized batches (e.g. 10,000) provides better manageability and fault tolerance. If a batch of 10,000 requests fails, you only have to resubmit 10,000, not 50,000. This also prevents problems with very large files.

---

**Q54.** How to monitor the progress and completion of batch processing when using the Batches API?

A) Poll the API and periodically check the batch status until it shows "completed" or "failed"
B) Using a webhook, the Batches API will send a notification on completion
C) Both are supported: polling or webhooks; the choice depends on your architecture
D) Unable to monitor progress

> **Answer: C**
> Explanation: The Batches API supports two ways to track completion: polling (periodic query status) and webhook (passive notification). The choice depends on your architecture. For simple scripts, polling may be sufficient; for production systems, webhooks are more efficient, but require a web endpoint to receive notifications.

---

**Q55.** You are using the Batches API for data processing. Two requests in processing failed. What should I do?

A) The entire batch is considered failed; all requests are resubmitted
B) Failed requests are marked as failed in the results; you can identify them (via custom_id) and resubmit only the failed requests
C) Failed requests are automatically retried
D) Failure cannot be recovered

> **Answer: B**
> Explanation: The Batches API returns partial results, marking which ones succeeded and which ones failed. With custom_id you can identify exactly which requests failed and only resubmit those. This is more efficient than resubmitting the entire batch.

---

**Q56.** You are preparing 10,000 classification requests for the Batches API. Each request is similar, only the input text differs. How to structure a request most efficiently?

A) Use a different system prompt for each request
B) Use the same system prompt and model parameters for all requests, put the input text in the user message for each request, use a unique custom_id
C) Each request should be completely unique
D) It is impossible to efficiently batch similar requests

> **Answer: B**
> Explanation: For efficiency, repeated tasks should use the same system prompts and parameters. Only the user message (input text) is changed. This allows Anthropic's infrastructure to optimize processing and maximize cost savings. A custom_id for each request allows you to track the results.

---

**Q57.** When using the Batches API, when should you decide not to use it and use the regular API instead?

A) Always use the Batches API, it's always better
B) If you need real-time or near-real-time results (< minutes), or need to use tool_use or multiple rounds of dialogue, use the regular API; the Batches API is suitable for offline, large-scale processing
C) Never use the Batches API
D) cannot be compared

> **Answer: B**
> Explanation: The choice between Batches API and regular API depends on the requirements. Batches: Cost efficient, can wait 24 hours, process at scale. General API: real-time response, tool_use, multi-round dialogue, interactive application. Choosing the right tool depends on your use case constraints.

---

**Q58.** You are using the Batches API for daily report generation. You have 1,000 unique reports to generate. Each report requires 2-3 Claude calls (data collection, first draft, review). Is the Batches API a good choice for doing this?

A) Yes, the Batches API works perfectly
B) No, because each report requires multiple rounds and the Batches API only supports single-round requests; either submit separate batches for each stage or use the regular API for multi-round requirements
C) Unable to generate reports using API
D) Someone should do this manually

> **Answer: B**
> Explanation: This highlights a key Batches API limitation: it does not support multi-turn conversations. You can work around this: submit one batch for data collection, then another for the first draft, and so on. But it's more complicated. For tasks that require a close feedback loop, a regular API or a multi-round review architecture may be more appropriate.

---

**Q59.** You are designing a large-scale classification system. The expected load is 5,000 classification requests per minute. Is the Batches API suitable for this?

A) Yes, the Batches API is designed for any scale
B) No, because the Batches API is designed for large-scale daily or weekly batch processing, rather than real-time, continuous traffic; for 5,000 requests per minute, using the regular API or a custom queuing system may be more suitable
C) A database should be used instead of an API
D) 5,000 requests/minute is too many for any system to handle

> **Answer: B**
> Explanation: Batches API is most suitable for offline, periodic large-scale processing. Sustained, consistently intensive traffic (like 5,000/minute for a live system) should use the regular API, possibly with queuing and rate limiting. The Batches API's 24-hour processing window is too long for a real-time system.

---

**Q60.** To summarize, which of the following statements about the Message Batches API best describes its position and trade-offs?

A) Batches API is the best choice in all cases
B) The Batches API provides 50% cost savings for large-scale, offline, non-time-sensitive tasks at the expense of no real-time guarantees and no support for tool_use or multiple rounds; the regular API is suitable for real-time, interactive applications; the choice depends on your latency and feature requirements
C) The Batches API is identical in functionality and cost to the regular API
D) There are only a few rare use cases suitable for any API

> **Answer: B**
> Explanation: This is the most accurate summary of the trade-offs between the Batches API and the regular API. The Batches API is not a replacement, but a complementary tool. Understanding the pros, cons, and constraints of both tools, and being able to choose the right tool for each use case, is the core of architect-level knowledge.

---