# Phase 6 — Iteration & Offline Testing

> **Output:** Passing eval baseline with documented scores and known failure modes
> **Gate:** Agent meets success criteria on the full eval suite before deployment.

---

## What this phase is

The core development loop: run evals, identify failures, make targeted changes, measure the result. Everything happens offline — against your test suite, not against real users. The goal is to reach a quality level that justifies the risk of shipping.

Two disciplines define this phase:

1. **Change one thing at a time.** Making multiple changes simultaneously makes it impossible to know what drove an improvement or a regression.
2. **Start simple, escalate only when evals tell you to.** The correct answer is rarely "use the biggest model" — it is more often "decompose the problem correctly."

---

## What you're actually iterating on

It is easy to read this phase as "prompt iteration" — but the unit being evaluated is the **whole agent system**, not the prompt in isolation. Your eval suite invokes the agent the same way production does: through code, with real tool calls, real model calls, and the same orchestration logic. Every test case runs the executable agent end-to-end. The output that gets scored is what the agent actually produces, not what an LLM would produce given a prompt.

That means *any* change that affects the agent's runtime behavior must pass the same offline gate before deploying. The eval discipline is identical across all of these — only the surface being changed differs:

| Change category | Examples |
|---|---|
| Prompt | System prompt edits, sub-task prompt edits, few-shot examples |
| Tool implementation | The code inside a tool changes — different query, different parsing, different error handling |
| Tool schema or catalog | A tool's description or input/output schema changes; a tool is added or removed |
| Model | Swapping the underlying model, changing temperature, changing reasoning effort |
| Agent topology | Single-agent → manager pattern; adding or removing a sub-agent; changing handoff conditions |
| Memory or context strategy | What gets summarized, what stays verbatim, when long-term memory is consulted |
| Retrieval / RAG configuration | Chunking strategy, embedding model, top-k, reranker |
| Guardrails | Threshold tuning, adding or removing a layer |

"Change one thing at a time" applies across this whole surface, not just inside the prompt. The category of the change matters for attribution: a regression after a tool implementation change has a different root cause — and a different fix — than a regression after a prompt change.

---

## How you run evals — two legitimate scopes

Eval runs come in two scopes. Both are first-class daily-use tools; the discipline is choosing the right one for what you are changing.

**Prompt-only evaluation** runs the dataset against a single LLM call — the prompt under test, with no tools, no orchestration, no downstream agent code. Most LLM observability platforms expose this directly from their UI: pick a prompt version, pick a dataset, hit run, get scored outputs in minutes. This is the fastest feedback loop you have, and it is the right tool for rapid wording, example, and output-format iteration.

**Full agent evaluation** runs the dataset through the executable agent end-to-end — real tool calls, real orchestration, real model selection, real memory and retrieval. Slower, more expensive, but the only mode that catches integration failures and the only one that validates changes outside the prompt surface.

| Use prompt-only when | Use full agent when |
|---|---|
| Iterating on wording, examples, or output format | The change touches tools, schema, topology, retrieval, memory, or guardrails |
| Isolating a prompt-driven failure from system-driven ones | Validating any multi-step or tool-calling behavior |
| Doing rapid back-to-back tries before committing a prompt edit | Reproducing a production trace |
| The agent is genuinely prompt-dominated (minimal tools, simple topology) | Final validation before deploying to production |

Two non-negotiable rules govern the relationship between the two modes:

- **The deployment gate is the full agent run.** Prompt-only is an iteration accelerator, never a deployment validator. A prompt that scores well in isolation can still produce broken behavior once it is wired back into tools, orchestration, and downstream code. No production deployment ships on prompt-only evals alone.
- **Once you change anything outside the prompt, prompt-only runs lose meaning until the full agent has re-baselined.** Tool changes, schema changes, topology changes — none of these are exercised by a prompt-only run, so a passing prompt-only score after a tool change tells you nothing about whether the agent still works.

---

## Checklist

### The iteration loop

- [ ] **Use a tiered run strategy — do not run the full golden dataset on every edit.** Running everything every time is too slow to iterate effectively. Instead, match the scope of your run to where you are in the loop:

  | Situation | How many cases | Scope |
  |---|---|---|
  | Actively debugging a specific failure | Single row — the failing case only | Match the surface being changed |
  | Rapid prompt iteration (wording, examples, output format) | The relevant subset | Prompt-only — fastest feedback |
  | Fixing a failure mode that lives in the prompt | The relevant subset (e.g. all `edge` cases) | Prompt-only first, then full agent before commit |
  | Fixing a failure mode in tools, topology, retrieval, or guardrails | The relevant subset | Full agent — prompt-only cannot exercise it |
  | Before committing any change to the agent system | Full golden dataset | Full agent |
  | Before deploying to production | Full golden dataset — non-negotiable | Full agent — non-negotiable |

  Single-row runs are for exploration and fast feedback. Full-dataset runs are for decisions. Never use a single-row result to conclude that a change is safe to ship.

