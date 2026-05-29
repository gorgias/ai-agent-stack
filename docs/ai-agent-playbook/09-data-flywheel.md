# Phase 9 — Data Flywheel

> **Output:** A running data flywheel — signal capture, trace selection, annotation, failure taxonomy, and golden dataset enrichment, all instrumented and owned
> **Gate:** Begins the moment production traffic flows. The flywheel is not a maturity stage you graduate to — it is the operating system of an agent in production.

---

## What this phase is

Once production traces are being captured (Phase 8), the question becomes: how do you systematically convert them into improvements? Without structure, production data accumulates and is never used. Engineers triage incidents reactively; the agent improves only when something visibly breaks. This is not iteration — it is firefighting.

The **data flywheel** is the structured loop that turns every production interaction into supervised data, every supervised data point into a stronger evaluation, every stronger evaluation into a better agent, and every better agent into more — and qualitatively richer — production interactions. It is the single most important operational concept in agent development. An agent without a data flywheel plateaus. An agent with a working one compounds.

> Production traces are not exhaust — they are the raw material the agent is built from. The data flywheel is how you refine that material.

### The loop, in stages

1. **Trace generation** — every production run emits a structured trace (set up in Phase 8).
2. **Signal capture** — quality and behavioral signals are attached to each trace at runtime (automated eval scores, guardrail trips, user feedback, escalations, latency, cost).
3. **Trace selection** — a routing layer pulls a subset of traces into the annotation queue based on signal-driven criteria. The rest stay searchable but unreviewed.
4. **Annotation** — a domain expert inspects each selected trace and produces a structured label.
5. **Failure grouping** — annotated traces are grouped into a living list of failure modes (the *failure taxonomy*).
6. **Golden dataset enrichment** — high-signal annotated traces are promoted into the golden dataset, with provenance.
7. **Evaluation strengthening** — the enriched dataset feeds offline evals and recalibrates the LLM-as-a-judge.
8. **Improvement** — Phase 10 acts on the taxonomy and the strengthened evals. This is where prompt, model, tool, and retrieval changes happen.
9. **More traces** — the improved agent generates more — and qualitatively different — production traffic, and the loop runs again.

Each turn of the loop should *measurably* improve coverage and quality. If turns of the loop produce no measurable change, the loop is broken — diagnose where the chain stalls.

---

## Minimum viable flywheel

Before reading the full checklist, anchor on the smallest version of the flywheel that still works. A first-time agent team can ship this and start compounding within a week:

- **Capture three signals per trace:** automated eval score, escalation flag, explicit user feedback (thumbs / rating).
- **Route into the review queue from three sources:** a uniform random sample for unbiased coverage, low automated eval scores, and every escalated conversation.
- **Have one domain expert review the queue on a weekly cadence**, producing a verdict (correct / incorrect / partially correct) plus a short note on what went wrong.
- **Group the notes** into named failure categories. Add to the list when new patterns appear; merge when categories overlap.
- **Promote high-signal annotated traces** into the golden dataset, recording where they came from.

That is the loop. Everything else in this chapter exists to scale or harden this baseline once it is actually running.

---

## Checklist

### Signal capture (the queue's inputs)

- [ ] **Attach every available signal to every trace.** A trace without signal context is hard to triage and useless for routing. Each trace should carry: automated eval scores, guardrail trip events, user feedback (thumbs, ratings, reports), escalation events, follow-up question indicators, latency, cost, tool call count, and any business outcome metric (resolution, return, conversion, refund).

- [ ] **Capture both explicit and implicit user feedback.** Explicit feedback (thumbs, ratings, reports) is high-signal but rare — typically under 1% of interactions. Implicit feedback (retries, escalations, follow-up clarifications, copy/paste, abandonment) is noisier but abundant. Use both. Do not depend on either alone.

- [ ] **Treat escalation as a primary failure signal.** When a conversation hands off to a human, the agent did not complete the task. Escalated traces are often a richer source of failure information than thumbs-down ratings.

### Trace selection (routing into the queue)

- [ ] **Do not pick traces manually.** Manual selection embeds reviewer bias, anchors on memorable incidents, and misses long-tail behavior. Routing must be rule-driven and reproducible.

