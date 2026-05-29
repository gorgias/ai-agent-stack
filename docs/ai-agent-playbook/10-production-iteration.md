# Phase 10 — Production Iteration

> **Output:** Iteration log (ongoing)
> **Gate:** None — this phase runs for the life of the agent.

---

## What this phase is

The ongoing loop that closes the gap between how the agent behaves today and how it should behave. Structurally similar to Phase 6, but driven by production signals rather than a static test suite. Where Phase 9 produces the *map* of what is wrong (the failure taxonomy, the enriched golden dataset, the calibrated judge), Phase 10 is where the agent is actually changed — prompts, tools, retrieval, model selection, guardrails. The two phases run in lockstep: Phase 9 keeps producing supervised signal, Phase 10 keeps consuming it.

> Every change made in production iteration must be validated against the offline eval suite before deploying. Production data reveals new failure modes; your existing test suite must continue to pass.

---

## Checklist

### Turning production signals into improvements

- [ ] **Prioritize from the failure taxonomy, not from the latest incident.** Phase 9 produces a categorized, frequency-counted taxonomy of what is wrong with the agent. That is the prioritization input — work the categories that hurt most in volume and impact, not the ones that happen to be top of mind. Reactive iteration optimizes for memorable failures and ignores the long tail.

- [ ] **Triage production failures by root cause before acting.** Categories in the taxonomy describe *what* is failing; root-cause triage decides *how* to fix it. Not all failures have the same fix:
  - *Prompt problem* — the instructions did not cover this case
  - *Tool problem* — a tool failed or was misused
  - *Retrieval / context problem* — the right information was not surfaced to the model
  - *Capability gap* — the model cannot do what is being asked
  - *Scope problem* — the request is out of scope

- [ ] **Use the enriched golden dataset as the validation surface.** Phase 9 is responsible for growing the dataset with annotated production traces; Phase 10 is responsible for running every candidate change against it. The two roles are distinct: Phase 9 owns dataset quality, Phase 10 owns whether a change passes.

- [ ] **Audit and remove over-restrictive constraints from previous iterations.** Prompts accumulate defensive patches over time. As the agent improves, some patches become counterproductive — causing the agent to withhold correct information or refuse legitimate requests. Review regularly and remove constraints that are no longer warranted.

- [ ] **Record the reason for every prompt change.** When adding a defensive instruction, log why. Without this, future maintainers cannot tell whether a constraint is still needed. This is how you prevent dead instructions from accumulating.

### Systematic improvement process

- [ ] **Run the full offline eval suite before deploying any production change.** The production signal identifies the problem; the offline eval suite validates the fix.

- [ ] **Confirm each change's effect in production, not just in evals.** After deploying a fix, monitor the relevant production metrics to confirm the expected improvement. A change that passes evals but does not move production metrics warrants investigation.

- [ ] **Add guardrails based on real-world vulnerabilities, not just anticipated ones.** The guardrail stack from Phase 7 is a starting point. Production usage reveals attack patterns and edge cases you could not have anticipated.

- [ ] **Revisit model selection periodically.** Models improve rapidly. A task that required a large, expensive model at launch may be handleable by a smaller, faster model months later. Re-evaluate when new models become available.

### Maintain the iteration log

- [ ] **Log every change** with: what changed, why (specific production signal — ideally a failure category from the Phase 9 taxonomy), before/after eval scores, observed production effect. This log is the agent's institutional memory and the primary onboarding artifact for new engineers.

- [ ] **Attribute changes back to flywheel categories.** When a change closes a category in the failure taxonomy, mark it. This is how the flywheel demonstrates its own value over time — and how stalled categories get re-prioritized.

---

## Output

An ongoing **iteration log** recording:
- Change description
- Motivating production signal
- Eval scores before and after
- Observed production effect (confirmed / not confirmed / regression)

> 📋 Template opportunity: Iteration log template.
