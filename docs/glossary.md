# AI Agent Glossary

> **Scope:** Terms needed before working on any aspect of the Gorgias AI agent stack — covering vocabulary used by Product, MLA, MLE, and Software Engineering.
>
> **Ordering principle:** Within each section, terms are ordered alphabetically for easy lookup. Sections themselves are grouped by topic.

---

## 1. Foundations — Language Models

**Context window**
The maximum number of tokens an LLM can read and reason over in a single call — both input and output combined. Content outside the context window is invisible to the model. Context window size is a hard constraint that shapes memory, retrieval, and conversation design.

**Embedding**
A fixed-length numeric vector that represents a token (or a chunk of text) in a high-dimensional space. Embeddings encode semantic meaning: tokens with similar meanings have vectors that are close together. They are the bridge between discrete text and the continuous math inside a neural network. Embeddings are also used externally, as the basis for semantic search in a vector database.

**Hallucination**
When a model generates text that is confident but factually incorrect or unsupported by its context. Hallucination is a fundamental property of probabilistic text generation, not a bug to be eliminated entirely, but a risk to be managed through grounding, retrieval, and evaluation.

**Inference**
Running a trained model on an input to produce an output. Inference is what happens every time you call an LLM API. It is distinct from training (adjusting model weights). In production systems, inference cost, latency, and throughput are primary operational concerns.

**Large Language Model (LLM)**
A neural network trained on massive text corpora to predict the next token given all prior tokens. "Large" refers to the parameter count (billions to trillions). LLMs are the core reasoning and generation engine behind AI agents.

**Token**
The smallest unit of text that an LLM processes. A token is roughly 3–4 characters of English text, but the exact mapping depends on the model's tokenizer. Token count — not character count — governs context limits, latency, and cost.

---

## 2. Prompt Fundamentals

**Chain-of-Thought (CoT)**
A prompting technique that instructs the model to produce explicit intermediate reasoning steps before its final answer. Prevents the model from jumping directly to a pattern-matched answer. Most effective on complex, multi-step reasoning tasks and on sufficiently large models.

**Few-shot examples**
Concrete input-output pairs included in the prompt to demonstrate the desired behavior, format, or reasoning style. The model generalizes from these examples without any parameter update. Also called multishot prompting. Few-shot examples are one of the most reliable ways to steer model output.

**Prompt**
The complete input sent to an LLM. A prompt contains instructions, context, examples, and/or a user query. Everything the model knows about the task at call time is in the prompt; the model cannot access information outside the context window.

**Prompt engineering**
The practice of designing, testing, iterating, and versioning prompts to steer model behavior toward a desired output. Treated as a software engineering discipline: prompts are testable, versionable artifacts, not ad hoc text.

**Prompt template**
A reusable prompt structure with defined slots for dynamic values (user input, retrieved documents, tool outputs). Templates separate static instructions from runtime-variable content and enable consistent structure across requests.

**ReAct**
A prompting pattern that combines reasoning and acting in a loop: the model alternates between a Thought (reasoning about what to do), an Action (calling a tool), and an Observation (reading the tool result), with each step conditioned on prior observations. ReAct is the foundation of the off-the-shelf agent SDKs most teams use today — the Vercel AI SDK, Anthropic's Agent SDK, and OpenAI's Agents SDK all implement variants of this loop out of the box.

**System prompt**
The part of the prompt that defines the model's role, behavior, policies, and constraints for the entire interaction. Set by the developer, not the user. In production agents, the system prompt is the primary artifact that MLAs author and version.

**User prompt**
The per-turn input from the end user — the question, request, or message the agent is responding to in a given turn.

**Zero-shot**
Prompting the model to perform a task using only instructions, with no examples. Zero-shot works well for tasks the model has seen frequently during training; it degrades on niche or unusual formats.

---

## 3. Context Engineering

**Context engineering**
The discipline of deciding what information to put in the context window — and how to assemble, compress, and position it — so the model has exactly what it needs to produce a correct output. Encompasses retrieval, memory, tool outputs, and dynamic injection. Broader than prompt engineering: it covers the full information environment, not just the instructions.

