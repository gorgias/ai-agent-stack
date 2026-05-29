# AI Agent Development Playbook

A step-by-step guide for building AI agents — from the first idea to a monitored, iterating production system.

Each phase produces a concrete artifact. You do not move to the next phase until that artifact exists.

---

## Phases

| #                                       | Phase                       | Output artifact                                                                              |
| --------------------------------------- | --------------------------- | -------------------------------------------------------------------------------------------- |
| [01](./01-problem-framing.md)           | Problem Framing             | Problem Framing Document                                                                     |
| [02](./02-agent-design.md)              | Agent Design                | Agent Design Document                                                                        |
| [03](./03-prompt-engineering.md)        | Prompt Engineering          | Versioned system prompt                                                                      |
| [04](./04-tool-integration-design.md)   | Tool & Integration Design   | Tool catalog with risk ratings                                                               |
| [05](./05-evaluation-design.md)         | Evaluation Design           | Evaluation suite                                                                             |
| [06](./06-iteration-offline-testing.md) | Iteration & Offline Testing | Passing eval baseline                                                                        |
| [07](./07-deployment-release.md)        | Deployment & Release        | Live agent with guardrails                                                                   |
| [08](./08-production-monitoring.md)     | Production Monitoring       | Monitoring dashboard                                                                         |
| [09](./09-data-flywheel.md)             | Data Flywheel               | Running flywheel with failure taxonomy, annotation pipeline, and instrumented health metrics |
| [10](./10-production-iteration.md)      | Production Iteration        | Iteration log (ongoing)                                                                      |

---

## Ground rules

**Vendor-agnostic.** This playbook describes *what* to do, not *how* your specific tool does it. "Version your prompts" is in scope. "Create a prompt version in Langfuse" is not.

**Strictly ordered.** Each phase depends on the previous. Do not write a system prompt before defining the task. Do not instrument monitoring before you have an eval baseline.

**One artifact per phase.** Every phase ends with a concrete output. That output is what gates entry to the next phase.

---

## Validation criteria for terminology

This section governs **what terms are allowed to appear in any chapter** of the playbook. Apply it whenever the playbook is updated — when adding new terminology, when reviewing a PR, or when auditing an existing chapter. A term that does not pass these criteria must be reworded, demoted to a footnote, or removed.

The audience is an engineer with **vague prior knowledge of AI agents** — they may have built parts of one, but they are not fluent in eval, observability, or qualitative-research vocabulary. The playbook must read like a senior coworker coaching them, not a whitepaper or an audit checklist.

A term's presence in a chapter is relevant only if **all five criteria are met**:

1. **Industry-standard.** The term is used by mature, production-proven sources (OpenAI, Anthropic, Google, established eval/observability vendors, peer-reviewed literature) in the same sense the playbook uses it. No invented vocabulary. No Gorgias-specific naming. If the term is industry-standard but has multiple competing definitions, the playbook must pick one and note the choice.

2. **Action-mapped.** The term maps to a concrete action, decision, or artifact an engineer will produce during agent development. Terms that exist only to describe a concept — without changing what the reader does — are removed or moved to the glossary.

3. **Practically encountered.** A working engineer building agent #1 at Gorgias's scale (~3,100 conversations/day) will plausibly encounter this term within the first 6 months of work. Terms that only apply to teams of 8+ reviewers, to specialized research workflows, or to mature flywheel operations at organizations 10× our scale belong in an *advanced practices* footnote — not in the main checklist.

4. **Coaching-voiced.** The term is introduced the way a senior coworker would introduce it: with the why, with a concrete example or aphorism, and with the trap it helps the reader avoid. Bare definitions are not enough. If a term cannot be introduced in this voice, it is likely too theoretical for the main body.

5. **Minimum-viable filter.** If a first-time agent builder could ship a safe, monitored agent without ever using this term, the term is *advanced* and must be marked as such (footnote, appendix, or explicit "advanced" tag). The main checklist contains only what is required for the minimum viable path through the phase.

### Examples — what passes and what does not

| Term | Verdict | Reason |
|---|---|---|
| *Golden dataset* | Passes | Industry-standard; maps to a concrete artifact; encountered immediately; coaching-voiced throughout; minimum-viable. |
| *Guardrail* | Passes | Standard; action-mapped (build one); encountered at deployment; introduced with the trap (irreversible actions). |
| *Trace* | Passes | Standard observability term; action-mapped (instrument it); encountered from day one of production. |
| *Open coding / axial coding* | Fails (criteria 3, 4, 5) | Qualitative-research vocabulary. Engineers will not recognize it. The underlying activity (read failing traces, group them, count them) can be expressed without the academic term. Demote to footnote. |
| *Cohen's kappa / Krippendorff's alpha* | Fails (criteria 3, 5) | Real metrics, but only relevant for teams running multi-annotator workflows. Move to an *if you have multiple annotators* sidebar. |
| *Principal annotator* | Fails (criterion 3) | Useful for 5+ reviewer teams. Overkill for a 1–2 person setup, which is the minimum-viable starting point. Mark as advanced. |
| *Generate-evaluate-repair loop* | Conditional pass | Industry-standard and action-mapped, but specialized. Must be explicitly tagged as an advanced pattern, not presented as default escalation. |

### Process for proposing a new term

When a chapter update introduces a new term, the author must:

- State which of the five criteria the term satisfies, with a one-line justification per criterion.
- Show how the term is introduced in coaching voice (the sentence that brings it in).
- Confirm whether the term belongs in the main checklist or in an advanced/footnote section.

Any term that cannot satisfy all five criteria is rejected from the main body. The glossary may still record it for reference — but the playbook itself stays disciplined.
