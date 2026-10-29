# Bitter lesson: interfaces

Full analysis for the inline `[bl: ...]` tags in `SKILL.md`. Read only when
evaluating the skill (cleanup, `report`, `ablate`). One row per tagged item,
keyed by the item's first words. Verdicts from the 2026-10-29 design.
Tags: invariant = rules of the game, always follow. strong = follow it, should
become a hard check or learned example. soft = guidance for weaker models,
candidate to expire. Checker: `skills/principles-bitter-lesson/scripts/bitter_lesson.py`.

| Item | Tag | Why | Review | Delete when | Ablation |
|---|---|---|---|---|---|
| Words to use | strong | Shared vocabulary between author, reviewer and worker briefs. A smarter model knows the terms; the value is everyone using the same ones. | 2027-04-29 | | |
| 1. Contract first | strong | Protocol: later slices and parallel workers need the interface before the body exists. Could become a plan-template check. | 2027-04-29 | | |
| 2. Every observable behavior is a contract | invariant | Rules of the system: alerts, dashboards and other teams read metric names, log lines and error text. Changing one, or what it counts, breaks them whatever the model. | | | |
| 3. Deep, not shallow | strong | Heuristic with strong evidence (Ousterhout, Matt Pocock). Could become a learned example in review. | 2027-04-29 | | |
| 4. Behavior is declared by the implementation | strong | Design rule: identity checks spread branching and hide behavior. Could become a lint for identity checks. | 2027-04-29 | | |
| 5. Validate at boundaries | strong | Heuristic with strong evidence (pstack boundary-discipline). Untrusted input at the edge is also a security rule. Softens Addy's 'never validate your own database': data an older or buggy version wrote is external. | 2027-04-29 | | |
| 6. Make illegal states unrepresentable | strong | Heuristic with strong evidence (pstack type-system-discipline, model-the-domain). The compiler enforces what the model forgets. | 2027-04-29 | | |
| 7. One error strategy per surface | strong | Callers cannot handle what they cannot predict. Could become a review check. | 2027-04-29 | | |
| 8. Additive changes only | invariant | Rules of rolling deploys: old and new code run on the same data and config during rollout and after rollback. | | | |
| 9. Evolve by addition, not by a second version | strong | Addy's one-version rule: a second major version is a fork every consumer must migrate to. | 2027-04-29 | | |
| 10. Pick the update shape on purpose | soft | Softens Addy's "PATCH is what clients want". A strong model picks the shape from the data. | 2027-01-29 | An ablation shows the same choice without it. | |
| 11. State-changing calls say what a retry does | invariant | Real invariant of distributed calls: a timeout leaves the effect unknown, and retries cluster in outages (Addy). | | | |
| 12. Timeouts and deadlines are part of the contract | strong | Missing from upstream. A fixed client timeout turns a server latency change into client errors. | 2027-04-29 | | |
| 13. State ordering and concurrency guarantees | strong | Missing from upstream. Unstated order gets assumed by callers. | 2027-04-29 | | |
| 14. Async and event contracts are contracts too | strong | Missing from upstream. Datagrams and queues have delivery and duplicate semantics callers depend on. | 2027-04-29 | | |
| 15. Say what a caller gets under overload | strong | Missing from upstream. Without a named backpressure signal callers retry harder under load. | 2027-04-29 | | |
| 16. Design it twice | soft | Coaching to avoid the first idea. A strong model may weigh alternatives unprompted, and subagent cost is real. | 2027-01-29 | An ablation shows the same interface choice without the parallel drafts. | |
| Schema and IDL quirks | strong | Each IDL has non-obvious rules (enums and unions as annotations, defaults validated not applied). Reading a neighbor shape is protocol; better tooling could make it a check. | 2027-04-29 | | |
| Domain names | soft | Naming coaching. A strong model follows the project's existing vocabulary by default. | 2027-01-29 | An ablation shows the same names without it. | |