**Context failure modes**
Categories of ways that injected context can degrade model behavior: context poisoning (stale/incorrect data injected as ground truth), context distraction (too much loosely relevant content unfocuses the model), context clash (contradictory facts with no conflict resolution), and token wastage (injecting content that the model never uses).

**Groundedness (faithfulness)**
The degree to which a model's answer is supported by the context it was given. A grounded answer contains only claims traceable to the retrieved documents or provided data. The inverse failure — the model ignores correct context and fabricates an answer — is the most dangerous form of hallucination in RAG systems.

**Long-term memory**
Information persisted across sessions in an external store (vector database for semantic recall, relational database for structured data) and retrieved back into context when relevant. Analogous to disk vs. RAM: persistent but requires an explicit read step to become available to the model.

**RAG (Retrieval-Augmented Generation)**
An architecture where relevant documents or facts are retrieved from an external source at runtime and injected into the prompt before the LLM call. RAG grounds the model's answer in external knowledge, reducing hallucination and extending the model beyond its parametric knowledge cutoff.

**Short-term memory**
The conversation history that lives inside the context window during a session — the model reads it directly. Ephemeral (disappears when the session ends) and bounded by the context window size. Not a separate store from the context window; just the part of it that holds the running dialogue.

**Vector search**
A search method that finds documents semantically similar to a query by comparing their embeddings in vector space. Used as the retrieval mechanism in RAG pipelines. A vector database (e.g., ChromaDB, Pinecone) stores pre-computed embeddings and returns the nearest neighbors to a query embedding.

---

## 4. Tools, Agents, and Orchestration

**Agentic system**
A software system in which an LLM takes sequences of actions — including tool calls — to complete a task across multiple steps and potentially multiple context windows. Unlike a single-turn LLM call, an agentic system maintains state, makes decisions, and takes actions with real-world consequences.

**Guardrails**
Validation and filtering applied to LLM inputs or outputs to enforce safety, format, and policy constraints. Examples: blocking outputs that fail a format check, filtering content policy violations, enforcing structured output schema compliance. Guardrails operate as a layer on top of the model, not inside it.

**Human-in-the-loop**
A design pattern where agent execution is paused to request human approval or input before continuing — typically for high-impact, irreversible, or ambiguous actions. An explicit control mechanism to prevent autonomous agents from taking destructive actions without oversight.

**Multi-agent orchestration**
A pattern where multiple specialized agents collaborate — one acting as an orchestrator routing tasks to others, or agents operating in a hierarchy or pipeline. Useful when a task is too broad for a single agent's context or expertise.

**Orchestration layer**
The code that manages the ReAct loop: sequencing LLM calls and tool executions, maintaining state, handling errors, and ensuring durable execution. At Gorgias, this is a custom ADK running on Temporal.io.

**Prompt injection**
An attack in which malicious instructions are embedded in user input or retrieved content, causing the model to treat attacker text as authoritative instructions. Particularly dangerous in RAG and agentic pipelines where external content is automatically injected into the context.

**Structured output**
A model response constrained to a machine-readable format (typically JSON) matching a pre-defined schema. Used when downstream systems need to parse the response programmatically. Structured outputs are more reliably enforced by the API harness (schema validation) than by prompt instructions alone.

**Subagent**
A specialized agent invoked by a parent agent to handle a specific subtask in isolation. Subagents allow parallel workstreams, isolated context, and specialization. Spawning subagents unnecessarily adds overhead; they are appropriate when tasks are independent, require isolated context, or can run in parallel.

**Tool (function call)**
A capability exposed to the LLM as a callable function with a defined schema. The model decides when to invoke a tool, what arguments to pass, and how to interpret the result. Tools let deterministic systems handle operations that require precision (calculation, database lookup, API call) while the LLM handles reasoning.

**Tool schema**
The name, description, and JSON parameter definition of a tool as seen by the LLM. The model selects and calls tools based on this schema alone — it does not see the implementation. A clear, accurate schema is critical to correct tool selection.

---

## 5. Prompt Management

**Playground**
An interactive environment for replaying a recorded conversation with a modified prompt, without running a full evaluation suite. The engineer loads a real past conversation, edits the system prompt, and re-runs the LLM call to observe the change in output. The playground is the fastest iteration loop in the MLA workflow — used to debug a specific failure before committing to a full experiment.

