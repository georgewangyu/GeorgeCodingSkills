# Bitter lesson: debugging-and-error-recovery

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

The principles section carries no tag: it is evidence mined from real
investigations, kept as reference, and refreshed by mining, not by ablation.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Stop the line | invariant | Rules of the game: work built on a broken step is wrong too, for any model. | | | |
| 2. Build a feedback loop that goes red | strong | Protocol from Matt diagnosing-bugs: the signal is what makes the rest mechanical. A stronger model may build it unprompted; until an ablation shows that, keep it. | 2027-04-30 | | |
| 3. Reproduce and minimise | strong | Proven practice in every source. Could become a hard check (the loop shows the user's symptom). | 2027-04-30 | | |
| 4. Can't reproduce? Debug in production | strong | Fills the gap every upstream skill leaves: production rarely reproduces locally. Learned from investigations, could become worked examples. | 2027-04-30 | | |
| 5. Rank hypotheses | strong | Protocol against anchoring. A stronger model may hold several hypotheses without being told. | 2027-04-30 | | |
| 6. Fix at the first place it went wrong | invariant | Feedback signal: a downstream fix leaves the cause in place and the bug comes back, whatever the model. | | | |
| 7. Regression guard | strong | Proven practice. Could become a hard check (a bug fix without a guard or a stated missing seam). | 2027-04-30 | | |
| 8. Verify end to end | invariant | Feedback signal: the user's symptom gone is the only evidence the bug is fixed. | | | |
| 9. Safety while debugging | invariant | Rules of the game: leaked secrets and unpreviewed mutations cause harm no model can undo. | | | |
