# Bitter lesson: code-review-and-quality

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 0. Route | strong | Protocol: which pass for which situation. Changes when the tools change. | 2027-04-30 | | |
| 1. Pin the change and its intent | invariant | Feedback signal: a review of the wrong snapshot or without the plan cannot be right, for any model. | | | |
| 2. Size and description | strong | Team practice from mined "why?" comments. Could become a PR description lint. | 2027-04-30 | | |
| 3. Review the tests first | strong | Protocol: tests state intent. A stronger model may order this itself. | 2027-04-30 | | |
| 4. Two passes: Spec, then Standards | strong | Protocol from Matt code-review. Separate contexts stop one axis masking the other; an ablation could show a strong model no longer needs the split. | 2027-04-30 | | |
| 5. Navigate, then find the blast radius | invariant | Feedback signal: proof by running code or reading the real call site, not prose. | | | |
| 6. Falsify the claims | invariant | Search against the code: a claim is only as good as the attempt to break it. | | | |
| 7. Design fit: what reviewers actually flag | strong | Team taste mined from 162 real comments. Each row should become a gate (lint, architecture test, static analysis) or an eval fixture, then leave this table. | 2027-01-30 | | |
| 8. Refute, then suppress | invariant | Measured: self-scored confidence is near random and suppression is the largest acceptance win. | | | |
| 9. Write the comments | strong | Style rules. Durable intent, but could be a lint on drafted comments. | 2027-04-30 | | |
| 10. Receiving review | strong | Protocol against performative agreement. A stronger model may verify by default. | 2027-04-30 | | |
