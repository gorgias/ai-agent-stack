# Agent Design Document

> Phase 2 output. Finished before any prompt is written.

---

**Agent name:**
**Author:**
**Date:**

---

## 1. Orchestration pattern

- [ ] Single-agent (default)
- [ ] Manager (central agent + sub-agents)
- [ ] Decentralized (handoff between agents)

Why this pattern (one sentence). If not single-agent, what evidence forced the choice.

## 2. Components

- **Model(s):** which model for which step.
- **Tools (preliminary):** short list — full catalog comes in Phase 4.
- **Instructions summary:** one-paragraph description of how the agent should behave.

## 3. Memory & state

- **Short-term:** how much conversation history is kept verbatim vs. summarized.
- **Long-term:** needed? If yes, what is stored and when it is retrieved.
- **State across steps:** what must persist, and where it lives.

## 4. Run loop & exit conditions

What ends a run? List every exit condition: final answer produced, specific tool called, error threshold hit, max step count reached.

## 5. Escalation & handoff

When does the agent stop and hand off to a human or another system?
