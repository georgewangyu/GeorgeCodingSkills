# Bitter lesson: deprecation-and-migration

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Decide before you deprecate | strong | Protocol; the caller count could come from a code-search script instead of judgment. | 2027-04-30 | | |
| 2. Advisory or compulsory | strong | Decision rule that protects callers; a strong model knows it but the default (advisory) is a team choice. | 2027-04-30 | | |
| 3. The process | strong | Order matters (replacement before removal, zero usage before delete); could become a removal checklist check. | 2027-04-30 | | |
| 4. Internal APIs: migrate callers, then delete | strong | Structure rule against shims with no outside users; a lint could flag unused compatibility layers. | 2027-04-30 | | |
| 5. Patterns | soft | Reference a strong model already knows. | 2027-01-30 | An ablation shows the same migration plans without it. | |
| 6. Worked example | soft | A learned example; expires once the agent reproduces the redirect pattern unprompted. | 2027-01-30 | An ablation shows renames still leave a dated redirect without it. | |
