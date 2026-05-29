# Skill Ideas

Backlog of skill ideas to build later. Not skills themselves — just specs.

---

## Idea 1 — Manage chapters in the AI agent playbook

**Trigger.** Any chapter-level change: "add a chapter about X", "update chapter N", "remove chapter N", "rename chapter N", "reorder chapters N and M", or any time the playbook structure needs to change.

**Operations.**
- **Add** — new topic, scope, why it matters.
- **Edit** — which chapter, what to change and why.
- **Remove** — which chapter, confirm no cross-references from other chapters or templates.
- **Rename / reorder** — new title or position, cascade index and file numbering.

**What the skill does.**
1. Identify the operation and target chapter(s).
2. For add/edit: research the topic — pull from existing playbook chapters, `glossary.md`, `initial-plan.md`, `raw-notes.md`, and external industry sources.
3. For add: draft in the same shape as existing phases (what the phase is, what you iterate on, how you run it, the loop, escalation, exit criteria).
4. For edit: apply the change surgically; do not rewrite sections not mentioned.
5. For remove: scan all chapters and templates for cross-references before deleting; surface any that would break.
6. For rename/reorder: update the file name, the chapter heading, and every reference across the repo.
7. Run validation checks before saving (see below).
8. Update `ai-agent-playbook/00-index.md` whenever ordering or existence changes.

**Validation checks (must pass).**
- Industry-standard vocabulary only — no invented terms, every key term either common or already in `glossary.md`.
- Tool-agnostic — describes what needs to happen, not how a specific vendor implements it.
- Action-mapped — every section ties to a concrete thing an engineer does, not just description.
- Practically encountered — covers a workflow that actually happens during agent building at Gorgias scale (~3,100 conversations/day).
- Minimum viable — no scope creep; nothing legitimate is forbidden.
- Coaching voice — written to an engineer who will execute, not as theory.
- Scored 5/5/5/5/5 against the playbook rubric (industry-standard, action-mapped, practically encountered, coaching-voiced, minimum-viable).
- No orphaned cross-references after a remove or rename.

**Output.** Modified/created/deleted file(s) under `ai-agent-playbook/` + updated `00-index.md`.

---

## Idea 2 — Manage terms in the glossary

**Trigger.** Any term-level change: "add X to the glossary", "update the definition of X", "remove X", "rename X to Y", or any time a term surfaces, becomes stale, or needs correction.

**Operations.**
- **Add** — candidate term and optionally the context where it came up.
- **Edit** — term name and what to change (definition, cross-references, section placement).
- **Remove** — term name; confirm no cross-references elsewhere in `glossary.md` or the playbook before deleting.
- **Rename** — old and new term name; cascade to all cross-references in the glossary and playbook.

**What the skill does.**
1. Identify the operation and target term(s).
2. For add/edit: look up the term to verify its industry-standard meaning (web search + cross-check against existing playbook usage).
3. For add: draft in the existing glossary format — 2-3 sentence definition, no methodology dump, cross-references using exact wording of existing entries; place alphabetically in the correct section.
4. For edit: apply the change surgically; do not rewrite unrelated parts of the entry.
5. For remove: scan `glossary.md` and all playbook chapters for cross-references before deleting; surface any that would break.
6. For rename: update the term heading and every cross-reference across `glossary.md` and the playbook.
7. Run glossary validation checks (Section 14 of `glossary.md`) before saving.

**Validation checks (must pass — from `glossary.md` Section 14).**
1. Industry-standard vocabulary (Section 12 is the only exception — Gorgias-internal names mapped to industry concepts).
2. Practical relevance scored 4 or 5 out of 5.
3. No duplicates or significant overlap with existing entries.
4. 2-3 sentence definition, not a methodology dump.
5. Technically accurate and current.
6. Understandable by an engineer with vague AI knowledge.
7. Cross-references resolve to existing entries using exact wording.
8. No orphaned cross-references after a remove or rename.

**Output.** Updated `glossary.md` with the change applied in the right section; no broken cross-references.

---

## Idea 3 — Document a pain point as a validation scenario

**Trigger.** "Log this pain point", "add this to validation scenarios", or whenever I paste a Slack message, Notion doc, GitHub PR, or DM describing something that's painful in the current AI agent workflow.

**Input.** The raw source — Slack permalink, Notion URL, PR link, pasted DM text, screenshot.

