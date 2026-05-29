# Tool Catalog

> Phase 4 output. One entry per tool. Keep the surface minimal — every extra tool risks incorrect selection.

---

## Catalog summary

| Tool | Purpose | Blast radius |
|---|---|---|
| | | low / medium / high |

> *Blast radius* = how much damage is done if this tool fires when it shouldn't. Combines reversibility, scope of effect (private vs. shared systems), financial impact, and security implications. It is **not** the probability of the model picking the tool incorrectly.

---

## Tool entry template

Copy and fill once per tool.

### `tool_name`

**One-line description.** What it does, in plain language.

**When to use.** Concrete situations where this tool is the right choice.

**When *not* to use.** Cases that look similar but call for a different tool — or no tool at all.

**Input schema.** Parameters, types, required vs. optional.

**Output schema.** Structure of what is returned.

**Blast radius.** `low` (read-only / lookup, nothing to undo) · `medium` (creates records, sends internal notifications — recoverable) · `high` (irreversible, financial, external communications, security-sensitive).

**High-blast-radius constraints (if applicable).** What the execution layer enforces — confirmation, authorization, allowlists. The prompt is not a guardrail.

**Known edge cases & error behavior.** What can go wrong and how the tool fails.
