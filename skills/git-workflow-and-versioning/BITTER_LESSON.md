# Bitter lesson: git-workflow-and-versioning

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Commits that do one thing | strong | Protocol for reviewable, revertable history. Message and staging rules could become commit-msg and pre-commit hooks. | 2027-04-30 | | |
| 2. Save points, then squash | strong | Protocol. The squash shape depends on the review tool; could be checked at review time (commit count, fixup subjects). | 2027-04-30 | | |
| 3. One change per checkout | strong | Protocol from real collisions of parallel agents in one tree. Could become a lock per checkout. | 2027-04-30 | | |
| 4. Push safety | invariant | Data-loss prevention: a force over someone's commits cannot be undone by a smarter model. | | | |
| 5. Guardrails as hooks | strong | This section is the move from prose to a hard check; it shrinks to a pointer once the hooks ship. | 2027-04-30 | | |
| 6. Redact before publish | invariant | A published secret or internal name cannot be recalled. | | | |
| 7. Finish deliberately | strong | Protocol: the integration decision belongs to the user's rules. | 2027-04-30 | | |
| 8. Versions and changelogs | strong | Consumer contract. Could be checked by an API diff and a changelog lint. | 2027-04-30 | | |
| 9. Git for debugging | soft | Command reference a strong model already knows. | 2027-01-30 | An ablation shows no change in how often bisect is used to find a regression. | |
