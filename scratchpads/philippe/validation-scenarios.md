# Validation Scenarios

Concrete painpoints from real Gorgias work, used as acceptance tests for the new stack. Each scenario must be reproducible end-to-end on the chosen tooling before we declare it solved.

Each entry: category (playbook phase), recap, expected solution (referenced in playbook), source.

---

## V-001 — Solutions team can't self-serve trace lookup by ticket_id

**Category:** Phase 8 — Production Monitoring

**Recap.** When a merchant flags a bad agent run, Solutions needs to pull the full prompt blueprint + JSON by `ticket_id`. Today the only paths are a PromptLayer API key (no scoped roles) or a CSV export from BQ.

**Expected solution.** Playbook Phase 8: queryable trace store with traces tagged by business identifier (`ticket_id`), single trace view showing prompt blueprint + JSON, and scoped read-only roles for non-engineering personas.

**Source.** [Slack — Anton Savytski, 2026-05-28](https://gorgias.slack.com/archives/C07LUTB7ZD0/p1779981833287389)

---

## V-002 — Non-technical users can't evaluate guidance changes offline before they go live

**Category:** Phase 2 — Prompt Engineering / Phase 4 — Offline Experimentation

**Recap.** Non-technical users (CX and Product Specialists) own the guidance layer — the instructions that shape how the agent uses a tool's output. They iterate on these independently from engineers. When they edit a guidance, there is no way to measure offline whether the new version performs better or worse than the previous one on a representative set of real conversations. The current process is: edit the guidance, ship it live, then observe outcomes manually — a 45–60 minute guesswork cycle per change with no controlled comparison. (Sandra Alvarenga, CX roundtable recap by Nicole Simmen, 2026-06-02/09.)

**Expected solution.** The platform must expose experiment primitives — versioned prompts, dataset runs, and side-by-side scoring — sufficient for engineers to build a lightweight self-service interface on top of them. Non-technical users should be able to trigger "run old guidance vs. new guidance against this dataset and show me the diff" without engineering involvement at runtime. If the platform also provides a low-friction native UX for this use case, that is a positive signal but not a hard requirement.

**Source.** [Slack — Sandra Alvarenga, 2026-06-09](https://gorgias.slack.com/archives/C07LUTB7ZD0); [Slack — Nicole Simmen roundtable recap, 2026-06-02](https://gorgias.slack.com/archives/C07LUTB7ZD0)

---

## V-003 — Agent has no memory of past conversations (episodic memory / "dreams")

**Category:** Phase 1 — Agent Design / Phase 2 — Prompt Engineering

**Recap.** Agents today are stateless across conversations — each run starts with no recollection of prior interactions with the same merchant or ticket thread. The idea: implement a "dreams" pattern where the agent, at the end of a run (or on a schedule), distills key facts from recent conversations into a compact summary that persists as part of its context on the next run. This is episodic memory via prompt injection.

**Phase 1 — Prompt-injected summary (immediate).** Write a prompt that reads a stored per-merchant summary blob and injects it into the system prompt, then updates that blob at conversation end. No retrieval, just a structured scratchpad.

**Phase 2 — RAG design (future).** As the memory corpus grows, move from a flat summary to a vector-indexed store. Relevant past episodes are retrieved at inference time rather than injected wholesale.

**Expected solution.** The observability platform must support structured metadata on traces (merchant/ticket identifiers) so the memory layer can query past runs. The router layer must pass through arbitrary context without truncating. Prompt versioning must cover the memory-injection template alongside the main system prompt.

**Source.** Philippe Diep, 2026-05-29
