# Bitter lesson: test-driven-development

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-30 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| 0. Use the repo's own verification skill first | strong | Protocol: the repo's own skill knows its launch, harness and evidence better than any generic rule. Could become automatic discovery by the harness. | 2027-04-30 | | |
| 1. Find how this repo tests | invariant | Rules of the game: the framework, its migration state and the merge gate are facts of the repo. No model can infer them without reading. | | | |
| 2. Real-world first | strong | Team judgment: we wrote too many low-value unit tests. A smarter model may pick the rung itself; until an ablation shows that, keep the ladder. | 2027-04-30 | | |
| 3. Every test names the real failure it catches | invariant | Feedback signal: a test that names no real failure cannot be judged, by the author or a reviewer. Holds for any model. | | | |
| 4. Pick the seams | strong | Protocol from Matt tdd. Could be learned from plan history once seams are always named in the slice. | 2027-04-30 | | |
| 5. Strict, focused, or none | strong | The pstack loose stance as a routing table. A stronger model may judge test cost directly. | 2027-04-30 | | |
| 6. One seam at a time, and show it can fail | strong | Protocol from Matt and superpowers: a test never seen failing may test nothing, and bulk tests check imagined behavior. A stronger model may judge when red is already obvious. | 2027-04-30 | | |
| 7. Prove-it for bugs | strong | Proven practice in every source. Could become a hard check (a bug CR without a repro or a stated reason). | 2027-04-30 | | |
| 8. Test behavior, not implementation | invariant | Feedback signal: a test pinned to internals breaks on refactor and sleeps through bugs, for any model. | | | |
| 9. Coverage gates | strong | Facts about how new-line gates and ratchets fail. Changes with the coverage tool. | 2027-04-30 | | |
| 10. Audit low-value tests | soft | A checklist that makes up for weak judgment on test value. | 2027-01-30 | A model writing tests produces no low-value catalog matches in an ablation. | |
| 11. Done means evidence | invariant | Feedback signal: an unlinked claim of green is not evidence, whatever the model. | | | |
