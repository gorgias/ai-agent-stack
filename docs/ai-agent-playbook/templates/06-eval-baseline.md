# Eval Baseline Record

> Phase 6 output. The snapshot of the agent that passed offline evals and is cleared to deploy. One per release candidate.

---

**Agent name:**
**Baseline ID / tag:**
**Date:**
**Author:**

---

## Versions in this baseline

Every piece that defines runtime behavior. If something is missing here, you cannot roll back to it.

- **Prompt version:**
- **Model & configuration:** (model, temperature, reasoning effort)
- **Tool versions:** (tool name → version)
- **Topology:** (single / manager / decentralized; sub-agents and how they're wired)
- **Memory & retrieval config:**
- **Guardrail config:**

## Eval scores

Full golden dataset, full agent run.

| Subset | Score | Pass threshold | Pass? |
|---|---|---|---|
| control | | | |
| edge | | | |
| boundary | | | |
| adversarial | | | |

## Known failure modes

Cases the agent still handles poorly. Carry these into Phase 8 monitoring and Phase 10 backlog.

| Failure mode | Severity | Notes |
|---|---|---|
| | | |
