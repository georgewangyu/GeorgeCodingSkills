# Bitter lesson: security-and-hardening

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Threat model first | invariant | Intent: trust boundaries and assets are facts of the system no model can infer without asking. | | | |
| 2. Three tiers | strong | Protocol that sets the human gate. Many rows could become lint or scanner rules. | 2027-04-30 | | |
| 3. Controls worth a second look | strong | Known foot-guns. Could move to scanners and tests. | 2027-04-30 | | |
| 4. In review | strong | Protocol for how security findings rank in review. | 2027-04-30 | | |
