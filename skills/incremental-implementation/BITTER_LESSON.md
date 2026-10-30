# Bitter lesson: incremental-implementation

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-29 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| Read the slice first | strong | Protocol with `planning-and-task-breakdown`: the slice fields and blockers are the contract. A smarter model still cannot see a deploy edge it never read. | 2027-04-29 | | |
| The slice loop | invariant | Feedback signal: CI on the change is the gate, and a slice is done only when its check passed (pstack sequence-verifiable-units). | | | |
| Who builds each slice | strong | Fresh context per slice keeps each worker focused and the controller lean (superpowers). Longer context may shrink the need; parallel workers on shared files stays a real hazard. | 2027-04-29 | | |
| Rule 0: simplicity first | strong | Heuristic with strong evidence (Addy, pstack laziness and subtract). Could become a learned example or a diff-size check in review. | 2027-04-29 | | |
| Scope discipline: log, do not fix | strong | Reviewability: drive-by edits hide in diffs. Could become a hard check (files outside the slice's Files line). | 2027-04-29 | | |
| Change rules | strong | Safe defaults, one change per revision and rollback-friendly order are deploy protocol. Rollout ordering is one of the most repeated agent mistakes in review. | 2027-04-29 | | |
| Fix loop | strong | A cap stops non-converging retry churn (superpowers). The round count is a proxy a smarter model may not need. | 2027-04-29 | | |
| Rulings, not stalls | strong | Protocol shared with start-playbook rule 2: decide two-way doors, stop on one-way doors. | 2027-04-29 | | |
| Keep a ledger | strong | Compaction loses state; re-running finished slices is the costliest failure superpowers observed. Longer context may shrink it. | 2027-04-29 | | |
