---
name: test-driven-development
description: "Prove a change works with the most real test that is still cheap. Use the repo's own verification skill first. Real-world first ladder: an integration or component test at the seam with real collaborators, then the running environment itself (staging behavior, load tests, integration runs, metrics after deploy), and unit tests only for pure logic with real edge cases. Every test names the real failure it would catch. Loose TDD: strict red-green where the loop is cheap, no new test over a bad one, tests at agreed seams not per method, prove-it for bugs, one seam at a time. Coverage met at the gate with value tests, never padding. Use when writing, fixing or reviewing tests, when a bug report arrives, when a coverage gate fails, or when the user says test this, add tests, TDD, red green, regression test, or prove it."
---

# Test-driven development

Most suites carry too many low-value tests. A test earns its place by catching a failure that would really happen, as close to how it would really happen as you can afford. Pick the most real test that is still cheap, name the failure it catches, and watch it run.

## When to use

- New logic or a changed behavior (`build` puts the tests in the same commit as the change).
- A bug report: prove-it first, when the repro is cheap.
- A coverage gate failure, a review that asks for a test, or a test file full of near-duplicates.

Not for: config, docs, or a rename with no behavior change. UI behavior in a real browser: `browser-testing`.

## 0. Use the repo's own verification skill first [bl: strong]

Each project should carry its own skill that says how to prove behavior in that repo: how to launch it, drive it, what evidence counts, how to clean up. Look for it first:

- `.claude/skills/verify*/SKILL.md` (or `.cursor/skills/verify*/`) in the repo;
- a VERIFY or Testing section in the repo's `AGENTS.md`, `README.md` or `CONTRIBUTING.md`.

If one exists, follow it, and treat the rest of this skill as defaults it can override. If none exists and the repo will be verified again, offer to create one (pstack `create-verification-skill` is a good generator, and `maintain-verification-skill` keeps it honest). Offer, never create it unasked.

## 1. Find how this repo tests [bl: invariant]

Read before the first test. Guessing the framework or the command is how a test never runs.

- Where tests live, their naming, and the neighbor test whose shape you will copy.
- The framework and the focused-test command (`pytest path::test`, `npx vitest run file`, `./gradlew test --tests`, `go test -run`). Use the checked-in wrapper (`./gradlew`, `make test`, a repo script) over a global tool.
- If the repo is migrating frameworks (JUnit 4 to 5, Jest to Vitest), new files use the new one.
- The merge gate: what CI runs on the pull request. That is the final word, not your local run.

## 2. Real-world first [bl: strong]

Go down this ladder and stop at the first rung that is cheap enough for the change.

| Rung | What | Example |
|---|---|---|
| 1. Seam with real collaborators | An integration or component test that drives the public seam with the real classes behind it, fakes only at the process edge (network, cloud APIs, clock). | an HTTP handler test against the real service layer and an in-memory store; a job runner driven end to end with a fake queue |
| 2. The environment as evidence | Watch the behavior where it really runs. The test is the check you run there, recorded. | staging behavior, a load test, an integration suite run, a dashboard or metric after deploy |
| 3. Unit test | Only for pure logic with real edge cases. | math, parsing, a state machine, a backoff schedule |

- Rung 2 is not a fallback for laziness. It is the right test when the failure only shows under real load, timing or config, and a lower rung would only test a fake of it. Put it in the task's done check so it actually happens.
- Rung 3 is not the default. A unit test that needs mocks of your own classes to run is a sign the logic belongs at rung 1.
- Fakes over mocks at every rung. A fake is a small working implementation (an in-memory store). A mock is a script of expected calls. Do not mock your own classes or plain data objects.

## 3. Every test names the real failure it catches [bl: invariant]

Before writing a test, write one line: the real failure it would catch, the way it would happen in production. For example: "a failed upload is swallowed and the job reports success".

- If you cannot name a real failure, do not write the test.
- If the only failure it catches is "someone changed the code" (a constant, a message, a call order nobody observes), it is a change detector. Drop it.
- Put the line in the test name or a one-line comment, so a reviewer can judge it.

## 4. Pick the seams [bl: strong]

