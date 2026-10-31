---
name: observability-and-instrumentation
description: "Make production behavior visible and diagnosable, written alongside the feature like tests. Define working first (the two to four questions on-call will ask), pick the signal for each (metric for how often and how fast, log for why, trace for where), metrics over logs for timings and counts, structured logs with a correlation ID and no secrets, RED and USE with bounded labels and percentiles, symptom alerts with a runbook and a justified threshold, and verify the telemetry itself. Includes reading rules from real investigations: know what a metric measures, small denominators flap, a missing metric is a signal. Use when adding logging, metrics, tracing or alerts, shipping anything that runs in production, reviewing a change with I/O, retries or queues, or after an incident that was hard to diagnose."
---

# Observability and instrumentation

Code you cannot observe is code you cannot operate. Instrument while you build, the same way you test. If a feature ships blind, the first bug report becomes archaeology instead of a query.

Your organization's telemetry stack, alarm conventions and data-handling rules win over this skill.

**Not for**: a failure happening right now (`debugging-and-error-recovery`), measured slowness (`performance-optimization`), launch-day watch and rollback triggers (`shipping-and-launch`, which this skill feeds).

## 1. Define "working" first [bl: strong]

Before adding any telemetry, write two to four questions on-call will ask:

```
FEATURE: payment retry
1. What share of payments succeed first try vs after a retry?
2. When one fails for good, why (provider error, timeout, validation)?
3. Is the provider slower than usual?
```

Every signal you add answers one of them. If you cannot name the questions, you are not ready to instrument.

## 2. Pick the signal [bl: strong]

| Signal | Answers | Cost |
|---|---|---|
| Metric | How often, how fast, in aggregate | Fixed per series, cheap to query and alarm |
| Structured log | What happened in this one case, and why | Grows with traffic |
| Trace | Where the time went across services | Per request, usually sampled |

- **Metrics over logs for timings and counts.** A duration or a count you will graph or alarm on is a metric. Parsing it back out of log lines is slow, costly and breaks when the text changes.
- Metrics say **that** something is wrong, traces say **where**, logs say **why**.

## 3. Structured logs [bl: strong]

- **Events, not prose**: a stable event name plus machine-readable fields (`event=payment_failed provider=... error_code=... attempt=3`), not an interpolated sentence.
- **Levels mean something**: error (an invariant broke, someone may act), warn (degraded but handled), info (a significant event), debug (detail, off in production).
- **A correlation ID on every line**, set at the boundary and carried on every outbound call and queue message. When several entry points write one log (scheduler, replay endpoint, manual run), stamp the entry point too.
- **Never log secrets, tokens, passwords or full personal data.** Allowlist fields; never log a whole request body.
- **Parameterize**, do not concatenate, and log an exception with its cause.

## 4. Metrics [bl: strong]

- **RED** for every endpoint and dependency: rate, errors, duration. **USE** for every resource (queue, pool, host): utilization, saturation, errors.
- **Durations are histograms; read percentiles** (p50, p99, max). An average hides the users having the worst time.
- **Labels come from small fixed sets** (route template, status class, provider). Never a user ID, raw URL, request ID or error text: that is a cardinality bomb and belongs in logs.
- **A metric's name and meaning are a contract.** Dashboards and alarms read it. Renaming it or changing what it counts is a breaking change (see `api-and-interface-design`): find its readers first.

## 5. Read your own metrics correctly [bl: strong]

From real investigations, these recur:

- **Know what a metric measures (numerator, denominator, reference point) before reading its value.** A backlog gauge stayed flat in an outage because both the head and the progress pointer froze.
- **Small denominators make rates flap.** A test environment's availability alarm fired on 3 errors in 200 samples. Alarm on a minimum count as well as a rate.
- **A missing metric is a signal, and alarm wiring can be wrong.** A metric vanished because its emitting thread died. Another alarm targeted a nonexistent partition and silently fell back to global data. Alarm on missing data where silence means broken.

## 6. Traces [bl: soft]

Use the vendor-neutral standard (OpenTelemetry) with auto-instrumentation for HTTP, RPC and database clients. Add manual spans only around meaningful units of work, with the attributes on-call will filter by. Carry context across every async boundary or the trace dies at the gap. Sample low by default; keep errors.

## 7. Alerts [bl: strong]

- **Alert on symptoms users feel** (error rate, latency percentile, queue age), not causes (CPU, one restart). Causes go on a dashboard.
- **Every alert is actionable.** If the response is "ignore it, it self-heals", delete it. Treat "auto-recovered" as unexplained, not resolved.
- **Every alert links a runbook**, even three lines: what it means, the first check, who to escalate to. Update the runbook when you close an incident that used it.
- **Threshold and duration come from the SLO or from history**, backtested across the population, not guessed. Global aggregates miss a single bad partition; tight per-unit thresholds false-page small fleets.
- **Two severities**: page (users hurt now) and ticket (degradation, this week).

## 8. Verify the telemetry itself [bl: invariant]

Instrumentation is code and can be wrong. Before calling it done:

- Force an error in a test environment and find it by correlation ID. Fields are structured, not `[object Object]`.
- Send test traffic. The series appear with the expected labels and sane values.
- Follow one request end to end in the trace view.
- Fire each new alert once (lower the threshold briefly). It reaches the right place and the runbook link works.

## Red flags

- A change with retries, queues or external calls and no new telemetry.
- A timing or count you can only get by parsing logs.
- Interpolated log lines, no correlation ID, or secrets in output.
- Unbounded metric labels; latency as an average only.
- A rate alarm with no minimum sample count.
- An alert with no runbook, or one that fires daily and gets acknowledged.
- A metric renamed or redefined with no search for its readers.
- "Works on my machine" as the only evidence a production feature is healthy.

## Verification

- [ ] On-call questions written; each signal maps to one.
- [ ] Timings and counts are metrics; logs are structured, correlated and clean of secrets.
- [ ] RED or USE in place, bounded labels, percentiles queryable.
- [ ] Each new alert is a symptom, has a runbook, a justified threshold and was test-fired.
- [ ] A forced failure was found from telemetry alone.

## Sources

- [addyosmani/agent-skills `observability-and-instrumentation`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/observability-and-instrumentation/SKILL.md) (MIT, see `THIRD_PARTY_LICENSE-agent-skills`): define working first, the signal table, structured logs, RED and USE, cardinality, symptom alerts, runbooks, verify the telemetry.
- [garrytan/gstack](https://github.com/garrytan/gstack) `canary` (see `THIRD_PARTY_LICENSE-gstack`): compare against a baseline, alert on changes not absolutes.
- Section 5 and the alert threshold notes are principles mined from the author's own production investigations (wording shared with `debugging-and-error-recovery`).
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