- [ ] **Establish your baseline by running the full golden dataset against the full agent at the start of a new iteration session.** Before touching anything, know your current scores across all subsets. Prompt-only runs are not a substitute for the baseline — they cannot tell you where the agent stands as a system.

- [ ] **Apply general hygiene before targeting specific failures.** Before chasing individual failing cases, clean up the easy stuff across the whole agent system: structure the prompt clearly and remove outdated patches; verify tool descriptions and schemas still match what the tools actually do; remove unused tools from the catalog; ensure the output contract is explicit. This often resolves several failures at once and gives you a cleaner surface to work on.

- [ ] **Fix one failure mode at a time.** Isolate each failing test case → make a targeted change → run the relevant subset to confirm the fix → run the full golden dataset before moving on to confirm no regressions were introduced.

- [ ] **Diagnose which part of the agent system is failing before changing anything.** Not every failure is a prompt failure. A wrong final answer can come from a bad tool result, a missing piece of context, an incorrect tool selection, a topology that puts the wrong agent in charge, or a genuine model capability gap. Read the trace and identify the *first* thing that went wrong — that is the surface to change. Fixing the prompt when the tool is broken just hides the real problem.

### Escalation strategy

When a failure persists, escalate through this ordered ladder. Each step touches a different part of the agent system. Do not skip steps — a more capable model on a poorly decomposed problem is less efficient than a smaller model on a well-decomposed one.

1. **Improve the prompt first.** Better task decomposition, clearer instructions, stronger examples. Cheapest to change, cheapest to revert.
2. **Fix or extend tools.** If failures trace to wrong tool selection, ambiguous tool descriptions, or missing capabilities, the fix is at the tool layer — not in the prompt. Sharpen the description, tighten the schema, or add the tool that should exist.
3. **Improve retrieval and context.** If the agent has the right tools but is acting on incomplete or wrong information, the fix is in what context reaches the model — chunking, top-k, reranking, source coverage.
4. **Decompose the task.** Split a single overloaded prompt into multiple focused steps. This is a topology change — you are introducing more structure into the agent system.
5. **Change the topology.** If a single-agent design keeps degrading on tool selection or instruction-following because the surface is too broad, move to a manager pattern with specialized sub-agents.
6. **Escalate to a more capable model.** A better model on a well-decomposed problem is the right last move, not the first one.

> *Advanced pattern — only reach for this when the steps above are not enough.* For tasks dominated by hard constraints (formatting rules, policy requirements, one-off user preferences), a **generate-evaluate-repair loop** can outperform a single-pass prompt. A *generator* produces a first draft, an *evaluator* checks it against the constraints and emits a structured list of violations, and a *repairer* makes targeted fixes based on that list. The loop runs until the evaluator finds no violations or a retry budget is exhausted. Put runtime-variable constraints (user preferences, one-off rules) in the evaluator's instructions — not in the generator and not in code. This is powerful but adds latency, cost, and operational complexity; do not adopt it as a default architecture.

### Before exiting this phase

- [ ] **Add new cases to the golden dataset for any failure discovered during iteration.** When a failure mode surfaces that was not already in the dataset, add it — validated by the domain expert. The golden dataset should grow to reflect every meaningful failure encountered.

- [ ] **Document the agent's known failure modes.** Every agent has cases it handles poorly. Write them down. This sets expectations for the initial release, informs monitoring in Phase 8, and gives you a prioritized backlog for post-launch iteration.

- [ ] **Record the passing state of the whole agent system.** Capture the full set of versions that define your deployment-ready baseline: prompt version, model and configuration, tool versions and schemas, topology (which agents and sub-agents are wired together), memory and retrieval configuration, guardrail configuration, and the eval scores produced by this exact combination. Anything missing from this record will be impossible to roll back to.

---

## Output

A **passing eval baseline** documenting:
- Prompt version
- Model and configuration
- Tool versions
- Eval scores per category
- Known failure modes and their severity

> 📋 Template opportunity: Eval baseline record.
