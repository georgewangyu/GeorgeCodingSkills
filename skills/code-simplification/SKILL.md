---
name: code-simplification
description: "Make code easier to read without changing what it does. Scope to the change, understand before removing (Chesterton's fence, git history), preserve behavior exactly, follow the repo's conventions, subtract before you add, minimize reader load (layers to trace, state to hold), clarity over cleverness, one simplification at a time and refactors in their own PR. Comments the diff adds are judged by a fresh agent. Mechanical cleanup passes go to the built-in /simplify. Use when a change works but reads heavier than it should, when review flags complexity, convoluted control flow, a one-caller wrapper, a mode flag, dead code or a knob with no caller, or when the user says simplify, clean up, deslop or reduce complexity."
---

# Code simplification

Simpler means a reader understands it faster, not fewer lines. The test: can a new reader answer "where does X come from?" and "what can change X?" in under 30 seconds? Every change here keeps behavior identical. If you cannot show that, do not make it.

## When to use

- After a slice works and its tests pass, before the PR revision goes up.
- A review flags complexity: convoluted control flow, a parallel class or one-caller wrapper, a mode flag in shared code, dead code or a knob with no caller. See `code-review-and-quality` section 7.
- Code you are about to extend is hard to trace. Subtract first, then add.

Not for: code that is already clear, code you do not understand yet, a module about to be rewritten, or a hot path where the simpler form is measurably slower (`performance-optimization`).

## 0. Scope and route [bl: strong]

- Default to the code this change touched. A drive-by refactor elsewhere is a separate PR, and on shared code it needs the owner's buy-in.
- Mechanical pass (reuse, small efficiencies, obvious cleanups in the current diff): run the built-in `/simplify` and review what it changed.
- Comments the diff adds or changes: have a fresh-context agent judge each one (keep it only if it says a non-obvious why). Never delete a pre-existing comment you do not understand.
- This skill is for the judgment calls: which layer to collapse, which branch to delete, which concept to stop making the reader hold.

## 1. Understand before you remove [bl: invariant]

Chesterton's fence: know why it is there before you take it down.

- What is this responsible for, who calls it (production callers, not just tests), and what does it call?
- Read `git log -p` and `git blame` for the lines. A guard added after an incident, a timeout tuned in a past PR, or a second code path kept for a rollout is load-bearing until shown otherwise.
- Which tests define its behavior? If none, write the characterization test first (`test-driven-development`).
- If you cannot answer these, read more. Do not simplify yet.

## 2. Preserve behavior exactly [bl: invariant]

- Same outputs, same errors, same side effects, same ordering, same metrics and log lines operators rely on.
- Existing tests pass without edits. A test you had to change means behavior changed: that is not a simplification.
- Run the suite after each change, and let CI on the PR be the gate.

## 3. Follow the repo's conventions [bl: strong]

Simpler means more like the rest of the repo, not more like your taste. Match neighbor code for imports, error handling, logging and metrics, naming, test style. A simplification that brings a new idiom into a file is churn.

## 4. Subtract before you add [bl: strong]

- Delete first: dead branches, unused parameters and fields, knobs with no production caller, speculative validators, stubs, redundant guards.
- Then build on the smaller base. Leave the design a little simpler behind the same or smaller surface.
- Design for observed use, not imagined edge cases. A flag "in case we need it" is a finding, not a feature.

## 5. Minimize reader load [bl: strong]

Two axes. Guard both.

- **Layers to trace.** Collapse a wrapper with one caller, an adapter with no second impl, a layer that repeats the next one's methods. Prefer boundaries that hide a real decision.
- **State to hold.** Locals over fields, fields over module state, returns over mutation. Derive instead of syncing two copies.
- Name the invariant once at the boundary, not in every consumer.
- Before adding a layer or a piece of state, ask whether it removes at least as much load elsewhere.
- Count concepts, not lines. A refactor that leaves the same number of concepts in the reader's head only moved them. Prefer the version where a whole branch, mode or class disappears.

## 6. Clarity over cleverness, with balance [bl: soft]

| Signal | Usually becomes |
|---|---|
| An if/else chain on the same shape | a table, an enum with behavior, or a dispatcher |
| Nesting three deep | guard clauses or a named predicate |
| Boolean parameters (`run(true, false)`) | two methods, or an enum |
| A mode flag read in shared code | the difference moved into the injected impl |
| The same decision made in three places | one source of truth, passed as a value |
| A generic name (`data`, `result`, `helper`) | a name in team vocabulary |

Balance traps: inlining a helper that gave a concept its name, merging two simple functions into one complex one, removing a seam that exists for testability, optimizing for line count.

## 7. One at a time, then step back [bl: strong]

- One simplification per commit, so a red CI run points at one change.
- A refactor and a behavior change are two PRs. A small rename inside a feature slice is fine.
- More than about 500 lines of mechanical change: use a script or codemod, not hand edits.
- Afterwards compare before and after as a whole. If it is not clearly easier to read, or a teammate would push back, revert it.

## Red flags

- A test edited to keep a "simplification" green.
- A guard or second path removed with no look at its history.
- A new idiom brought into a file under the name of cleanup.
- Refactor and feature mixed in one PR.
- A "cleaner" version with the same number of concepts.
- Pre-existing comments deleted without understanding them.

## Related

- `code-review-and-quality` (design fit), the built-in Claude Code `/simplify` for the mechanical pass.
- `test-driven-development` for a characterization test, `incremental-implementation` for slice order.

## Sources

- [addyosmani/agent-skills `code-simplification`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/code-simplification/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): preserve behavior, follow conventions, clarity over cleverness, balance, scope to the change, Chesterton's fence, one change at a time.
- [cursor/plugins pstack](https://github.com/cursor/plugins/tree/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills) `principle-minimize-reader-load`, `principle-subtract-before-you-add`, `principle-laziness-protocol` and `no-comments` at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): reader load on two axes, delete before building, the smallest diff, comments judged by a fresh agent.
- Built-in Claude Code `/simplify` for the mechanical pass. compound `ce-simplify-code` and gstack `deslop-shared-libs` compared, not adopted.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
