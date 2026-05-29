# Phase 1 — Problem Framing

> **Output:** Problem Framing Document
> **Gate:** Success criteria are defined and measurable before moving on.

---

## What this phase is

Establish whether building an agent is actually the right answer. Agents are suited to workflows that require multi-step reasoning, judgment over ambiguous inputs, or dynamic decision-making. They are not the right choice for simple, deterministic workflows where a script or a single LLM call is sufficient.

Also: define what success looks like *before* you build. Without it, you will not know when to ship, and you cannot measure whether iteration is making things better or worse.

> The most expensive mistake in agent development is building the wrong thing well.

---

## Checklist

- [ ] **Describe the workflow in plain language.** Write out the sequence of steps a human currently performs. If you cannot describe it clearly, you are not ready to build it.

- [ ] **Confirm this is an agent problem, not a script problem.** An agent is warranted when at least one is true:
  - The workflow involves complex judgment or context-sensitive decisions that rules cannot handle.
  - The existing rules-based system is so brittle it is cheaper to replace than maintain.
  - The workflow requires interpreting unstructured data (natural language, documents).
  - If none apply — build a deterministic solution instead.

- [ ] **Define success in measurable terms.** What does a good outcome look like? What does a bad one look like? These definitions become your evaluation criteria in Phase 5.
  > *Example: "resolves the customer's issue without escalation in under 2 minutes."*

- [ ] **Rank your failure modes.** What are the worst things the agent could do? (Wrong information delivered confidently, irreversible actions taken without confirmation, data leaked.) Rank them. This shapes guardrail design in Phase 7.

- [ ] **Establish hard constraints.** Non-negotiables before design begins: maximum latency, cost per run, accuracy floor, data the agent must never access, actions it must never take autonomously.

- [ ] **Define the scope boundary.** What is explicitly in scope? What must it hand off to a human or another system? Unclear scope is one of the most common causes of production failures.

---

## Output

A one-page **Problem Framing Document** answering:
1. What is the workflow?
2. Why can't a deterministic system handle it?
3. What does success look like (measurable)?
4. What are the ranked failure modes?
5. What are the hard constraints?
6. What is the scope boundary?

> 📋 Template opportunity: Problem Framing Document.
