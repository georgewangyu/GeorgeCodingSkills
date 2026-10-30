# Bitter lesson: quality-constraints

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-29 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| Pick the mode | strong | Brownfield detect-only, greenfield write-once, floor-only when nobody is there. Protocol that keeps a second rules list from drifting against the team's gates. A smarter model may pick on its own; the no-interview-without-a-user rule stays. | 2027-04-29 | | |
| Brownfield: report the bar | strong | Read the real gates instead of inventing rules. Could become a script that lists the gates from the build files. | 2027-04-29 | | |
| Greenfield: write CONSTRAINTS.md once | strong | Shared output format other agents read later. Four questions with defaults keep the intake short. Could become a template generator. | 2027-04-29 | | |
| Check kinds | invariant | Feedback signal: a bar made only of the agent's own tests is circular at any model size. | | | |
| Guard the bar | invariant | Feedback signal: loosening a check to pass is the cheapest road to green for any optimizer, so it must stay visible. | | | |
