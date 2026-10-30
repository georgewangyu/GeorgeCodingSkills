---
name: performance-optimization
description: "Make something faster or cheaper only with measurement. Write the claim first, vet the measurement (the limiter, tuned like production, physical limits, errors counted, five alternating runs, end-to-end share, the work actually happened), then measure, find the bottleneck, change one thing, keep or revert (neutral is a revert), guard the metric, and log every attempt including the reverted ones. Use when a latency, throughput, GC, memory, CPU or cost problem is reported, before claiming a speedup or a regression, when tuning a pool, a timeout or a runtime flag, or when the user says optimize, perf, benchmark, load test or why is this slow."
---

# Performance optimization

Measure, find the limiter, fix it, measure again, and keep only what paid for itself. Most bad perf work is a trusted number that measured something else.

## When to use

- A latency, throughput, GC, memory, CPU or cost problem is reported or alarming.
- Before writing "X% faster" or "this regressed" in a PR, issue or doc.
- Tuning a pool size, a timeout, a worker count or a runtime flag.

Not for: a change with no evidence of a problem. A live incident: `debugging-and-error-recovery` first.

## 1. Write the claim first [bl: strong]

Write the sentence you expect to ship, with its numbers: "full GCs per host per hour drop from 7 to under 1 at normal traffic". Every later step tests that sentence. No claim, no work.

## 2. Vet the measurement [bl: invariant]

Answer each with evidence from a run, not a guess about the code. One run is fine for a ballpark someone asked for; still check 4 and 7 and say it was one run.

1. **What limits it?** Name the limiter: CPU per thread, GC, a lock, I/O, a downstream, the load generator itself. Profile in a run you do not report. If a change did not move the number, the limiter says why.
2. **Was it tuned like production?** Same build, flags, config, pools, cache warmth, data shape and versions on every side. A side on defaults compares configurations, not code.
3. **Does it break a physical limit?** Bytes per second vs disk and network, ops per second times cost vs cores. Removing a piece that takes 10% of the run can save at most about 11%. A result past a limit measured a cache, a no-op or a bug (`principles-first-principles`).
4. **Did it error?** Count failures, throttles and retries. Rejections are fast. Check outputs are right, not just present.
5. **Does it reproduce?** At least five runs per side, alternating, median and range. A gap inside run-to-run noise is no difference.
6. **Does it matter end to end?** Report a micro result as a share of the path a caller waits on.
7. **Did the work happen?** Confirm the timed region did the work: requests reached the server, the config was read, the rows were written.

## 3. The loop [bl: strong]

1. **Measure** a baseline with the same command, conditions and budget you will use after.
2. **Identify** the bottleneck from the evidence in section 2, not from the code you suspect.
3. **Fix** one thing. Three changes together give one number you cannot attribute.
4. **Keep or revert.** Better beyond noise and correctness still green: keep. Within noise, worse, or a test went red: revert. Neutral is a revert, because kept code is maintained forever.
5. **Guard** the metric the caller feels: an alert or dashboard on it, a load-test check, or a budget in CI. Not every number you have.

## 4. Log every attempt [bl: strong]

Reverted work leaves no trace in git, so the same dead idea comes back next quarter. Keep a short ledger in the issue or the investigation notes:

| Attempt | Before -> after | Kept? | Why |
|---|---|---|---|
| Bigger heap | full GCs 7/h -> 7/h | reverted | Live data was under 10% of the heap; heap size was not the limiter |

## Worked examples

- **Full GCs that were not heap pressure.** A JVM service paged on full GC count. A heap increase shipped and changed nothing. Question 1 found the limiter: the collector's adaptive sizing had shrunk the survivor space to a few MB with a tenuring threshold of 1, so short-lived objects were promoted and filled the old generation with garbage, forcing a full GC every few minutes. Live data was under 10% of the heap. The fix targets survivor sizing and allocation, not the heap.
- **A load test that measured defaults.** A throughput run on a packaged deploy looked fine, but the packaging had dropped the worker process config, so the service ran on built-in defaults. Questions 2 and 7: the run measured a configuration nobody would ship. Record a config fingerprint with every run.

## Red flags

- A speedup claimed from one run, or with no limiter named.
- One side tuned and the other on defaults.
- Several changes measured together.
- A neutral change kept "because it does not hurt".
- A reverted attempt with no ledger row.
- An optimization that skips work the product needed (a validation, a fresh read, an await).

## Related

- `principles-scientific-method` (claim, prediction, baseline), `principles-first-principles` (bound the number), `debugging-and-error-recovery` for a live problem, `code-review-and-quality` (performance in review).

## Sources

- [addyosmani/agent-skills `performance-optimization`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/performance-optimization/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): measure, identify, fix, keep or revert, guard; neutral is a revert; log every attempt. Its Core Web Vitals and frontend tables are linked, not copied.
- [cursor/plugins pstack `benchmark-checklist`](https://github.com/cursor/plugins/blob/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills/benchmark-checklist/SKILL.md) at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): the seven questions, write the claim first, alternate runs.
- garrytan/gstack `benchmark` compared, not adopted. The worked examples are the author's own, anonymized.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
