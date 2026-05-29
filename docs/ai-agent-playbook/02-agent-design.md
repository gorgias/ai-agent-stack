# Phase 2 — Agent Design

> **Output:** Agent Design Document
> **Gate:** Architecture, memory model, and exit conditions are decided before writing a single prompt.

---

## What this phase is

Design the agent's architecture before touching any implementation. Every agent is built from three components:

- **Model** — the LLM that drives reasoning and decisions
- **Tools** — functions or APIs the agent calls to act or retrieve information
- **Instructions** — explicit guidelines defining how the agent behaves

The architecture question is how these are composed and orchestrated.

> Start with the simplest architecture that could work. Add complexity only when evaluations tell you it is necessary.

---

## Checklist

### Orchestration pattern

- [ ] **Default to a single-agent architecture.** One model, the right tools, clear instructions, running in a loop until an exit condition is met. This handles the majority of use cases. Do not design a multi-agent system before you have evidence it is needed.

- [ ] **Define the run loop and exit conditions.** Every agent needs an explicit stop condition: a final answer is produced, a specific tool is called, an error threshold is exceeded, or a max step count is reached. Without this, the agent runs indefinitely.

- [ ] **If multi-agent is needed, choose a pattern:**

  | Pattern | Description | Use when |
  |---|---|---|
  | **Manager** | A central agent orchestrates specialized sub-agents via tool calls. Maintains control and synthesizes results. | You need a single point of control and unified output. |
  | **Decentralized (handoff)** | Agents pass execution to one another. Each fully takes over when control is transferred. | Tasks are cleanly separable and no synthesis is needed. |

  **Signal to split:** instructions contain so many conditional branches that a single prompt is failing, or tools are so numerous/overlapping that tool selection degrades.

### Memory and state

- [ ] **Design your in-context (short-term) memory strategy.** Conversation history is ephemeral. Decide how much history to keep verbatim and whether older turns should be summarized.

- [ ] **Decide if you need long-term memory.** Long-term memory persists across sessions. It is warranted when the agent must remember user preferences, past interactions, or accumulated knowledge. If you build it, decide: what to store, when to retrieve (default: on every query), and how to expire stale data.

- [ ] **List all state that must persist across steps.** In a multi-step agent, state degrades if not managed explicitly. For each item, decide how it is stored (in-context, structured file, database).

### Model selection

- [ ] **Start with the most capable model to establish a baseline.** Once you have evaluation results, swap in smaller, cheaper models where they pass. Do not prematurely limit performance.

- [ ] **Match model capability to step complexity.** Simple retrieval or classification steps can often use a lighter model. Reserve the most capable model for steps requiring genuine reasoning.

---

## Output

An **Agent Design Document** covering:
1. Chosen orchestration pattern (single / manager / decentralized)
2. Component list: model(s), tools (preliminary), instructions summary
3. Memory and state management approach
4. Run loop and exit conditions
5. Escalation and handoff points

> 📋 Template opportunity: Agent Design Document.