**Prompt caching**
A provider-level optimization (e.g., Anthropic's `cache_control`) that caches the model's internal state for a fixed prompt prefix across requests. When many requests share the same system prompt, prefix caching reduces both latency and cost by avoiding recomputation of the shared portion.

**Prompt label**
A named pointer to a specific prompt version (e.g., `staging`, `prod`). Labels are used to promote a version through environments without changing the identifier callers use. Rolling back = moving the label to a prior version; no code change required.

**Prompt registry**
A centralized store for versioned prompts, decoupled from the application code that uses them. Prompts are fetched by name and version at runtime, meaning a prompt can be updated without a code deployment. The registry tracks metadata: author, target model, purpose, test results.

**Prompt versioning**
Treating prompts as immutable, versioned artifacts — like source code. Each change creates a new version; existing versions are never edited in-place. Versioning enables rollback, auditability, and reproducibility of any past agent behavior.

**Rollback**
Reverting the active prompt in production to a prior version by moving the production label back to a previous entry in the prompt registry. Because prompt labels are pointers to a version, a rollback requires no code deployment — it is a runtime configuration change that takes effect immediately.

---

## 6. Evaluation

**Annotation**
The human review and labeling of model outputs, used to create or validate ground truth in the golden dataset. Annotators assess outputs against a rubric and assign labels or scores. Annotation is the calibration anchor for automated scorers: LLM-as-a-judge quality is measured by how closely it agrees with human annotation on the same examples.

**CI-gating evaluation (regression testing)**
An automated evaluation run triggered in CI (e.g., a GitHub Actions workflow) that re-runs the eval suite on every change and blocks the deploy if any previously-passing case fails or a defined quality threshold isn't met. The "regression" framing emphasizes catching cases that used to pass; the "CI-gating" framing emphasizes that it's an automated quality gate in the deployment pipeline. Same mechanism, different lens.

**Deterministic metric**
An evaluation metric computed by a rule or algorithm, producing a stable score with no LLM involvement. Examples: exact match, JSON schema validation, functional correctness (run code against unit tests), BLEU/ROUGE n-gram overlap. Reliable but requires a reference answer and cannot assess open-ended quality.

**Evaluation suite (eval suite)**
The full collection of test cases, metrics, and graders used to assess an agent's quality. Running the eval suite produces a structured result — scores per test case, per metric — that can be compared across prompt versions or model configurations.

**Experiment**
A single run of the evaluation suite under a specific configuration — a particular prompt version, model, or parameter set. An experiment is not a test case; it is the act of executing test cases and recording results for that configuration. Experiments are compared against each other (e.g., prompt v2 vs. prompt v1) to make promotion decisions. Most evaluation platforms (Braintrust, Langfuse, LangSmith) use "experiment" as the primary unit of comparison.

**Golden dataset**
A human-curated collection of representative inputs paired with expected outputs or behaviors. The ground truth for offline evaluation. Must be built deliberately — not generated purely from production heuristics — and reviewed by domain experts. It is the single most important data asset for iterating on agent quality.

**LLM-as-a-judge**
Using a capable LLM to evaluate the output of another LLM (or the same one) at scale. The judge is given a rubric and produces a score with reasoning, enabling evaluation of open-ended, qualitative outputs where deterministic metrics fall short. Known risks include verbosity preference, position bias, and sycophancy — mitigated with structured rubrics and calibration against human labels. Judge quality is measured by agreement with human annotators on the same examples; ~80% agreement is enough for directional signal but not enough to be a sole quality gate.

**Offline evaluation**
Testing agent or model behavior against a fixed dataset before any change reaches production. The primary mechanism for validating prompt changes, model swaps, or parameter adjustments. Offline evaluation is fast and repeatable — it runs on historical or curated data, not live traffic.

**Offline experimentation**
The iterative loop of swapping a prompt (or model, or parameters), running it against the golden dataset, and comparing the result against the current baseline. The target cycle time is fast enough that an MLA can make progress multiple times per day.

**Online evaluation**
Automated evaluation running continuously against live production traces — not on a held-out dataset. Online evaluation monitors the agent's behavior on real traffic. Alerts fire when metrics degrade. It is not triggered manually; it runs automatically as traces arrive.

**Qualitative metric**
An evaluation metric that assesses properties like tone, coherence, helpfulness, or policy compliance — things that cannot be reduced to a rule. Typically scored by a human annotator or an LLM-as-a-judge using a defined rubric.

**Rubric**
The structured scoring framework a scorer applies. A rubric defines the dimensions being assessed (e.g., helpfulness, tone, policy compliance — kept independent so a rater doesn't conflate two judgments into one), the levels per dimension (binary is the safest default), and an anchor example per level so different raters interpret the levels the same way. The same rubric is used by humans and LLM-as-a-judge scorers, which is what makes their outputs comparable.

**Scorer / Grader**
The function or process that assigns a score to a model output for a given test case. Scorers can be deterministic (a code function checking format or exact match), model-based (an LLM-as-a-judge), or human (an annotator reviewing the output). A single test case can be scored by multiple scorers simultaneously. The terms "scorer" and "grader" are used interchangeably across evaluation platforms.

**Test case**
A single entry in the evaluation suite: a specific input (or sequence of inputs) paired with the expected agent behavior or outcome. Test cases cover control cases (unambiguous baselines), edge cases (known past failures), and capability boundary cases (when the agent should escalate or refuse). Test cases are versioned alongside prompts.

---

## 7. Observability and Monitoring

**Annotation queue**
A workflow that routes selected production traces to a human reviewer for labeling. Each trace gets a label against the rubric, which then feeds the golden dataset and LLM-as-a-judge calibration. A queue that isn't worked on a regular cadence is a backlog, not a flywheel.

**Data flywheel**
The feedback loop where production traces are reviewed and promoted into the golden dataset, so future evaluations get more representative over time. The flywheel only turns if signal capture, trace selection, annotation, and dataset enrichment each have a clear owner and a regular cadence.

**Failure taxonomy**
A versioned, owned document listing every known failure category for the agent, each with a definition, an exemplar trace, current frequency, and the open improvement(s) addressing it. The taxonomy is the agreed-upon map of what's wrong with the agent today and what to fix next — it evolves as categories are resolved, merged, or added.

**Observability**
The ability to understand what an agent is doing at runtime, based on the data it emits (traces, spans, logs, metrics). In LLM systems, observability covers: what was sent to the model, what came back, how long it took, how much it cost, and what quality scores were attached. Without observability, debugging and iteration rely on guesswork.

**Production monitoring**
Dashboards and alerts tracking aggregate agent health metrics in real time: error rates, latency percentiles, cost per conversation, quality score distributions. Distinct from trace-level debugging: monitoring surfaces trends and anomalies across the full traffic volume.

**Promotion**
Taking an annotated production trace and inserting it into the golden dataset as a new test case. Promoted traces carry provenance (source trace, annotator, date, failure category) so the dataset stays auditable. Not every annotation is promoted — filters remove noisy or redundant traces first.

**Span**
A single operation within a trace — one LLM call, one tool invocation, one retrieval step. Spans are the children of a trace and can nest. Span-level data lets engineers pinpoint exactly which step in a pipeline degraded quality or increased latency.

**Trace**
A structured record of a complete agent execution: all LLM calls, tool invocations, inputs, outputs, latencies, token counts, and costs, captured end-to-end in a hierarchical structure. A trace is the atomic unit of observability for an agentic system.

**Trace selection**
The routing layer that decides which production traces enter the annotation queue. In practice, several sampling strategies run in parallel (random baseline, low eval score, escalations, negative user feedback, uncertainty sampling) so the queue isn't biased toward one type of failure.

---

## 8. Deployment and Lifecycle

**A/B testing (prompts)**
Running two prompt variants simultaneously on live traffic — routing a configurable percentage of requests to each — and measuring the impact on evaluation metrics. Enables data-driven promotion decisions rather than offline-only validation.

**AI agent development lifecycle**
The end-to-end process for building an AI agent: Agent Design → Prompt Engineering → Dataset Curation → Offline Experimentation → CI-Gating Evaluation → Deployment → Production Monitoring and Online Evaluation → Iteration. The lifecycle is tool-agnostic — it describes what must happen, not how a specific platform implements it.

**Canary deployment**
A deployment strategy where a new version is exposed to a small percentage of traffic first, before full rollout. If metrics degrade on the canary, the change is rolled back without impacting most users.

**Cost tracking**
Per-call, per-agent, and per-conversation visibility into LLM spending in USD, broken down by model, token type (input/output), and tool call. Essential at production volume (hundreds of thousands of conversations per day) to detect cost spikes from prompt changes or model migrations before they become incidents.

**LLM router (LLM gateway)**
A service that routes LLM API calls to one or more providers (OpenAI, Anthropic, Gemini, open-source models), centralizing concerns like authentication, model fallback, streaming, cost tracking, and rate limiting. Acts as an abstraction layer between application code and LLM providers. At Gorgias, the internal equivalent is MLGateway.

**LLMOps**
The operational discipline for deploying, monitoring, and iterating on LLM-powered systems in production. Extends MLOps with LLM-specific concerns: prompt versioning, context management, qualitative evaluation, and cost governance at token-level granularity.

**Streaming**
Delivering LLM output tokens to the client progressively as they are generated, rather than waiting for the full response. Streaming reduces perceived latency for end users by showing the first tokens in milliseconds. Requires the LLM router, the serving layer, and the client to support server-sent events (SSE).

---

## 9. Fine-tuning

**Fine-tuning**
Continuing the training of a pre-trained foundation model on a curated domain-specific dataset to bake new behavior into the model's weights. Fine-tuning is the last resort after prompt engineering and RAG have been exhausted — it requires data, compute, and ongoing maintenance. A fine-tuned model produces a new model artifact that must itself be versioned, evaluated, and served.

---

## 10. Red Teaming and Security

**Jailbreak**
An adversarial input that causes the model to bypass its safety instructions or content policies — typically through roleplay framing, formatting tricks, or persuasion. A jailbreak is a subset of what red teaming is designed to detect.

**PII masking**
Stripping or redacting personally identifiable information from inputs before they are sent to an external LLM provider. Required when agent conversations contain customer data and the provider's data handling does not meet compliance requirements.

**Red teaming**
Systematic adversarial testing of an agent to discover security vulnerabilities and policy violations that standard evaluations miss. A red team generates adversarial prompts targeting specific failure modes (jailbreaks, prompt injection, PII leakage, toxicity generation) and evaluates the agent's responses. Treated as a recurring process, not a one-time gate.

---

## 11. Inference Performance

**Temperature**
A generation parameter that controls the randomness of token sampling. Temperature < 1 sharpens the probability distribution (more deterministic, predictable output); temperature > 1 flattens it (more random, creative output). Temperature 0 approaches greedy decoding (always picking the most probable token).

**TTFT (Time to First Token)**
The latency from sending a request to receiving the first generated token. The primary driver of perceived responsiveness in streaming systems.

---

## 12. Gorgias Context

> These terms are specific to Gorgias's internal stack and organization. They map to industry-standard concepts defined in the sections above.

**ADK (Agent Development Kit)**
Gorgias's internal custom framework for building and running AI agents. The ADK implements the ReAct loop and tool execution on top of Temporal.io, providing durable, fault-tolerant agent execution. It is the orchestration layer that all production agents run on.

**MLA (Machine Learning Analytics)**
A Gorgias role focused on agent quality. MLAs work on prompt engineering, golden dataset curation, offline evaluation, and promoting prompts to production.

**MLE (Machine Learning Engineer)**
A Gorgias role focused on ML infrastructure and model concerns: model selection, fine-tuning decisions, and evaluation infrastructure.

**MLGateway**
Gorgias's internal LLM router: the service that receives LLM API calls from the ADK and routes them to a provider (currently OpenAI and Gemini). MLGateway is the component being evaluated for replacement as part of the SPADE initiative — its current limitations include no streaming, limited model coverage, no vision support, and ongoing maintenance overhead.

**PromptLayer**
The external SaaS tool currently used for prompt versioning and the MLA playground (replaying traces with modified prompts). PromptLayer is being fully replaced as part of the SPADE initiative due to reliability issues and missing features (no selective re-run, no metric visualization, no A/B testing support).

**SWE (Software Engineer)**
A Gorgias role focused on the engineering layer of AI agents: the agent loop, tool implementations, tool schemas, and integration with external systems via the ADK.

**Temporal.io**
The workflow orchestration engine underlying the ADK. Temporal provides durable execution: if an agent step fails partway through, Temporal can replay it from the last successful checkpoint rather than starting over. It is the reliability primitive that makes long-running (~1 minute) agent conversations safe at production volume.

---

## 13. References

The resources below collectively cover the terms in this glossary. They are grouped by topic, then ordered roughly from foundational to applied within each group.

### Agent architecture and design

**[LLM Powered Autonomous Agents — Lilian Weng](https://lilianweng.github.io/posts/2023-06-23-agent/)**
The canonical reference on agentic system architecture. Covers the ReAct loop, tool use, short-term and long-term memory, multi-agent patterns, and subagent design.

**[Building Effective Agents — Anthropic](https://www.anthropic.com/research/building-effective-agents)**
Anthropic's practical guide to agent design, based on production deployments across dozens of teams. Covers orchestration patterns, human-in-the-loop, guardrails, and when to use agents vs. simpler workflows.

**[Patterns for Building LLM-based Systems & Products — Eugene Yan](https://eugeneyan.com/writing/llm-patterns/)**
A field-tested catalog of patterns covering evaluation, retrieval-augmented generation, fine-tuning, caching, guardrails, defensive UX, and the feedback collection patterns that feed a data flywheel.

### Prompt engineering

**[Prompt Engineering Overview — Anthropic](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)**
Covers the full prompting toolkit: system prompt, user prompt, few-shot, zero-shot, chain-of-thought, prompt templates, structured output, and temperature.

### Production engineering

**[Building LLM Applications for Production — Chip Huyen](https://huyenchip.com/2023/04/11/llm-engineering.html)**
The definitive production engineering reference. Covers LLMOps, RAG, evaluation, hallucination, golden datasets, cost tracking, streaming, production monitoring, canary deployments, and the data flywheel.

### Evaluation and error analysis

**[LLM Evals: Everything You Need to Know — Hamel Husain](https://hamel.dev/blog/posts/evals-faq/)**
A comprehensive FAQ on evaluating LLM applications. Covers golden dataset construction, scorer design, LLM-as-a-judge calibration, multi-turn evaluation, and the role of error analysis in everything else.

**[Why is error analysis so important in LLM evals, and how is it performed? — Hamel Husain](https://hamel.dev/blog/posts/evals-faq/why-is-error-analysis-so-important-in-llm-evals-and-how-is-it-performed.html)**
The industry-standard methodology for turning annotated traces into a failure taxonomy — the source of the workflow that underlies the failure taxonomy entry above.

**[How can I efficiently sample production traces for review? — Hamel Husain](https://hamel.dev/blog/posts/evals-faq/how-can-i-efficiently-sample-production-traces-for-review.html)**
Sampling strategies for trace selection beyond uniform random — clustering, sorting by user feedback, prioritizing high-probability failure patterns.

**[How to Calibrate LLM-as-a-Judge with Human Corrections — LangChain](https://www.langchain.com/articles/llm-as-a-judge)**
The mechanics of judge calibration: collect human corrections in an annotation queue, measure agreement, recalibrate when agreement drops.

**[LLM Evaluation Rubrics: Templates, Examples, and Reviewer Calibration — Twine](https://www.twine.net/blog/llm-evaluation-rubrics/)**
Practical guidance on rubric design: define the task, separate dimensions cleanly, anchor scores with evidence.

### Data flywheel and continuous improvement

**[Data Flywheels for LLM Applications — Shreya Shankar](https://www.sh-reya.com/blog/ai-engineering-flywheel/)**
The foundational framework for closed-loop self-improving LLM applications. Evaluation → monitoring → continual improvement. The source of the "validators as probes" framing, dynamic few-shot retrieval based on past traces, and the principle that humans must be in the loop on a regular cadence.

**[Agent-in-the-Loop: A Data Flywheel for Continuous Improvement in LLM-based Customer Support — Airbnb (arXiv 2510.06674)](https://arxiv.org/pdf/2510.06674)**
The most concrete published implementation of a production data flywheel, run by Airbnb on real customer support traffic. Demonstrates how LLM-based pre-filtering removes 14–35% of annotations as noise and reports retraining cycles shrinking from months to weeks.

**[Data Flywheel: using your production data to build better LLM products — Rogério Chaves (LangWatch)](https://rchavesferna.medium.com/data-flywheel-using-your-production-data-to-build-better-llm-products-c01be1193a86)**
A short, principles-first introduction to the data flywheel concept. Useful as a one-page primer for collaborators who need to understand why the loop matters before getting into mechanics.

**[The LLM Evaluation Flywheel: Building a Systemic Guardrail — Paul Deepakraj Retinraj](https://pauldeepakraj-r.medium.com/the-llm-evaluation-flywheel-building-a-systemic-guardrail-cff75af643d9)**
Covers the evaluation-side flywheel: how production traces become the ground truth for the next iteration, with emphasis on the relationship between online evaluation and golden dataset curation.

### User feedback signals

**[User Feedback in LLM-Powered Applications — Winder.ai](https://winder.ai/user-feedback-llm-powered-applications/)**
A practical taxonomy of explicit vs. implicit user feedback signals, with the empirical observation that explicit feedback rarely exceeds 1% of interactions.

**[User Feedback in Human-LLM Dialogues: A Lens to Understand Users But Noisy as a Learning Signal — arXiv 2507.23158](https://arxiv.org/abs/2507.23158)**
Research showing that user feedback is highly informative for understanding user intent but noisy as a direct training signal — which is why feedback is routed through annotation rather than directly into training data.

### LLMOps platforms (vendor documentation)

**[Langfuse Documentation](https://langfuse.com/docs)**
The reference for the LLMOps platform layer. Covers traces, spans, observability, prompt versioning, prompt registry, prompt labels, online and offline evaluation, LLM-as-judge, test case management, and CI-gating.

---

## 14. Validation Criteria

Before adding a term to this glossary, it must pass all of the criteria below. When updating an existing term, criteria 3–7 still apply. When removing a term, also verify criterion 8.

**1. Industry-standard vocabulary.**
The term must be how the AI/LLM engineering industry actually refers to this concept, with consistent usage across at least two reputable sources (vendor docs, well-known practitioner blogs, foundational papers). No invented vocabulary. The only exception is Section 12 (Gorgias Context), which exists explicitly to map internal names to industry-standard concepts defined elsewhere in the glossary.

**2. Practical relevance — 4 or 5 out of 5.**
A Gorgias engineer building or iterating on AI agents must reasonably encounter or use this term in their day-to-day work. Score 1 (background trivia, never used) to 5 (used daily). Terms scoring 1–3 do not belong here, no matter how technically interesting. When in doubt, leave it out — the playbook can reference deeper material.

**3. No duplicates or significant overlap.**
Before adding, check whether an existing entry already covers the concept. If two terms describe the same mechanism with different framing (e.g., regression testing vs. CI-gating evaluation), merge them into a single entry that names both framings rather than creating two.

**4. Definition is 2–3 sentences, not a methodology dump.**
Glossary entries define the term and its purpose. They do not prescribe processes, enumerate every sub-concept, or substitute for the playbook. If a definition needs more than 3 sentences, it is trying to be a methodology document — move the methodology to the playbook and keep only the definition here.

**5. Technically accurate and current.**
The definition must be correct. References to specific tools, models, vendor APIs, or numerical thresholds must be accurate at time of writing and should be re-audited on each glossary update.

**6. Understandable by an engineer with vague AI knowledge.**
The target reader is a Gorgias engineer who knows roughly what an LLM is but is not steeped in the AI stack. The definition should land without requiring the reader to go look up other concepts elsewhere. Jargon used inside a definition must either be common engineering vocabulary or itself defined in this glossary.

**7. Cross-references resolve.**
If the definition mentions another term defined in this glossary, that term must actually exist in the glossary. Use the term's exact wording (e.g., "ReAct loop," not "agent loop," since Agent loop is not an entry here).

**8. Removals do not orphan cross-references.**
When removing a term, search the rest of the glossary (including the References section) for mentions of it. Either rewrite the affected entries to avoid the removed term, or do not remove it.