A seam is the public boundary where a caller observes behavior: a public function, a route, an emitted event, a written row. Tests go at seams, never against internals.

- Name the seams in the plan or task (`plan`, `interfaces`), one line each on what a test there catches and what it misses. In an interactive session, confirm them with the user before writing tests.
- One seam, a few behaviors. Not one test per method. A private helper is covered through the seam that uses it.

## 5. Strict, focused, or none [bl: strong]

| The test target | Do this |
|---|---|
| Cheap and local (pure logic, a fast component test) | Strict red-green: watch the test fail for the right reason, make it pass, refactor. |
| A clear seam, but the loop is slow (heavy build, CI-only setup) | Write the test first for the seam, then the code, then the next seam. Show it could fail (section 6). |
| Expensive, timing-bound, production-only state, vague repro | No new test. Use the closest executable check (rung 2, a script, a log or metric assertion, `browser-testing`) and say so. |

Prefer no new test over a bad test. A bad test mostly tests mocks, pins the current implementation, depends on timing or global state, needs broad harness work for a small fix, or would be deleted right after proving the fix.

## 6. One seam at a time, and show it can fail [bl: strong]

1. Write the test with its real failure line (section 3).
2. Run it and watch it fail for the reason you expect, not a typo or an import error. A test that passes on its first run is testing something that already works.
3. Write the smallest change that makes it pass. Then the next seam.
4. Do not write all the tests first and all the code after. That is horizontal slicing: bulk tests check imagined behavior.
5. When the loop is too slow to watch red locally, show the test could fail another way: a test-only commit whose CI run went red, or a temporary revert of the fix with the test failing.

Run the full suite with the repo's own command before calling it done, and name any failure you saw, including ones you did not cause.

## 7. Prove-it for bugs [bl: strong]

A bug with a cheap repro starts with a test, not a fix.

- Write the repro at the seam the user hit, as high on the ladder as is cheap. It asserts the correct behavior, so it fails on today's code.
- For a hard bug, have a subagent write the repro from the bug report alone, without your theory of the fix. A test written knowing the fix tends to test the fix.
- Fix, rerun: the repro passes and the rest of the suite still passes.
- Keep the repro focused on the bug. Sibling coverage for the wider class comes after, in its own change.
- No cheap repro: say so, and use rung 2 or the closest executable check.

## 8. Test behavior, not implementation [bl: invariant]

Judge each test by two edits:

- **Break the behavior** (a wrong value, a skipped side effect, an effect done twice). The test must fail.
- **Change the internals** and keep every observable result. The test must still pass.

Rules that follow:

- Assert what the contract promises: the return value, the error type callers branch on, the event, the row, the request sent. Not which private helper ran, not log wording.
- Derive the expected value from the requirement, as a literal. An expected value computed by the code under test always passes.
- Wire fixtures like the real composition root, so a passing test cannot hide a wiring difference.
- Tests that differ only in inputs become one parameterized test.
- No sleep-then-assert timing tests. Inject the clock and assert the timing logic.
- Never change a test to match a wrong implementation, and never loosen an assertion to get green.

## 9. Coverage gates [bl: strong]

Coverage is a gate, not the goal. Meet the gate with tests at the seams. Read which check failed before you write a line.

The gate is the floor. About 85% new-line coverage is a soft ceiling: go past the gate toward it only with tests that each catch a real failure, and stop when the next test would only cover lines. Never 85% for its own sake.

- **New-line coverage below the threshold.** The new lines are not reached. Read why: usually a test never gets to the branch, not a missing test. Add the seam test that reaches it.
- **A whole-project ratchet.** It can fail with new lines fully covered: it is noisy between identical runs and falls when a refactor deletes well-covered code. The answer is a seam test or a recorded override with the reason. Never padding.

A covered line with a meaningless assertion is worse than an honest gap. Name the gap in the pull request.

## 10. Audit low-value tests [bl: soft]

When you touch a test file, or a review says the tests are bloated, ask four questions of each test (gstack `test-audit`):

1. What real failure does it catch (section 3)?
2. What credible regression makes it fail?
3. Why does no existing test catch that already? Prefer a new row in an existing parameterized test.
4. Does it need a production seam (an export, a flag, a hook) only tests use? Then test at the real boundary.

