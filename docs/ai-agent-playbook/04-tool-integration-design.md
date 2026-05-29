# Phase 4 — Tool & Integration Design

> **Output:** Tool catalog with risk ratings
> **Gate:** All tools defined, described, and tested independently before wiring into the agent.

---

## What this phase is

Tools extend the agent from generating text to taking real-world actions: querying databases, calling APIs, writing records, sending messages, invoking sub-agents. Without well-designed tools, an agent can only reason — it cannot act.

Tool design is as important as prompt design. The agent's ability to select and use the right tool depends entirely on how clearly tools are defined. A vague description is indistinguishable from a precise one in code — but to the model, the description is the primary signal for when and how to use it.

> Give the model a tool instead of an instruction for anything that requires reliable execution. Instructions shape behavior; tools provide capability.

---

## Checklist

### What tools to build

- [ ] **List all data the agent needs to retrieve.** Data tools: querying records, searching knowledge bases, reading documents, looking up structured data. Every external data source the agent will read from needs a tool.

- [ ] **List all actions the agent needs to take.** Action tools: creating records, sending messages, updating databases, triggering processes. Every external system the agent will write to or modify needs a tool.

- [ ] **Identify orchestration tools.** In multi-agent systems, sub-agents exposed as callable tools are orchestration mechanisms. Include them in the catalog with the same rigor.

- [ ] **Keep the tool surface minimal.** Only include tools the agent actually needs. Every additional tool increases the chance of incorrect selection. If tool selection is failing, look for overlapping or ambiguous tools before adding more.

### How to define each tool

- [ ] **Write a precise, detailed description for every tool.** The description tells the model what the tool does, when to use it, when *not* to use it, what arguments it requires, and what it returns. Vague descriptions produce inconsistent tool use.

- [ ] **Define clear input/output schemas.** Specify parameter types, required vs. optional, and the structure of the return value. The model uses this schema to construct calls.

- [ ] **Make tools composable and single-purpose.** One tool, one job. Avoid tools that bundle multiple concerns. Well-scoped tools can be reused across agents and are easier to test.

- [ ] **Test every tool independently before connecting it to the agent.** Verify correct behavior for happy paths, edge cases, and error cases. Tool failures are a common source of agent failures — catch them in isolation.

### Risk management

- [ ] **Assign a risk rating to every tool (low / medium / high).** Rate based on: read-only vs. write access, reversibility, whether it affects shared systems, financial impact, security implications.

  | Rating | Examples |
  |---|---|
  | Low | Read-only queries, lookups, searches |
  | Medium | Creating records, sending notifications |
  | High | Irreversible deletions, financial transactions, sending external communications |

- [ ] **Enforce high-risk tool constraints at the execution layer, not just the prompt.** The prompt is not a sufficient guardrail for irreversible actions. Build confirmation requirements or authorization checks into the execution logic itself.

- [ ] **Add input validation to tool implementations.** Where possible, fail gracefully on bad inputs rather than propagating errors silently downstream.

---

## Output

A **tool catalog** for the agent, with for each tool:
- Name and one-line description
- Input/output schema
- When to use / when not to use
- Risk rating (low / medium / high)
- Known edge cases and error behavior

> 📋 Template opportunity: Tool definition template.
