# Phase 8 — Production Monitoring

> **Output:** Monitoring dashboard with defined metrics, alerting thresholds, and trace-level debugging
> **Gate:** Active from the first production request. Not a post-launch addition.

---

## What this phase is

Once in production, the source of truth shifts from your test suite to real-world usage. Real users generate inputs you did not anticipate, surface failure modes your evals missed, and create edge cases that only exist in the wild. Monitoring is how you see this.

Monitoring an agent is more complex than monitoring conventional software. You are not just checking whether a function returned the right value — you are measuring whether a chain of reasoning, tool calls, and decisions produced a good outcome.

> Every metric you track should have a clear line to an action you would take if it degrades. Metrics without action paths are noise.

---

## Checklist

### Tracing

- [ ] **Capture full execution traces for every agent run.** A trace records the complete sequence: inputs received, reasoning steps, tool calls made (with arguments and responses), final output. Traces are essential for debugging failures that cannot be reproduced from input/output alone.

- [ ] **Log inputs, outputs, latency, and cost at the step level.** Each step should have its own timing and cost data so you can identify which step is responsible for spikes.

- [ ] **Attach evaluation scores to traces at runtime.** Where possible, run automated quality checks on production outputs and attach scores to the corresponding trace. This correlates quality signals with specific execution paths.

### Quality metrics

- [ ] **Run automated quality evaluation continuously in production.** Use the same LLM-as-judge approach from Phase 5 on production outputs. Run on a sample if cost is a constraint, but run it regularly — not just offline.

- [ ] **Track task completion rate.** What fraction of conversations result in the user's goal being met without escalation? This is the primary outcome metric for most agents.

- [ ] **Track escalation rate.** What fraction of conversations are handed off to a human? A rising escalation rate is an early signal of inputs the agent cannot handle.

- [ ] **Track guardrail trigger rate per guardrail.** A guardrail that never fires may be misconfigured. One that fires constantly may be over-sensitive and degrading user experience.

- [ ] **Track tool selection accuracy.** Are there patterns in incorrect tool calls? Systematic errors point to a prompt or tool description problem.

### Operational metrics

- [ ] **Track latency at percentile level (p50, p95, p99).** Averages mask tail behavior. High p99 latency signals problematic edge cases.

- [ ] **Track cost per conversation.** Unusual cost spikes (more tool calls than expected) signal runaway behavior.

- [ ] **Track error rates by category.** Distinguish model errors, tool call failures, guardrail trips, and infrastructure errors. Each has a different remediation path.

### Feeding the data flywheel

Traces are not just for debugging — they are the raw material for improving the agent. Phase 8 is responsible for capturing traces and attaching the signals that make them routable; turning them into supervised data is Phase 9's job.

- [ ] **Make every trace flywheel-ready.** Capture the signals downstream selection will rely on: automated eval scores, guardrail trip events, escalation flags, explicit user feedback (thumbs, ratings), implicit user feedback indicators (retries, follow-ups, abandonment), latency, cost, and tool call count. Missing a signal here means missing a routing criterion in Phase 9.

- [ ] **Expose traces to a queryable store.** Phase 9's sampling strategies (metric-driven, uncertainty, stratified, cluster-based) require querying traces by signal. A trace store you cannot query is a trace store that cannot feed a flywheel.

> See [Phase 9 — Data Flywheel](./09-data-flywheel.md) for the trace selection, annotation, taxonomy, and dataset enrichment loop that consumes these traces.

### Alerting

- [ ] **Set alerting thresholds before launch.** For each key metric, define the threshold that triggers an alert and the action it should trigger (investigate, page, rollback). Do not discover thresholds reactively after incidents.

---

## Output

A **monitoring dashboard** covering:
- Quality metrics (task completion, escalation rate, eval scores)
- Operational metrics (latency percentiles, cost per conversation, error rates by category)
- Guardrail trigger rates
- Trace access for step-level debugging
- Trace store with all routing signals attached, queryable by Phase 9's selection layer
- Alerting thresholds documented and active

> 📋 Template opportunity: Standard monitoring metrics checklist.