- [ ] **Run the default trio of sampling strategies, each with its own quota.** A queue with one feeder is a queue with one blind spot. Start with these three — they cover most of what a first-year agent needs:

  | Strategy | What it catches |
  |---|---|
  | *Random baseline* | Unbiased coverage of the true production distribution — the "what does normal traffic actually look like" feed |
  | *Metric-driven* | Low automated eval score, guardrail trip, high latency, high cost, anomalous tool call count — the "something measured looks wrong" feed |
  | *Outcome-driven* | Escalations, user-reported issues, negative explicit feedback — the "the user told us it failed" feed |

  Document each strategy's quota explicitly so reviewer capacity is shared, not contested.

- [ ] *(Advanced)* **Add specialized sampling strategies as the flywheel matures.** Once the default trio is running and reviewer throughput is steady, layer in the more targeted feeds below. They are not required for a first-time deployment — adopt them when the default trio stops surfacing new failure modes:

  - *Uncertainty sampling* — traces where the LLM judge had low confidence or disagreed with itself across runs
  - *Stratified sampling* — sample within each user type, feature, intent category, or customer segment to ensure tail coverage
  - *Cluster-based sampling* — embed user inputs, cluster them, and sample proportionally with deliberate oversampling of small clusters (edge cases tend to live in small clusters)

- [ ] **Size the queue against reviewer capacity, not trace volume.** Routing every interesting trace produces an untouchable backlog. Cap intake at what reviewers can actually process at the configured cadence. A queue that consistently overflows is a queue that is silently dropping signal.

### Annotation (the human-in-the-loop work)

- [ ] **The annotator is a domain expert, not the engineer.** The same role identified in Phase 5. Annotation is the calibration anchor for everything downstream — a wrong label propagates through the entire flywheel.

- [ ] **Define the annotation output schema before reviewers start.** Inconsistent labels cannot be aggregated. At minimum, capture: a verdict (correct / incorrect / partially correct), the failure category (drawn from the taxonomy), a corrected expected output, free-text rationale, and a promote-to-golden flag.

- [ ] **Use multiple annotation modalities where appropriate.** Industry-validated annotation types include:
  - *Verdict labels* — binary correct/incorrect; easiest to align on; the cleanest input for LLM-as-a-judge calibration
  - *Pairwise preference* — when two candidate outputs exist, which is better; powers preference-based fine-tuning and judge alignment
  - *Adoption decision + rationale* — would the reviewer actually use this output in production; combines a verdict with operational realism
  - *Context relevance* — was the retrieved context useful; routes signal to retrieval and RAG improvements
  - *Missing-knowledge identification* — what *should* have been in context but wasn't; feeds knowledge base improvements

  Choose the subset that matches your agent's actual failure surface — and be explicit and consistent. Adding modalities later is expensive.

- [ ] **Prefer binary verdicts over scaled labels when possible.** Binary labels are easier to align on, easier to calibrate against, and easier for LLM-as-a-judge to reproduce. Reserve scaled labels for dimensions that genuinely require them (helpfulness, tone, completeness).

> *Advanced — only relevant once you have more than one reviewer.* With multiple annotators, two practices become useful: (a) **measure how often reviewers agree** on a shared sample, and refine the rubric when they don't (low agreement almost always means an unclear rubric, not a bad reviewer); (b) **designate one reviewer as the tie-breaker** so disagreements have a canonical resolution. Formal agreement metrics exist for this — *Cohen's kappa* for two reviewers, *Krippendorff's alpha* for three or more — and become worth computing once you have a stable multi-reviewer setup. With a single reviewer, both practices are noise.

### Error analysis (turning labels into a structured list of failure modes)

- [ ] **Use a structured error-analysis process, not ad-hoc impressions.** The process is straightforward — the names for it come from qualitative-research literature, but the work itself is mundane:

  1. *Read each failing trace and write a short note on what went wrong.* Focus on the **first** observable failure, because upstream errors mask downstream ones.
  2. *Group similar notes into named categories.* This is the most important step — the categories become the taxonomy.
  3. *Count traces per category.* This exposes what hurts most in volume, not just what is most memorable.
  4. *Stop when new traces stop revealing new categories.* In practice this means roughly 20 consecutive new traces without a new group. Plan for at least 100 reviewed traces before calling the taxonomy stable.

- [ ] **Maintain a living failure taxonomy.** The taxonomy is a versioned document listing each failure category, its definition, an exemplar trace, current frequency, and the open improvement work addressing it. This is the agreed-upon map of what is wrong with the agent — it is the primary input to prioritization in Phase 10.

