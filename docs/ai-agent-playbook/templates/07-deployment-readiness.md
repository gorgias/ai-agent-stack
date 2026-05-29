# Deployment Readiness Checklist

> Phase 7 output. All items must be checked before the agent sees its first real user.

---

**Agent name:**
**Baseline ID being shipped:**
**Date:**
**Owner:**

---

## Guardrails active

- [ ] Targeted guardrails built for the ranked failure modes from Phase 1
- [ ] Multiple guardrail layers wired up (relevance, safety, PII, content, tool safeguards, output validation — pick the set that matches your risk profile)
- [ ] High-risk tool constraints enforced at the **execution layer**, not in the prompt

## Human-in-the-loop

- [ ] Escalation conditions documented
- [ ] Escalation paths tested end-to-end
- [ ] Human override / correction is captured back to the trace store

## Instrumentation

- [ ] Tracing live — every run produces a structured trace
- [ ] Quality and operational metrics flowing to the dashboard
- [ ] Alerting thresholds configured

## Release plan

- **Initial rollout scope:** (internal users / beta cohort / constrained query type)
- **Go criteria:** what must be true to proceed.
- **No-go / rollback criteria:** the threshold (error rate, escalation rate, specific failure class) that triggers rollback.
- **Rollback procedure:** documented and tested.
