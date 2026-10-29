---
name: build
description: Build a plan one slice at a time. Each slice is the smallest complete change, ends in a pull request (or a new revision of one) whose CI is green and whose done check passed, and gets a fresh-context review before the next slice starts. Fresh worker or subagent per slice when slices are independent. Rule 0 simplicity, scope discipline (log, do not fix, anything outside the slice), safe defaults, rollback-friendly. CI on the change is the gate, not a local run. Use when the user says "build", "implement the plan", "build S1", "work the frontier", "next slice", or when a plan or a one-PR spec is ready to implement.
---

# Build

One slice, one green PR revision, one review, then the next slice. Never build on a red base.

## When to use

- A `plan` exists and a slice is on the frontier, or a one-PR `spec` is ready (its done check is the plan).
- A change touches more than one file, or you are about to write more than about 100 lines before anything checks them.

Not for:

- No spec or plan yet: `spec`, then `plan`. Unsure which API or library behavior is real: `source-driven` first.
- One-line, one-file fixes. Just make the change and let CI check it.

The chain is `spec` (what and the check) -> `plan` (slices, order) -> `build` (one slice at a time) -> review.

## Read the slice first [bl: strong]

Read the slice block from the plan before touching code: Goal, Done check, Verify, Blocked by, Files, Size, Maps to. For a full plan, also read the design contract and the slice's consumes and produces. Check every blocker is cleared, including deploy edges ("S1 deployed to staging" means deployed, not merged). If the slice has no done check that can fail, stop and fix the plan, not the code.

## The slice loop [bl: invariant]

```
read slice -> implement smallest complete change -> self-check -> commit
     ^                                                               |
     |                                                               v
 next slice <- mark done <- review (fresh context) <- CI green + done check
```

1. **Implement** the smallest change that makes the slice's Goal true. Tests in the same commit.
2. **Self-check** the diff: only the slice's files, no stray edits, no build output staged (`git add <files>`, never `git add -A`).
3. **Commit** with a message that says what the slice makes true.
4. **Open or update the PR** for the slice, as a draft unless the user says otherwise. CI on the PR is the build gate. "It passed on my machine" is not evidence.
5. **Wait for CI** on every required job. Red means the fix loop below, not the next slice.
6. **Run the done check** named in the slice (unit tests at the edges, coverage on new lines, the metric in staging). Paste the evidence link, not "tests pass".
7. **Review in fresh context**: a reviewer subagent that sees the slice, the diff and the global constraints, not your session. Guard the bar: a lowered threshold, an easier test, a silenced checker or an unfinished stub is a finding, not a fix.
8. **Mark the slice done** in the plan index with the PR link. Only then start the next slice.

## Who builds each slice [bl: strong]

- **Independent slices (different files or packages): one fresh worker per slice**, with the slice as its brief (the `spec` brief format plus a Plan section). The brief always links the plan and its design contract; out of scope names the other slices.
- **Inside a worker: subagents for within-slice work** (writing tests, fixing CI, reviewing).
- **Coupled slices sharing a contract:** one worker, in order. The contract slice goes first.
- **Never two workers on the same files at once.**
- Do not paste prior slices' history into a brief. Give the slice, the interfaces it touches and the global constraints. Hand big artifacts over as file paths.

## Rule 0: simplicity first [bl: strong]

Before writing code, ask what the simplest thing that could work is. After writing it:

- **Subtract before you add.** Remove dead code, stub references and redundant guards first, then build on the smaller base.
- **Minimize the diff.** Fewer lines beat elegant boilerplate. Three similar lines beat a premature abstraction.
- **One source of truth per decision.** Do not repeat the same choice in several places; pass the result.
- **No speculative generality.** No settings, validators or layers the spec did not ask for.
- Would a staff engineer ask "why didn't you just ..."? Then do that.

## Scope discipline: log, do not fix [bl: strong]

Touch only what the slice needs. Anything else you notice goes in a list at the end of the report:

```
NOTICED, NOT TOUCHED:
- <file or symbol link>: <what>  -> want a ticket for it?
```

No drive-by refactors, import reshuffles, modernized syntax or extra features. Never delete a comment you do not understand. Leave the files you do touch a little cleaner, but no further.

## Change rules [bl: strong]

