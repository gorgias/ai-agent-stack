# Templates

One template per phase of the [AI Agent Development Playbook](../00-index.md). Each template captures only the **main points** an engineer needs to produce the phase's output artifact — not every detail in the chapter. Read the corresponding phase file when you need depth.

These are starting points. Copy a template, fill it in, version it, keep it alive.

| # | Phase | Template | Output it produces |
|---|---|---|---|
| 01 | Problem Framing | [01-problem-framing.md](./01-problem-framing.md) | Problem Framing Document |
| 02 | Agent Design | [02-agent-design.md](./02-agent-design.md) | Agent Design Document |
| 03 | Prompt Engineering | [03-system-prompt.md](./03-system-prompt.md) | Versioned system prompt |
| 04 | Tool & Integration Design | [04-tool-catalog.md](./04-tool-catalog.md) | Tool catalog with risk ratings |
| 05 | Evaluation Design | [05-golden-dataset.md](./05-golden-dataset.md) | Golden dataset |
| 06 | Iteration & Offline Testing | [06-eval-baseline.md](./06-eval-baseline.md) | Passing eval baseline record |
| 07 | Deployment & Release | [07-deployment-readiness.md](./07-deployment-readiness.md) | Deployment readiness checklist |
| 08 | Production Monitoring | [08-monitoring-dashboard.md](./08-monitoring-dashboard.md) | Monitoring dashboard spec |
| 09 | Data Flywheel | [09-failure-taxonomy.md](./09-failure-taxonomy.md) | Failure taxonomy |
| 10 | Production Iteration | [10-iteration-log.md](./10-iteration-log.md) | Iteration log |

## How to use these

- **Copy, don't edit in place.** Each agent gets its own copy. The templates here are the canonical form.
- **Fill in what's there, then stop.** The sections are deliberately minimal. If you find yourself adding ten subsections, ask whether the addition is genuinely needed or whether you're padding.
- **Keep them living.** Templates 05 (golden dataset), 09 (failure taxonomy), and 10 (iteration log) are never "done" — they grow with the agent.
- **Versioning.** Anything that defines runtime behavior — prompt, tool catalog, eval baseline — lives in source control with a changelog.
