# AI Agent Stack exploration

## Purpose

This document is the North Star for the [SPADE](https://coda.io/@gokulrajaram/gokuls-spade-toolkit) we will write. It captures the current state, the vision, the AI agent development lifecycle we want to define, and the tooling investigation we need to conduct. The lifecycle is the primary and stable output of this work. The tooling stack is the secondary output and may evolve over time.

---

## Current State and Pain Points

### Team Structure (for context only)

Today, building an AI agent involves multiple roles: Software Engineers (SWEs) who own the orchestration layer, Machine Learning Analytics (MLAs) who are primarily prompt engineers, Machine Learning Engineers (MLEs) who handle more technical ML concerns, and Product who are domain experts. The friction between these roles — especially between the orchestration side and the prompt/model configuration side — slows down iteration and creates unnecessary dependencies.

**For the purpose of this investigation, assume a single engineer works end-to-end on all aspects of building an AI agent.** Ownership and governance between roles is out of scope for now.

### MLGateway

Our internal LLM routing service is called MLGateway. It exposes API endpoints that route calls to LLM providers. Its current limitations are severe:

- Only ~5 models available
- No streaming support
- Limited vision and multi-modal support
- Limited file type support
- Blocks access to new provider capabilities and open-source models
- Only supports OpenAI and Gemini today
- Requires ongoing internal software engineering effort to maintain — it is not a commodity service but a custom codebase that must be kept alive

### Prompt Management

Prompts currently live in PromptLayer with versioning. PromptLayer is being fully replaced — it is unreliable and not fit for purpose. A full replacement is required.

### Evaluation

Online evaluation exists but is built in-house and poorly implemented. There is no reliable offline evaluation pipeline. Evaluation datasets are generated dynamically from production traces using heuristics, which results in variable quality. There is no golden dataset. CI-gating evaluation does not exist in a proper form — only some GitHub Actions making ad hoc calls.

### Tracing

There is no end-to-end tracing in place today.

### Scale

- **Agents in production:** ~4
- **Volume:** ~150,000 conversations per day
- **Agent runtime:** ~1 minute per run
- **Orchestration:** Custom ADK running on Temporal.io (reliable, durable execution)
- **LLM providers:** OpenAI and Gemini (open-source models not yet tested but desired)

---

## Vision

Any engineer — regardless of whether they specialize in orchestration, prompt engineering, or ML — should be able to build, evaluate, ship, and iterate on an AI agent end-to-end, with expert consultation available when needed. Product stakeholders participate in the non-technical aspects, primarily dataset review and domain validation.

This initiative has two outputs:

1. **The AI Agent Development Lifecycle** — a stable, tool-agnostic process that defines how an AI agent is built from start to finish. This lifecycle should remain valid regardless of which tools are used to implement it.
2. **The Implementation** — how the lifecycle is implemented using the selected tooling stack. This is the variable part; it will be shaped by the tools we choose.

---

## Non-Negotiables

- **Industry-standard vocabulary throughout.** We do not invent terminology. Every term used must reflect common industry usage. A glossary will be produced and maintained as part of this work.
- **The lifecycle is stable and tool-agnostic.** Tools change; the lifecycle should not.
- **Only mature, production-proven tools are eligible.** We run ~300,000 conversations per day across ~4 agents in production. We cannot afford immature tooling. Every platform considered must have a demonstrated track record at scale, be in active use by large organizations, and have a stable API surface. Early-stage or experimental tools are disqualified regardless of their feature set.
- **Prompt versioning and labeling are required.** Every prompt change must be versioned and labeled. We must be able to reproduce any previous state.
- **Offline evaluation metrics must provide meaningful deployment signals.** Offline metrics inform the decision of whether to ship a change. Online evaluation monitors what matters in production — a curated subset of business and technical metrics. Not every offline metric needs to be mirrored online, but the two sets must be designed coherently. A metric used in both contexts should not require two separate definitions.
- **Cost tracking and alerting are required.** Given our volume and the risk of a prompt or model change spiking cost, we must have visibility and alerting on LLM spend.

---

## The AI Agent Development Lifecycle

The following phases define the end-to-end process of building an AI agent. This is the stable core of this initiative.

### 1. Agent Design

Define the agent's purpose, scope, inputs and outputs, the tools it can use, constraints, and success criteria. This is the spec phase.

### 2. Prompt Engineering

Write, iterate on, and version system prompts, few-shot examples, and any other LLM instructions. Prompts are versioned and labeled. Changes are tracked.

### 3. Dataset Curation

Build and maintain a golden dataset of representative inputs and expected outputs. This dataset drives all offline evaluation. It must be curated deliberately — not generated purely from heuristics. Production traces can inform it but must be reviewed before inclusion.

### 4. Offline Experimentation

Test combinations of prompts, models, and parameters against the golden dataset. Compare runs. This loop must be fast and accessible — an MLA should be able to swap a system prompt, run it against the dataset, and compare results to the current baseline with minimal friction.

Evaluation tasks can range in complexity. A simple task tests a single prompt against a dataset and grades the output. A more complex task invokes an agent, waits for a single turn to complete, and grades the response. An even broader task might simulate a full conversation — for example, a shopper simulation that interacts with the agent across multiple turns — and then grades the overall outcome. The platform must support this range of task complexity.

### 5. CI-Gating Evaluation

Before any change ships to production, an automated evaluation run executes against the golden dataset and must pass a defined threshold to unblock the deployment. This is a nice-to-have capability — the platform should support it, but it is not a hard requirement for initial adoption.

### 6. Deployment

Roll out the agent or an updated configuration to production. Includes support for canary deployments and A/B testing to progressively expose changes and enable rollback if metrics degrade. (Detail to be specified later.)

### 7. Production Monitoring and Online Evaluation

Collect end-to-end traces from every production run. The platform automatically runs evaluation metrics against production traces — no manual triggering. Alerts fire when metrics degrade or costs spike.

### 8. Iteration

Use production trace data and evaluation results to improve the dataset, refine prompts, reconsider model selection, or adjust tool definitions. This feeds back into Phase 3 and Phase 2, creating a continuous improvement loop.

---

## Tooling Investigation

### Track 1 — LLM Router (replacing MLGateway)

We need a mature solution for routing LLM calls across providers. A single-platform option covering both routing and observability should be investigated first, but a best-of-breed combination is equally acceptable.

**Requirements:**

- Streaming support
- Vision and multi-modal support
- Wide model and provider coverage, including open-source models
- Proxy mode vs. SDK client mode — both architectures must be evaluated (proxy adds a network hop but centralizes concerns; SDK client routes directly with no proxy dependency)
- Cost tracking and alerting per model, per agent, per run
- Compatibility and integration with the chosen observability platform — the router layer must appear in traces
- Transparent SLA, SLO, and availability guarantees

**Suggested tools to investigate:** LiteLLM, Portkey, Helicone, OpenRouter (non-exhaustive — expand during research)

### Track 2 — LLM Observability and Evaluation Platform (replacing PromptLayer and in-house eval)

**Requirements:**

- Prompt versioning and labeling
- Fast experiment loop: swap a prompt, run against the golden dataset, compare to baseline
- Offline evaluation with CI integration (blocking gate — nice-to-have)
- Online evaluation: automatically run metrics against production traces without manual triggering
- Metrics defined in offline evaluation should be reusable in online monitoring without requiring separate definitions; online monitoring covers a curated subset of what matters in production
- Dataset management: storage, curation, and versioning of golden datasets
- Cost tracking and alerting (if not covered by the router layer)
- Data flywheel tooling: ability to promote production traces into the dataset (nice-to-have; can be manual if not provided)
- Compliance considerations: traces will contain customer conversation data — self-hosted deployment options must be evaluated (compliance requirements TBD)
- Compatibility with a custom ADK on Temporal.io via manual instrumentation (OpenTelemetry or platform SDK)
- Experiment primitives sufficient for engineers to build self-service interfaces for non-technical users: at minimum, the platform must expose versioned prompt management, dataset runs, and side-by-side scoring via API or SDK so that a thin wrapper can be built without re-implementing eval infrastructure. A native low-friction UX for non-technical users is a positive signal but not a hard requirement.

**Suggested tools to investigate:** Langfuse, Braintrust, LangSmith, OPIK (non-exhaustive — expand during research)

### Evaluation Criteria for All Platforms

Every platform will be assessed on:

- Feature coverage against the requirements above
- **Maturity and adoption (hard filter):** the platform must be in active production use by large organizations. Uptime history, known incidents, and time-in-market are all signals. Immature tools do not proceed to full evaluation.
- Cost at our scale — see dedicated section below
- Self-hosted vs. SaaS availability
- Quality and responsiveness of customer support
- Community activity and ecosystem health
- Frequency and quality of product updates
- Compatibility with our custom ADK / Temporal.io stack
- Integration story between the router layer and the observability platform

---

### Cost Evaluation Methodology

Cost is a first-class evaluation criterion and must be assessed rigorously, not estimated loosely. Each platform prices differently — per trace, per span, per token processed, per seat, per API call, or a combination — so a like-for-like comparison requires translating our actual production usage into each platform's pricing model.

**Production usage baseline (inputs to the cost model):**

- ~150,000 conversations per day
- ~4 agents, each with a different profile (number of LLM calls per conversation, average token counts, tool call frequency, trace depth)
- Agent runtime: ~1 minute per conversation on average
- Each conversation produces at minimum: 1 trace, N spans (to be measured per agent), M LLM calls with input/output token counts

Before evaluating any platform, we must instrument or estimate the following per-agent metrics:

- Average number of LLM calls per conversation
- Average input and output token count per LLM call
- Average number of spans per trace
- Average number of tool calls per conversation
- Evaluation frequency: how often offline evaluation runs and against how many dataset entries

**Current costs (reference baseline):**

| Tool                                         | Monthly     | Annual       |
| -------------------------------------------- | ----------- | ------------ |
| MLGateway (internal infra + eng maintenance) | $20,000     | $240,000     |
| PromptLayer                                  | $7,500      | $90,000      |
| **Total current spend**                      | **$27,500** | **$330,000** |

**Cost projection approach:**

For each platform, compute:

1. **Ingestion cost** — cost of sending traces and spans at production volume (~150,000 conversations/day × spans per conversation)
2. **Storage cost** — cost of retaining trace data for a defined retention window (retention period TBD)
3. **Evaluation cost** — cost of running LLM-as-judge or automated metrics, both online (production traces) and offline (dataset runs)
4. **Prompt management cost** — if the platform charges for prompt storage or API calls against the prompt registry
5. **Seat or user cost** — if the platform charges per user
6. **Router cost** — for the LLM routing layer specifically: cost per routed call or per token proxied, if applicable

**Total cost of ownership** must account for all of the above, not just the headline pricing tier. Platforms that appear cheap at low volume can become expensive at 150,000 conversations/day and must be stress-tested against our actual numbers.

Where a platform does not publish pricing at our scale, we will contact their sales team and document the response as part of the evaluation.

---

## Open Questions

- **Compliance level:** The required compliance posture for trace data containing customer conversations is not yet defined. Self-hosted options must remain on the table until this is resolved.
- **Data flywheel:** Whether this is handled by the platform or implemented manually is TBD pending platform evaluation.
- **Canary and A/B deployment detail:** The mechanics of prompt rollout, traffic splitting, and rollback will be specified in the implementation phase.
- **Open-source models:** No open-source models have been tested yet. The router evaluation must validate that chosen providers are accessible and well-supported.