- **One logical change per revision.** A refactor and a behavior change are two slices.
- **Keep it green.** After every revision CI passes and existing tests pass. Never leave the main branch broken between slices.
- **Safe defaults.** A new setting defaults to today's behavior. The slice that flips it is separate and has its own deploy edge.
- **Rollback-friendly.** Additive first. Do not delete something and replace it in the same revision. Name what a rollback of this slice leaves behind.
- **Flags for incomplete work.** If a slice must merge before the feature is ready, it ships behind a flag that is off.

## Fix loop [bl: strong]

- Red CI: read the failed job's log, fix the root cause, push a new revision. Do not retry unchanged code.
- Review finding: fix it in the same slice, push a revision, re-review only the fix.
- Cap at three rounds per slice. Past the cap, write a ruling (what you decided, why, what it costs if wrong) and either park the finding for the user or re-plan the slice. Rounds past the cap do not converge.
- A failure that also breaks the main branch or other PRs is not your diff. Show the evidence (the same job failing with no PR) and move on.

## Rulings, not stalls [bl: strong]

A running build does not wait on a human for two-way doors. Decide conflicts, ambiguities and small plan defects; record each as a plan revision `Ruling: <decision> - <why> - <cost if wrong>` and keep going. Stop only for one-way doors: merging, publishing, production changes, deletes, or a plan so broken that every path is a guess.

## Keep a ledger [bl: strong]

Context does not survive compaction. The plan's index (slice, PR link, revision, CI, done check, review verdict) is the ledger. Update it after every slice. On resume, trust the ledger, `git log` and the PR page over memory, and never rebuild a slice the ledger marks done.

## Non-interactive runs

Workers and loops have no live user. Build the slices in the brief, open drafts only, take the recommended answer on two-way doors and log it as an assumption, and write the ledger and the NOTICED list into the report. Never merge or publish.

## Red flags

- More than about 100 lines written with nothing checking them.
- A slice started while the previous one is red, unreviewed or blocked by an undeployed edge.
- "Tests pass" with no CI link.
- A revision mixing a refactor with a behavior change, or touching files outside the slice.
- A setting added and flipped in the same revision.
- Fixes made in the controller's own context instead of the slice's worker.
- The same command re-run on unchanged code "to be sure".

## Worked example

Slice S1 of the `plan` example: a service health check that fails a host only when all of its workers are degraded.

- **Read the slice.** S1: add a degraded-workers percentage threshold, default 100, replacing a hardcoded "all workers degraded" check. Done check: 100 matches old decisions, 80 fails at 19/23 and not at 18/23, zero workers never fails.
- **Safe default.** Default 100 keeps today's behavior, so S1 ships with no behavior change; the value 80 was its own slice.
- **The gate.** The worker's local test run hung for 90 minutes at full CPU in an unrelated test. It was killed and the worker pushed the revision to rely on CI. That is why step 4 trusts CI, not a local run.
- **Fix loop.** The first CI runs were red because no one was watching them. The fix loop found a lint error (operator wrap) and a no-op test fixture, fixed both, and every job went green.
- **Scope discipline.** While reading the check, the worker noticed a "dry run mode" setting that nothing reads, so the check was live all along. It reported this as NOTICED, NOT TOUCHED rather than fixing it inside S1.
- **Ruling.** An ordering check showed older builds ignore the unknown setting, so S2's deploy edge dropped and S2 folded into the same PR (plan Revision 1). Both kept their numbers.
- **Not done yet.** A later revision failed a coverage-decrease check, so the ledger keeps S1 open until that check is green (step 6).

## Sources

- [addyosmani/agent-skills `incremental-implementation`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/incremental-implementation/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): the increment cycle, rule 0 simplicity, scope discipline with the noticed list, one thing at a time, keep it compilable, flags, safe defaults, rollback-friendly, the red flags.
- [obra/superpowers `subagent-driven-development`](https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/subagent-driven-development/SKILL.md) at `2ce8aa1`, MIT (see `THIRD_PARTY_LICENSE-superpowers`): a fresh implementer per task, a review gate after each task, a capped fix loop, rulings instead of stalls, the ledger that survives compaction, briefs as files with no pasted history, no parallel implementers on shared files.
- [cursor/plugins pstack `principle-sequence-verifiable-units`](https://github.com/cursor/plugins/blob/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills/principle-sequence-verifiable-units/SKILL.md), `principle-laziness-protocol` and `principle-subtract-before-you-add` at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): each unit ends verifiable before the next starts; minimize the diff, one source of truth per decision; subtract before you add.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
