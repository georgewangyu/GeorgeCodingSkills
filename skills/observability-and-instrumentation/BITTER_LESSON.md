# Bitter lesson: observability-and-instrumentation

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Define "working" first | strong | Protocol that stops log-everything; the questions could be required in a PR template. | 2027-04-30 | | |
| 2. Pick the signal | strong | Decision table; metrics over logs for timings and counts is a team rule. | 2027-04-30 | | |
| 3. Structured logs | strong | Most of it can become a lint (no interpolation, required correlation field, secret scan). | 2027-04-30 | | |
| 4. Metrics | strong | Cardinality and percentile rules could be lint checks on metric definitions. | 2027-04-30 | | |
| 5. Read your own metrics correctly | strong | Principles mined from real investigations; learned examples, not yet a check. | 2027-04-30 | | |
| 6. Traces | soft | Reference a strong model already knows. | 2027-01-30 | An ablation shows the same tracing setup without it. | |
| 7. Alerts | strong | Runbook link and justified threshold could be required fields in alarm config. | 2027-04-30 | | |
| 8. Verify the telemetry itself | invariant | Feedback signal: unverified telemetry is untested code. | | | |
