# Bitter lesson: code-simplification

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 0. Scope and route | strong | Protocol: which tool for which pass, and review-gated packages. | 2027-04-30 | | |
| 1. Understand before you remove | invariant | Chesterton's fence: history and callers are facts no model can infer without reading. | | | |
| 2. Preserve behavior exactly | invariant | The definition of the task; CI is the gate. | | | |
| 3. Follow the repo's conventions | strong | Team taste; could be learned from neighbor code or enforced by lint. | 2027-04-30 | | |
| 4. Subtract before you add | strong | pstack principle; durable intent, could become dead-code gates (ErrorProne). | 2027-04-30 | | |
| 5. Minimize reader load | strong | pstack principle; the target of simplification, stated as a measurable test. | 2027-04-30 | | |
| 6. Clarity over cleverness, with balance | soft | A signal table a strong model already knows. | 2027-01-30 | An ablation shows no difference without the table. | |
| 7. One at a time, then step back | strong | Protocol that keeps a red CI run attributable. | 2027-04-30 | | |
