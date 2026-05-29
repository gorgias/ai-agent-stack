# Monitoring Dashboard Spec

> Phase 8 output. Live from the first production request — not added later. Every metric below should map to an action you would take if it degrades.

---

**Agent name:**
**Owner:**
**Dashboard link:**

---

## Quality metrics

- [ ] Automated eval score on production sample (same LLM-as-judge as offline)
- [ ] Task completion rate
- [ ] Escalation rate
- [ ] Guardrail trigger rate (per guardrail)
- [ ] Tool selection accuracy

## Operational metrics

- [ ] Latency at p50, p95, p99
- [ ] Cost per conversation
- [ ] Error rate, broken down by category (model, tool, guardrail, infra)

## Tracing

- [ ] Full execution trace per run (inputs, reasoning, tool calls, output)
- [ ] Step-level latency and cost
- [ ] Eval scores attached to traces at runtime
- [ ] Traces stored in a **queryable** store (required so Phase 9 routing can select on signals)

## Flywheel-ready signals attached to every trace

(So the Phase 9 selection layer has something to route on.)

- [ ] Automated eval score
- [ ] Guardrail trip events
- [ ] Escalation flag
- [ ] Explicit user feedback (thumbs, ratings)
- [ ] Implicit feedback (retries, follow-ups, abandonment)
- [ ] Latency, cost, tool call count

## Alerting

One row per alert.

| Metric | Threshold | Action (investigate / page / rollback) |
|---|---|---|
| | | |