- [ ] **Re-run the grouping periodically.** Categories die when fixed, merge when redundant, split when too coarse, and are added when new failure modes emerge. A static taxonomy is a stale one. Treat it like the golden dataset: versioned, owned, reviewed on cadence.

### Dataset enrichment (closing the loop)

- [ ] **Every promoted annotation enters the golden dataset with provenance.** Record: source trace ID, annotator, date, failure category, reason for promotion. Without provenance you cannot trust the dataset later — and you cannot revise an entry whose context you have forgotten.

- [ ] **Promote into the matching subset.** Annotations that test refusal go into `boundary`. Annotations that exercise prompt injection go into `adversarial`. Routine failures go into `edge`. Use the subset structure defined in Phase 5 — do not dump everything into one bucket.

- [ ] **Use annotated traces to calibrate LLM-as-a-judge.** A continuously enriched bank of human-labeled examples is the most reliable calibration sample for an LLM judge. Re-check judge agreement against this bank on a defined cadence — monthly is a common starting point. When agreement drops, recalibrate the judge before trusting its scores again.

- [ ] **Pre-filter annotations before promoting to high-signal datasets.** Not every annotation should be promoted. Apply rule-based filters (reviewer disagreement, low confidence) and an LLM-based consistency check (does the label cohere with the reviewer's free-text rationale and the actual response?). Published industry results show 14–35% of raw annotations are typically filtered as noise. Promoting unfiltered annotations poisons the dataset and the judge.

### Operating the flywheel

- [ ] **Assign a named owner for the flywheel as a whole.** Not the queue alone, not the dataset alone — the entire loop. This person is accountable for queue health, annotation throughput, dataset growth, and taxonomy maintenance. Without single-point ownership, the loop stalls in the handoffs.

- [ ] **Schedule a regular review cadence.** Weekly is the most common starting point. Daily is required for high-volume agents with rapidly evolving traffic. The cadence is non-negotiable: a flywheel that turns inconsistently does not compound.

- [ ] **Instrument the flywheel itself.** Track and review:
  - *Queue depth* — how many traces are waiting to be annotated
  - *Time-to-process* — median time from trace selection to completed annotation
  - *Promotion rate* — fraction of annotated traces that enter the golden dataset
  - *Dataset growth* — entries added per period, by subset
  - *Judge-to-human agreement* — current agreement rate between LLM-as-a-judge and the reviewer on a held-out calibration sample
  - *Production-improvement attribution* — fraction of recent agent improvements traceable to a flywheel-sourced finding

  These metrics tell you whether the flywheel is spinning or stuck. Stalled flywheels are the worst kind of debt: the agent quietly stops improving while still looking healthy on the dashboard.

---

## Anti-patterns

The traps below show up in every team that builds a flywheel without naming them first. Recognize them before they take root:

- *The accumulating queue* — traces are routed in but never annotated. The queue is a backlog, not a flywheel.
- *The annotated-but-orphaned label* — reviewers annotate, but annotations never reach the golden dataset. Effort spent, no compounding.
- *The frozen taxonomy* — failure categories defined once and never revisited. The agent's failure surface evolves; the taxonomy must too.
- *The unattributed improvement* — agent changes are deployed without linkage to flywheel-discovered findings. You cannot tell what the flywheel is producing, and the case for resourcing it weakens over time.
- *The unrouted feedback* — user thumbs-down land in a metric and stop there. They should route the trace into the queue.
- *The one-signal queue* — only low-eval-score traces enter the queue. Coverage of the true production distribution is lost, and tail failures stay invisible.

---

## Output

A **running data flywheel** consisting of:
- Signal capture wired into every production trace
- Documented trace selection criteria across the default trio (random + metric-driven + outcome-driven), with quotas, plus any advanced strategies added later
- Annotation workflow with a defined schema and a calibrated rubric (with multi-reviewer agreement measurement layered in only once you have multiple reviewers)
- A maintained, versioned failure taxonomy
- Dataset enrichment loop with provenance and subset routing
- Named owner, scheduled review cadence, and instrumented flywheel health metrics

> 📋 Template opportunity: Failure taxonomy schema (category · definition · exemplar trace ID · frequency · open improvement · owner · last reviewed).
