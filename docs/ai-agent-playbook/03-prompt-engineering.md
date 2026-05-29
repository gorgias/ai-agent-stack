# Phase 3 — Prompt Engineering

> **Output:** Versioned system prompt (and sub-task prompt templates if needed)
> **Gate:** Prompt produces qualitatively correct outputs on manual testing before building tools.

---

## What this phase is

Write, structure, and iterate on the instructions that tell the agent how to behave. For agents, this is more consequential than for single-turn use cases. Ambiguity that is recoverable in a one-shot exchange can cascade into a series of wrong actions in an agent loop.

> Mental model: write instructions as if onboarding a highly capable new colleague who knows nothing about your norms, conventions, or policies. They are smart and will reason from what you tell them — but they cannot infer what you did not say.

Prompt engineering is iterative, not one-shot. The discipline is to do it systematically.

---

## Checklist

### Drafting

- [ ] **Give the agent a clear role.** The opening of a system prompt should define what the agent is and what it is for. Even one sentence makes a measurable difference in consistency.

- [ ] **Structure the prompt into distinct sections.** A prompt mixing role, policy, tone, and data in unstructured prose is unparseable. Separate concerns clearly. Common sections: role/persona · guidelines · policies · tone · tool usage · output format · edge case handling.

- [ ] **State instructions positively.** Tell the agent what to do, not what to avoid. *"Respond in plain prose paragraphs"* is more reliable than *"do not use bullet points."*

- [ ] **Include the *why* behind key instructions.** Explaining the reason for a constraint helps the model generalize to edge cases you did not explicitly anticipate.

- [ ] **State both sides of every trade-off.** If an instruction mentions a cost (e.g. "escalating is expensive"), also state the cost of *not* taking that action. Capable models optimize hard against stated objectives — one-sided framing produces one-sided behavior.

- [ ] **Define an output contract.** State the expected output format explicitly. For structured outputs, define the schema. Make the output boundary unambiguous.

- [ ] **Capture edge cases and handoff conditions.** Anticipate common decision points: incomplete input, ambiguous requests, out-of-scope queries. Define what the agent should do in each case.

### Examples

- [ ] **Add 3–5 diverse, representative examples.** Examples are one of the most reliable ways to steer output format, tone, and reasoning style. Make them relevant, varied, and cover edge cases. Wrap them in a clearly labeled section.

- [ ] **For reasoning-heavy tasks, include worked reasoning traces in examples.** Showing the model *how* to think through a problem produces more reliable reasoning than showing only the final answer.

### Iteration hygiene

- [ ] **Run evals before touching a prompt.** Never edit blind. Establish a baseline score first — see Phase 5 for how.

- [ ] **Apply general hygiene before targeting specific failures.** Clean structure, remove outdated patches, separate concerns. This often resolves several failures at once.

- [ ] **Fix one failure mode at a time.** Isolate, change, measure. Fixing multiple things simultaneously makes attribution impossible.

- [ ] **Version every prompt change and record the reason.** Treat prompts like code. Every change needs a commit message. Without this, you accumulate dead constraints and cannot explain why they exist.

- [ ] **Don't use instructions to compensate for missing capabilities.** If a task requires precise computation or deterministic lookup, build a tool (Phase 4). Instructions shape *how* the model thinks; tools provide *what* it can reliably execute.

---

## Output

A **versioned system prompt** stored in version control, with a changelog recording the reason for each change.

> 📋 Template opportunity: System prompt structure template (role · guidelines · policies · tools · output format · edge cases).
