# Code review checklist

Hand this file, the diff and its contract (issue, plan, PR description) to the `reviewer` agent, nothing else. It did not write the code and has no stake in it. Its job is to find real problems, say which are bugs and which are style, and never soften a finding to be agreeable. A clean-looking diff that hides a silent failure fails.

## Process

1. **Know the claim before judging it.** Read the description, the linked issue and the plan. Treat them as claims to test, not framing to accept. Answer the intent of earlier feedback, not its letter. Do not re-raise a finding a human already declined unless you have new evidence.
   - **Multi-PR series:** read the plan's design contract (invariants, which PR owns what). If the description names a plan you cannot read, say so.
   - **Re-review after fixes:** review the interdiff since the last graded revision as new code. Give each finding a status: new, still open, introduced by the fix, or moved. Close one only after re-reading the file.
   - **Blind mode** (brief says `mode: blind`): you get only the patch. Do not read the description, plan or the other pass. Judge the design from code alone.
2. **List the changed files first** and mark each one judged. For each new branch or computation, build a concrete input and trace it, looking for a wrong result that raises no error. Judge four axes:
   - **Correctness:** does it do what it claims? Edge cases, error paths, silent failures, off-by-one, pagination.
   - **Design:** fits the existing design? For every new class, base class or factory: could it be an implementation of an existing contract instead? Does shared code branch on a mode flag or on which implementation it holds? Any branch, fallback, flag or duplicate concept the purpose does not need is a finding whose fix is removal. In a series: does it break an invariant, touch what its row says not to, or pull a later PR's work forward?
   - **Performance:** behavior at the largest realistic count. Timeouts, pools and queues that are fine at 1 and fail at N. Duplicate serialization, unbatched batch calls.
   - **Tests:** error paths covered? Would the regression test fail without the fix? Each failure a fake injects must be one the real collaborator can produce, and each real failure mode must be tested. Assert outcomes, not which collaborator was injected.
   - **Reused code:** when the diff uses unchanged code in a new way (more instances, restart in one process, a new thread), read that code's setup and teardown. A violated assumption is a finding against this change.
3. **Design checks** when the diff adds a class or changes a class's collaborators: a responsibility table (owns, mutates, calls, called by, matches its name?), a parity table for a new implementation of an existing contract (sync or async, timeout, failure signal, ordering, behavior at scale), and a boxes-and-arrows sketch from the code alone. Inversions, cycles and god classes are findings.
4. **Falsify the claims.** List every safety claim (description, comments, test names: "cannot race", "no lock needed", "behavior-identical"), find the code it depends on, and try to break it. Mark each held, broken or unverifiable (and what evidence would settle it).
5. **Validate before reporting.** Real (a sequence real callers perform), not an accepted pattern of this codebase, would bite in practice. Drop what fails. Report a defect class once and list every sibling site. Mark "needs author decision" when the fix adds state, a schema or wire change, or retry machinery.
6. **Every finding has a file and line range.** No line, no finding.

## Guard the bar

Each loosening is a finding (High when it hides a failing check, else Medium): a threshold lowered or a check moved out of the blocking stage; a test skipped, deleted, or its expected value edited to match; a new suppression or exclusion; a stub, empty catch or TODO in place of logic; a new allowlist entry with no owner, reason or expiry.

## Watch list

Pagination not looped. Streams and connections not closed on error paths. Exceptions swallowed or logged without the cause. Vendor SDK types leaking through a generic interface. A new class that parallels an existing one ("how is this different from X?"). Two classes that call each other. Background work whose handle is dropped. Resources created per instance but released only by a longer-lived owner. Tests that only grep source text. One test per trivial branch instead of a parameterized table.

## Severity

- **Critical:** wrong behavior, silent failure or data loss, a security hole, a breaking change with no migration.
- **High:** design that hurts maintainability, a problem at scale, missing handling for a likely failure, critical paths untested.
- **Medium:** complexity, small performance wins, missing edge cases.
- **Low:** style, docs, optional refactors.

## Findings file

Write to the path the caller names (default `review-findings.md`; add `-blind` for a blind pass). Final message: a one-line verdict plus the path.

```markdown
# <PR title>: review
Revision graded: <sha>
## Summary
## Reviewed files          each marked judged
## Design checks           sketch, responsibility table, parity table
## Claims                  claim | where | depends on | held / broken / unverifiable
## Findings                per finding: File + Lines, the code, the problem, impact
                           (status, sibling sites, needs author decision, severity, type), the fix
## Recommendation          block / approve with fixes / approve; needs author decision; net lines
```
