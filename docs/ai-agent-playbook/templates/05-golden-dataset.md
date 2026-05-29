# Golden Dataset

> Phase 5 output. Living document. Owned by a named domain expert. Versioned in source control.

---

**Owner (domain expert):**
**Last reviewed:**

---

## Cases

One row per test case. Use the same dataset for all subsets — never split them into separate files.

| ID | Subset | Input | Expected output or criteria | Added by | Reason added | Date |
|---|---|---|---|---|---|---|
| | control | | | | | |
| | edge | | | | | |
| | boundary | | | | | |
| | adversarial | | | | | |

**Subset meaning:**
- `control` — unambiguous cases; baseline behavior and regression tests.
- `edge` — hard, unusual, or ambiguous inputs.
- `boundary` — escalation, refusal, handoff behavior.
- `adversarial` — prompt injection, jailbreaks, PII leakage, policy violations under pressure.

---

## Scoring rubric (for LLM-as-judge)

For subjective subsets, define how the judge scores. Keep it short and unambiguous — the rubric is calibrated against the domain expert's ratings.

---

## Evaluation method per subset

| Subset | Method | Reviewer |
|---|---|---|
| control | deterministic or LLM judge | domain expert |
| edge | LLM judge | domain expert |
| boundary | LLM judge | domain expert |
| adversarial | safety judge or human review | trust & safety |
