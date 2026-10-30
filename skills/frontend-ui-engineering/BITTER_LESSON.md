# Bitter lesson: frontend-ui-engineering

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-29 design (George).
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 1. Read the project's UI rules | invariant | Rules of the game: each project has silent failure modes (an unrebuilt committed bundle ships nothing, a generated file drifts). No model can infer them without reading. | | | |
| 2. Name the mode first | strong | The mode picks the rule set. A strong model may classify on its own; the one line is a cheap alignment check. Could become a learned example. | 2027-04-29 | | |
| 3. Structure and state | strong | House patterns. Could become lint rules (component size, prop depth) or learned examples. | 2027-04-29 | | |
| One owner per piece of state | invariant | Real invariant: two writers to one value lose updates silently (the worked example's presence lockout). Holds for any model. | | | |
| 4. Tokens, look and copy | strong | Taste calibration against generated defaults. Will drift as model defaults change; review the default list each half year. | 2027-04-29 | | |
| 5. Accessibility floor | invariant | External standard (WCAG 2.1 AA) and a user promise: aria-modal, focus and labels are contracts with assistive tech. | | | |
| 6. Verify in the running app | invariant | Feedback signal: only the rendered screen proves a UI change. Tests and green builds do not. | | | |
| 7. Doubt the risky ones | soft | Routing nudge to doubt-driven-review. Candidate to expire once doubt-driven-review's own trigger list catches UI decisions. | 2027-01-29 | doubt-driven-review is invoked on UI changes without this pointer. | |
