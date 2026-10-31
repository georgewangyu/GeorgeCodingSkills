# Bitter lesson: shipping-and-launch

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Pre-launch checklist | strong | Checklist that could become a change-plan template with required fields. | 2027-04-30 | | |
| 2. A rollback that still works after the change | invariant | An unrecoverable change is a one-way door; no model capability makes a missing rollback safe. | | | |
| 3. Alarms and their changes live before release | invariant | Unmonitored release is blind; a pending alarm change cannot catch anything. | | | |
| 4. Test at the target, not below it | strong | Evidence rule from a real review; could become a required link in the plan. | 2027-04-30 | | |
| 5. Measurable rollback criteria | strong | Numbers per stage are checkable; the threshold table is a default to tune. | 2027-04-30 | | |
| 6. Worst case includes losing a zone | strong | Lesson from a real review; could become a required row in capacity plans. | 2027-04-30 | | |
| 7. Flags and staged rollout | strong | Protocol; deploy tooling enforces most of it once configured. | 2027-04-30 | | |
| 8. The first hour | strong | Protocol; a scripted canary can take it over. | 2027-04-30 | | |