Low-value signs: no assertion, a value compared with itself, a copied fixture or constant list, a mock-call test duplicated at a real boundary, the same contract invoked twice, production code whose only callers are tests. Keep a test that guards a real contract even if it is slow. Retire tests in their own change, never mixed into a behavior change.

## 11. Done means evidence [bl: invariant]

"Tests pass" with no output is not done. Report:

- the test command and its result (counts, exit code), and the CI run on the pull request;
- the evidence each new test could fail, and for a bug the repro test name;
- the rung-2 evidence when the done check named one;
- any behavior left without a test, and the check used instead.

An agent's "success" is not evidence either. Check the run yourself.

## Worked example: a coverage gate at 66.66%

A change routed error paths in three background workers to a new fault reporter. The new-line coverage gate failed at 66.66% against 70%. The uncovered lines were the new fault routes inside each worker's flush loop.

- **Section 9, read why.** The two existing flush-timer tests built the worker without its start-up step. Both returned at the "not started" check and never entered the loop that held the new lines. More tests of that shape would have added nothing.
- **Sections 2 and 3.** The real failure is "a fault on the flush path is not reported and the worker keeps going". The seam is the flush timer of a started worker with its real collaborators, asserting the fault is reported. That is rung 1, and it is what a caller observes.
- **Section 1.** The first CI runs failed on import order and line length in the new test files. Reading the neighbor files first would have saved those runs.

A second case: a whole-project ratchet failed twice (89.51%, then 89.46%, against 89.73%) with new lines fully covered and identical code. That is the ratchet's noise. The answer was an override with the reason, not more tests.

## Red flags

- A test with no real failure line, or one whose only failure is "the code changed".
- A unit test with mocks of your own classes where a seam test with real collaborators would run.
- Coverage chased past the gate with tests nobody would miss.
- The repo's own verification skill skipped, or one created without asking.
- All tests written first and the code after (horizontal slicing).
- One test per method instead of tests at the agreed seams.
- A test that passed on its first run, or an expected value computed by the code under test.
- "All tests pass" with no command output.
- A test deleted or loosened to make the suite green.

## Related

- `build` runs this per slice. `plan` names the seams and the done check, `interfaces` the contracts. `browser-testing` for UI behavior.
- `constraints` guards the bar: a lowered threshold or an easier test is a finding.

## Sources

- [cursor/plugins pstack `tdd`](https://github.com/cursor/plugins/blob/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills/tdd/SKILL.md), `create-verification-skill`, `maintain-verification-skill`, `principle-test-behavior-not-implementation` and `principle-prove-it-works` at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): the loose stance (strict only with a cheap target, prefer no new test over a bad test, the closest executable check), the project-local verification skill, the two imagined edits, failing-before evidence.
- [mattpocock/skills `tdd`](https://github.com/mattpocock/skills/blob/5beb35b040e3de97e54fa9e386d7e42d15c295c2/skills/engineering/tdd/SKILL.md) at `5beb35b`, MIT (see `THIRD_PARTY_LICENSE-mattpocock-skills`): tests at seams agreed up front, tautological tests, horizontal slicing as the anti-pattern.
- [addyosmani/agent-skills `test-driven-development`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/test-driven-development/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): discover the stack first, red green refactor, the prove-it pattern, test sizes, real over mocks, a subagent writes the repro. The pyramid is replaced here by the real-world-first ladder.
- [obra/superpowers `test-driven-development`](https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/test-driven-development/SKILL.md) with `writing-good-tests.md`, and [`verification-before-completion`](https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/verification-before-completion/SKILL.md) at `2ce8aa1`, MIT (see `THIRD_PARTY_LICENSE-superpowers`): watch it fail for the right reason, name the break, no change detectors, run the full suite, evidence before claims. Its Iron Law is not adopted.
- [garrytan/gstack `test-audit`](https://github.com/garrytan/gstack/blob/576839cb8ce73c08a05a71efa91e80ca19fa630b/test-audit/SKILL.md) at `576839c`, MIT (see `THIRD_PARTY_LICENSE-gstack`): the value bar, the low-value catalog, retire in separate batches.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