**What the skill does.**
1. Read and digest the source.
2. Produce a concise recap of the pain point (what's broken, who hits it, when).
3. Map it to the relevant playbook phase (Phase 1-10 under `ai-agent-playbook/`).
4. Sketch the expected solution by referencing the specific playbook section that should resolve it — so we know exactly what the new stack must prove out.
5. Append as the next `V-###` entry in `validation-scenarios.md`, following the existing format (category, recap, expected solution, source link).

**Output format (matches `validation-scenarios.md`).**
```
## V-### — <one-line title>

**Category:** Phase N — <phase name>

**Recap.** <2-4 sentences: what's painful, who hits it, current workaround>

**Expected solution.** Playbook Phase N: <concrete capability the new stack must deliver, referencing the playbook section>.

**Source.** [<platform> — <author>, <date>](<url>)
```

**Why it matters.** This builds a concrete acceptance-test backlog. When the new LLM stack is ready, every V-### must be reproducible end-to-end before we declare the stack adopted.

---

## Idea 4 — Playbook-driven AI agent build (multi-layer architecture)

**Status.** Architecture sharpened — not a single skill, a stack of skills + Claude Code `/goal` + scheduled tasks. Ready for deeper spec work, not yet ready to build.

**Trigger.** "Build me an AI agent for X", "scaffold a new agent following the playbook", or starting any new agent project that should follow the 10-phase lifecycle.

**Input.** A high-level agent idea (purpose, scope, target merchants, success criteria — rough is fine).

### Why not one mega-skill

Three structural reasons a single end-to-end skill fails:
- **Context.** By Phase 6 the build would carry framing doc, design doc, prompts, tool definitions, golden dataset, and eval results in context. Quality degrades and most of it isn't relevant to every later phase.
- **Time scales.** Phases 1-5 are hours of design. Phase 6 is hours of compute. Phase 7 is human approval + progressive rollout. Phases 8-10 unfold over weeks of production. A skill runs in a turn.
- **Human gates.** The playbook exists to teach an engineer to build an agent. If the skill walks Phase 1→10 without them reading what it produced, the playbook's purpose is defeated. Acceleration, not replacement.

### Architecture — four layers

**Layer 1 — Per-phase skills.** One skill per phase that:
- Reads only the prior-phase artifacts it strictly needs (Phase 4 needs Phase 2 topology, not Phase 1 framing).
- Produces the current phase's artifact in a fixed template.
- Runs the phase's validation checks before declaring done.
- Returns a one-paragraph summary + path to the artifact, never the full artifact in chat.

State lives in files (or Notion / Linear), not in conversation.

**Layer 2 — `/goal` wrapping each phase skill.** Claude Code's `/goal` command loops draft → check → revise → check until a completion condition is met. Fits per-phase skills cleanly: condition is "artifact exists at `<path>` AND passes the validation checks." The engineer doesn't babysit revisions — `/goal` runs the inner loop.

Works for Phases 1, 2, 3, 4, 5 and the setup portion of 8, 9. Does **not** work for 6, 7, 10 (synchronous, blocks on long-running compute or human gates).

For this to work the completion condition must be machine-checkable: a script that runs the validation (template completeness, cross-reference resolution, no TODOs, smoke test pass) or a rubric prompt that scores 5/5 across the criteria. Vague conditions let the evaluator declare victory early.

**Layer 3 — Scheduled tasks for long-running ops.** Skills configure systems but do not operate them.
- Phase 6 — kicks off the eval run as a background job, exits; a scheduled task pings when results land.
- Phase 8 — wires up dashboards, alerts, annotation queue; doesn't watch them.
- Phase 9 — annotation review cadence reminder + a separate "annotation review" skill that processes the queue.

**Layer 4 — Orchestrator skill.** Reads an `agent-build-state.md` (or Notion page / Linear project) that tracks: current phase, artifact paths, validation status, blockers. Tells you what's next, blocks the next phase until the prior phase passes validation and the engineer signs off, surfaces cross-phase consistency issues (Phase 4 tools that don't appear in Phase 2 topology, Phase 5 eval cases that don't cover Phase 1 failure modes).

### Validation mechanisms per phase — five layers

1. **Template completeness** — every required field filled, no TODOs.
2. **Phase-specific validation checks** — like the chapter rubric in Idea 1, the glossary checks in Idea 2.
3. **Cross-phase consistency** — Phase N references resolve to entities defined in Phase <N.
4. **Smoke test** where applicable — Phase 3 prompts run against 1-3 cases, Phase 4 tool stubs return valid responses, before declaring the phase done.
5. **Human signoff** as the final gate — don't engineer around it.

LLM-as-judge scoring on prompt and design-doc quality is a useful sixth, used as tiebreaker, not gate.

### Phase fit table

| Phase | Per-phase skill | `/goal` wrap | Scheduled task |
|---|---|---|---|
| 1 — Problem framing | ✓ | ✓ | — |
| 2 — Agent design | ✓ | ✓ | — |
| 3 — Prompt engineering | ✓ | ✓ | — |
| 4 — Tool integration design | ✓ | ✓ | — |
| 5 — Evaluation design | ✓ | ✓ | — |
| 6 — Offline iteration | ✓ | ✗ (async) | ✓ (eval runs) |
| 7 — Deployment | ✓ (checklist) | ✗ (human gate) | — |
| 8 — Production monitoring | ✓ (setup) | ✓ for setup | ✓ (digests) |
| 9 — Data flywheel | ✓ (setup) | ✓ for setup | ✓ (review cadence) |
| 10 — Production iteration | ✓ | ✗ (weeks) | ✓ (loops back to 6) |

### Open questions

- What's the artifact format per phase — markdown in repo, Notion pages, Linear issues, mix? Affects how the orchestrator reads state.
- Where does the per-phase validation logic live — inside the skill, as a separate validation script, or both?
- How does the orchestrator interact with Idea 1 (chapter authoring) and Idea 2 (glossary terms) when new vocab/phases surface mid-build?
- Phase 6 and Phase 8 depend on the new LLM stack existing. Until SPADE picks the platform, those phases can't be fully built — but the skill scaffolding can.
- How do we keep these skills in sync when the playbook itself evolves? Auto-regenerate from the playbook source?

### Next step

Prototype Phase 1 and Phase 2 skills with `/goal` wrappers and machine-checkable completion conditions. Use the experience to refine the architecture before building Phases 3-5. Phases 6+ wait for the SPADE stack decision.
