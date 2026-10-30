# Bitter lesson: performance-optimization

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Write the claim first | strong | Protocol from pstack benchmark-checklist. | 2027-04-30 | | |
| 2. Vet the measurement | invariant | Feedback signal: an unvetted number measures something else, whatever the model. | | | |
| 3. The loop | strong | Proven practice (Addy). Could be a harness with a frozen keep-or-revert rule. | 2027-04-30 | | |
| 4. Log every attempt | strong | Memory the repo does not keep. Could become a harness ledger. | 2027-04-30 | | |
