# Phase 5 — Evaluation Design

> **Output:** Golden dataset + scoring rubric + evaluation methods
> **Gate:** Golden dataset covers baseline, edge cases, and boundary conditions — and has been reviewed by a domain expert — before iteration begins.

---

## What this phase is

Define, before you iterate on anything, how you will know if the agent is working. Without an evaluation baseline, you cannot tell whether a prompt change is an improvement or a lateral shift. You might fix one failure while introducing another.

> Build your golden dataset before you start debugging. This is the single most important process discipline in agent development.

### What is a golden dataset?

A golden dataset is a curated collection of inputs paired with their expected outputs (or evaluation criteria), validated by someone who deeply understands what "correct" looks like for your specific task. It is not a static artifact — it is a **living document** that you:

- **Create** before iteration begins, seeded with the cases you can anticipate
- **Enrich** whenever a new failure mode is discovered — in iteration or in production
- **Revise** when the task definition changes or when domain knowledge evolves
- **Maintain** with domain expert involvement: the people who understand the task at a business level, not just technically

The golden dataset is the ground truth for every decision in the agent's lifecycle. Every prompt change, model swap, and architectural decision is measured against it. It is what makes iteration systematic rather than intuitive.

> Without a golden dataset, you are flying blind. With a stale or uncurated one, you are flying with a broken compass.

---

## Checklist

### Build the initial golden dataset

- [ ] **Involve a domain expert from the start.** The person writing test cases must understand what a correct output looks like — not just technically, but in the context of your actual use case. This is often not the engineer building the agent. Identify your domain expert(s) before this phase begins.

- [ ] **Write control cases.** Unambiguous inputs with a clearly correct output. These confirm baseline behavior and serve as regression tests as the prompt evolves.

- [ ] **Write edge cases.** Scenarios where you expect the agent to struggle: unusual inputs, ambiguous requests, missing information, conflicting instructions. These are how you discover failure modes before real users do.

- [ ] **Write boundary and handoff cases.** Tests for the agent's awareness of its own limits: does it escalate when it should? Does it refuse out-of-scope requests correctly? Does it recognize when to hand off to a human?

- [ ] **Make the dataset representative and diverse.** Cover the full distribution of inputs you expect in production, including the long tail. A dataset that only covers easy cases gives false confidence.

- [ ] **Write test cases before building, not after.** This forces clarity about what the agent should and should not do, and prevents overfitting implementation to a narrow set of cases.

- [ ] **Store the golden dataset in version control.** Like prompts and code, changes must be tracked. You need to know when a case was added, why, and by whom.

### Keeping the dataset alive

A golden dataset that nobody owns drifts out of sync with reality within weeks. These habits keep it useful — they are lightweight, not bureaucratic:

- [ ] **Name someone who owns it.** One domain expert reviews and approves changes. This is an ongoing responsibility, not a one-off handoff. Without a named owner, the dataset quietly rots and you find out only when an eval score stops correlating with reality.

- [ ] **Add new cases when you discover new failure modes.** Every time offline testing or production surfaces a failure the dataset did not catch, add a case. Make this a habit, not a project — if it requires a meeting, it will not happen.

- [ ] **Revisit existing cases on a cadence.** As the task evolves, some cases become outdated, misleading, or wrong. A quick pass every major prompt version (or quarterly, whichever comes first) is enough to keep things honest.

- [ ] **Keep the bar high on what counts as "golden".** Not every test case you run during iteration belongs in the golden dataset. Golden cases are the ones a domain expert has explicitly signed off on as representative ground truth. Everything else stays in an exploratory pile.

- [ ] **Organize the golden dataset into named subsets — do not create entirely separate datasets.** You want to run all subsets before every change, so separating them creates the risk of one being forgotten. Instead, use one dataset with clearly labeled subsets, each with its own evaluation method and reviewer:

  | Subset | What it covers | Evaluation method | Reviewer |
  |---|---|---|---|
  | `control` | Baseline, unambiguous cases | Deterministic or LLM judge | Domain expert |
  | `edge` | Hard, unusual, or ambiguous inputs | LLM judge | Domain expert |
  | `boundary` | Escalation, refusal, handoff behavior | LLM judge | Domain expert |
  | `adversarial` | Prompt injection, jailbreaks, PII leakage, policy violations under pressure | Safety-specific judge or human review | Security / trust & safety reviewer |

  Subsets can be run selectively during fast iteration cycles, but the full dataset must pass before any deployment.

### Choose evaluation methods

Each type of output needs an appropriate evaluation method. Use a portfolio — no single method is sufficient.

- [ ] **Use deterministic checks for verifiable outputs.** Code that runs, JSON that validates, facts that can be checked against a source — write automated pass/fail checks. Fast, cheap, unambiguous.

- [ ] **Use LLM-as-judge for quality and behavior.** For outputs requiring subjective assessment (tone, helpfulness, policy compliance, reasoning quality), use a capable model as an automated judge with a structured scoring rubric. Calibrate the judge against human ratings on a sample to catch systematic biases.

- [ ] **For tool-using agents, evaluate the sequence of decisions.** Final output alone is not enough. Evaluate: did the agent choose the correct tool? Call tools in the right order? Pass correct arguments? Interpret results correctly? Each tool decision is a separate failure point.

- [ ] **For multi-turn agents, evaluate at two levels.**
  - *Turn-level*: which specific step caused a failure?
  - *Conversation-level*: was the overall task completed? Was the dialogue coherent?

- [ ] **Include latency and cost as evaluation dimensions.** Track tokens, cost, and response time per test case. A model with a small quality improvement but 2× latency may be worse overall for real-time use.

### Red teaming

- [ ] **Test for adversarial inputs before shipping.** Standard evals assume cooperative users. Before launch, explicitly test for:
  - **Prompt injection** — malicious instructions embedded in retrieved content that hijack agent behavior
  - **Jailbreaks** — attempts to override safety instructions through adversarial framing
  - **PII leakage** — the agent exposing sensitive data in its outputs
  - **Policy violations under pressure** — the agent violating stated constraints when a user pushes back

---

## Output

A **golden dataset** containing:
- Test cases by category (control / edge / boundary / adversarial), validated by a domain expert
- Scoring rubric for LLM-as-judge evaluations
- Documented evaluation methods per test category
- Version history with the reason each case was added or modified
- Named owner(s) responsible for ongoing maintenance

> 📋 Template opportunity: Golden dataset schema (input · expected output or criteria · category · added by · reason · date).
