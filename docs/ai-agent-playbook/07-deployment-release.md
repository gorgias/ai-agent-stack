# Phase 7 — Deployment & Release

> **Output:** Live agent with active guardrails, instrumentation live, human escalation paths tested
> **Gate:** All of the above confirmed before opening to users.

---

## What this phase is

The work between "passing evals offline" and "running in production." The core concern is safety: when the agent encounters real users and real edge cases, it should fail gracefully rather than catastrophically.

> The right deployment posture for a new agent is not "ship it and see." It is "ship cautiously with full observability and human oversight, then progressively relax constraints as confidence grows."

Guardrails are not an afterthought. They are a primary design concern and a layered system — no single guardrail is sufficient, but multiple specialized ones together create a resilient agent.

---

## Checklist

### Guardrail design

- [ ] **Start from your ranked failure modes (Phase 1).** Build targeted guardrails for the failures you identified as most damaging. Add new ones as you discover vulnerabilities in production.

- [ ] **Layer multiple types of guardrails.** Use a combination appropriate to your risk profile:

  | Guardrail type | What it does |
  |---|---|
  | Relevance filter | Flags or rejects inputs outside the agent's defined scope |
  | Safety classifier | Detects prompt injection attempts and jailbreaks |
  | PII filter | Prevents the agent from exposing personal data in outputs |
  | Content moderation | Flags harmful, harassing, or policy-violating inputs |
  | Tool safeguards | Enforces risk ratings — requires confirmation before high-risk tool calls |
  | Rules-based protections | Deterministic checks: blocklists, input length limits, regex filters |
  | Output validation | Verifies responses meet format, policy, and quality requirements |

- [ ] **Enforce high-risk tool constraints at the execution layer.** Do not rely on the prompt to prevent irreversible actions. Build confirmation or authorization requirements into the execution logic itself.

### Human-in-the-loop

- [ ] **Define explicit escalation conditions.** When does the agent transfer control to a human? Common triggers: repeated failure to understand intent, a requested action exceeds authorization, the request is explicitly out of scope. Document and test these paths before launch.

- [ ] **Start with more human oversight than you think you need.** Early in deployment, over-escalating is safer than under-escalating. Relax thresholds progressively as confidence in the agent's behavior grows.

- [ ] **Capture human corrections.** When a human overrides the agent, record it. This is one of the highest-signal inputs into the Phase 9 data flywheel — every override is, in effect, a free annotation.

### Release strategy

- [ ] **Start with a limited rollout.** Begin with a small, defined user population — internal users, a beta cohort, or a constrained scope of queries. Validate behavior in the real world before expanding.

- [ ] **Define go/no-go criteria before launch.** What would trigger a rollback? Document a threshold (error rate, escalation rate, specific failure class) that triggers an automated or manual rollback — before any user is on the system.

- [ ] **Ensure all instrumentation is live before users arrive.** Production monitoring (Phase 8) must be active from the first real request. You cannot retroactively add observability to production traffic.

---

## Output

A **live agent** with:
- Active layered guardrail stack
- Documented and tested escalation paths
- Instrumentation live (traces, metrics, alerts)
- Go/no-go criteria documented
- Rollback procedure defined
