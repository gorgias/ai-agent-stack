# Iteration Log

> Phase 10 output. Ongoing for the life of the agent. The institutional memory of how this agent got to be the way it is.

---

**Agent name:**
**Maintainer:**

---

## Log

One row per change. A change is anything that affects runtime behavior: prompt, tool, model, topology, memory, retrieval, guardrail.

| Date | What changed | Motivating signal (failure category or production metric) | Eval before | Eval after | Production effect (confirmed / not confirmed / regression) | Author |
|---|---|---|---|---|---|---|
| | | | | | | |

---

## How to use this

- Every change goes in — including reverts.
- *Motivating signal* should reference a category from the Phase 9 failure taxonomy whenever possible. This is how the flywheel demonstrates value.
- *Eval before / after* is the full golden dataset against the full agent. Never use a prompt-only score here.
- *Production effect* is filled in **after** the change has been live long enough to observe. A change that passed evals but did not move the production metric is worth a note.
