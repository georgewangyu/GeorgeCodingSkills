# Bitter lesson: ci-cd-and-automation

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. One check command | strong | Structure, not judgment: one entry point is what keeps hook and CI from drifting. Could become a lint that the required job calls `check`. | 2027-04-30 | | |
| 2. Gates in order, fail fast | strong | Protocol for cost and feedback speed. The order could be checked against the `check` script. | 2027-04-30 | | |
| 3. Required checks on the default branch | invariant | Without a required check every other gate is optional; no model capability changes that. | | | |
| 4. CI failures are structured input | strong | Protocol for the fix loop; the classification could be learned from fix-round history. | 2027-04-30 | | |
| 5. Babysit a pull request, with bounds | strong | Bounds and stop-on-human protect the user's control; the polling mechanics may move into a tool. | 2027-04-30 | | |
| 6. Releases | strong | Consumer contract; release tooling can enforce most of it. | 2027-04-30 | | |
| 7. Deploys: preview, stage, canary, rollback | strong | Protocol from real deploy failures; the canary could become a scripted check per repo. | 2027-04-30 | | |
| A written rollback before the first deploy | invariant | An unrecoverable deploy is a one-way door; the plan must exist before it is needed. | | | |
| 8. Dependencies and supply chain | strong | Bots and scanners do the work; this section shrinks to a pointer once each repo has them. | 2027-04-30 | | |
| 9. Public repo hygiene | invariant | A leaked secret or a moved action tag cannot be recalled after the fact. | | | |
| 10. Keep CI fast | soft | Optimization reference a strong model already knows. | 2027-01-30 | An ablation shows no change in CI duration or job layout without it. | |
| 11. Starter snippets | soft | Per-language defaults that age with the tools. | 2027-01-30 | An ablation shows new repos get an equivalent single `check` without the snippets. | |
