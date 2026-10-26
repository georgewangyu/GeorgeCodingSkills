# Bitter lesson: start-playbook

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. 
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/bitter-lesson-check/scripts/bitter_lesson.py`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| Skill Discovery | strong | Routing tree is team protocol. Could be generated from skill frontmatter or learned from routing history. | 2027-04-26 | | |
| 1. Surface Assumptions | strong | Protocol. The local override narrows it: proceed on reversible work, stop on prod. | 2027-04-26 | | |
| 2. Manage Confusion Actively | soft | Coaching for today's tendency to guess. | 2027-01-26 | An ablation shows models stop and ask on conflicting specs without it. | |
| 3. Push Back When Warranted | soft | Coaching against sycophancy. | 2027-01-26 | An ablation shows no change in pushback on bad approaches without it. | |
| 4. Enforce Simplicity | soft | Prose coaching. The durable form is size caps and reviewer gates. | 2027-01-26 | A diff size cap or a simplicity reviewer gates every change. | |
| 5. Maintain Scope Discipline | strong | Could become a hard diff-scope check against the task. | 2027-04-26 | | |
| 6. Verify, Don't Assume | invariant | Feedback signal. Includes the Definition of Done in `references/definition-of-done.md`. | | | |
| Failure Modes to Avoid | soft | Restates behaviors 1 to 5 as a list. | 2027-01-26 | Behaviors 1 to 5 are deleted or gated, or an ablation shows no change. | |
| 2. Skills are workflows | soft | Rigid step order makes up for weak judgment on which steps matter. | 2027-01-26 | Models pick steps by judgment and an ablation shows no skipped verification. | |
| Lifecycle Sequence | soft | Fixed sequence is a heuristic. | 2027-01-26 | Routing picks the right skills without a fixed sequence. | |
| Quick Reference | soft | Hand-kept copy of skill descriptions. | 2027-01-26 | The table is generated from skill frontmatter. | |
| Local overrides | strong | Local protocol on top of upstream. | 2027-04-26 | | |
| Verify: the checks that run | invariant | Feedback signal: the change's own CI is the build gate. | | | |
| Surface assumptions: on reversible | strong | Protocol. Could become a hard gate on production actions. | 2027-04-26 | | |
| Comments: comment cleanup only | strong | Protocol. A diff-scoped comment reviewer can enforce it. | 2027-04-26 | | |
