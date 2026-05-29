# Problem Framing Document

> One-pager. Phase 1 output. Sign off before designing anything.

---

**Agent name:**
**Author:**
**Date:**

---

## 1. Workflow

What is the task, in plain language? Describe the sequence of steps a human currently performs.

## 2. Why an agent

Why can't a deterministic system (rules, scripts) handle this? Name the specific reason: judgment required, brittle rules, unstructured inputs, etc.

## 3. Success criteria

What does a good outcome look like, measured? One sentence, observable.

## 4. Ranked failure modes

The worst things this agent could do, worst first. This list drives guardrail design later.

1.
2.
3.

## 5. Hard constraints

Non-negotiables: max latency, max cost per run, accuracy floor, data the agent must never access, actions it must never take autonomously.

## 6. Scope boundary

In scope:
Out of scope (must hand off):
