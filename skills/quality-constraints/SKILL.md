---
name: quality-constraints
description: The top-down half of the quality bar. On an existing package, detect the gates that already run (linters, coverage gates, static analysis, CI) and report them as the bar, with no new file. On a greenfield project, write CONSTRAINTS.md once (a short grill-me round with defaults), and later runs read it. Every rule records its origin, declared or mined. Guards the bar on every diff (threshold moved, test made easier, checker silenced, unfinished work, new exception) so loosening is loud. Use when the user says "constraints", "quality bar", "set up our standards", "what gates does this package have", when starting a new project, or when an agent keeps silencing checks to get green. The bottom-up half is mistake-miner.
---

# Constraints

Write the bar down once, check it by machine, and make lowering it loud.

## When to use

- A new project or package has no written bar.
- The user asks what the gates are, or wants standards set.
- An agent is silencing checks, skipping tests, or editing a threshold to get green.
- Before an autonomous loop runs on a package whose only gate is a test suite the agent also writes.

Not for:

- A package with a bar already (detected gates or a CONSTRAINTS.md) that nobody is changing. Read it and follow it.
- Spikes and throwaway scripts.
- Adding a rule because of a repeated mistake. That is `mistake-miner` (find and cluster) and pstack `correct` (make it impossible). This skill only records the result.

## Two halves of one bar

| Half | Skill | Source of rules | Origin tag |
|---|---|---|---|
| Top-down | `quality-constraints` | Chosen up front (interview, defaults) | `declared` |
| Bottom-up | `mistake-miner` then pstack `correct` | Repeated mistakes, proved on a past instance | `mined` (with evidence link) |

Both land in the same place: the package's gates, plus one table row per rule that says what enforces it.

## Pick the mode [bl: strong]

- **Brownfield (code exists): detect, do not write.** No new file by default. Teams already own their gates, and a second list drifts from them.
- **Greenfield (new project or package): write `CONSTRAINTS.md` once.** Later runs read it and never re-interview.
- **Non-interactive (worker, loop, CI): floor only.** Apply the floor below, say so, and flag the rest for a human. Never run the interview without a live user.

Say the mode in one line.

## Brownfield: report the bar [bl: strong]

Read, do not ask. Look for:

| Gate | Where |
|---|---|
| Style and lint | `checkstyle*.xml`, a shared style config, `.eslintrc*`, `ruff.toml`, `biome.json` |
| Static analysis | SpotBugs, PMD, ErrorProne, ArchUnit tests, analyzers in the build file |
| Coverage | A coverage gate on the change, `jacoco` or `coverage` config, thresholds in the build file |
| Build gate | CI on the change (`.github/workflows/`, `.gitlab-ci.yml`) |
| Agent rules | `AGENTS.md`, `CLAUDE.md`, agent rules files, existing CONSTRAINTS.md |

Reply with one table: gate, what it enforces, where it runs (edit, pre-commit, CI), and whether it blocks or warns. Then the gaps that matter for this task in one line each. Do not propose new rules here. A new rule comes only through pstack `correct`'s prove-it loop: a real past mistake, a check that fails on it, then the rule.

## Greenfield: write CONSTRAINTS.md once [bl: strong]

1. **Detect first.** Stack, test runner, linters, CI, agent harness. Report in two lines, ask only what is left.
2. **One `grill-me` round, four questions, each with a recommended answer and a default** so "I don't know" still gives a working file:
   - Which dimensions beyond the floor? (default: coverage on changed lines and security scanning)
   - Block or warn when a check fails mid-task? (default: block on the floor, warn on the rest for two weeks)
   - Target numbers, or measure today and hold the line? (default: measure and hold)
   - Slowest check before work is handed back? (default: about 90 seconds at task end, unlimited in CI)
3. **Write the file** at the repo root, using the template below. Every number has a reason.
4. **Install what each dimension needs.** Prefer the de facto tool the ecosystem already uses over a hand-rolled checker. A row with a number and no command is an aspiration, not a constraint.
5. **Point at it.** One line in `AGENTS.md` or `CLAUDE.md`: read CONSTRAINTS.md before writing code, and do not weaken it to make a change pass.

```markdown
# Constraints

Last reviewed: <date> by <owner>

## Floor (always on, no setup)
- No new suppression comments (checker-disable, ignore pragmas) without a reason and an expiry.
- No unimplemented stubs, empty catch blocks, or TODOs standing in for the work.
- No skipped or deleted tests without a reason in the commit message.
- No secrets in source.
- This file is not weakened in the same change that was failing it.

## Enforced
| Rule | Why this number | Checked by (command) | Runs at | Origin |
|---|---|---|---|---|

## Measured, not yet enforced (ratchet: must not get worse)
| Metric | Today | Direction |
|---|---|---|

## Exceptions
| ID | Rule | Path | Reason | Owner | Expires |
|---|---|---|---|---|---|
```

`Origin` is `declared` for rules chosen here, or `mined: <evidence link>` for rules added from the mistake-miner review.

Sane defaults when there is no opinion: changed-line coverage at least 80 percent, project coverage must not fall, no high-or-above dependency findings, exceptions live 90 days. Measure and hold beats an invented number the codebase fails today.

## Check kinds [bl: invariant]

Rank each check by one question: can the agent make it pass with code that does not work?

- **External**: a vulnerability database, a real browser, an accessibility standard. The agent cannot argue with it.
- **Project**: lint rules, layer boundaries. A human owns the config.
- **Suite**: the project's own tests. Most useful, and the only circular kind.

A bar made only of suite checks proves the agent agrees with itself. Keep at least one external or project check.

## Guard the bar [bl: invariant]

On every diff, look for the cheapest roads to green. Tightening is silent. Loosening is loud: name it in the review, never fix it quietly.

1. **A threshold moved.** A coverage number, alarm threshold, timeout, or severity lowered; a check moved out of the blocking stage; a CONSTRAINTS.md or lint config row weakened.
2. **A test got easier.** A skip or disabled annotation added, a test file deleted, assertions removed from a test that stayed, an expected value edited to match the new output.
3. **A checker got silenced.** A new suppression comment or config exclusion (coverage ignore, lint disable, static-analysis suppress).
4. **Work is unfinished.** A stub that throws, an empty catch, a TODO where the logic should be.
5. **An exception appeared.** A new allowlist entry or exceptions row nobody discussed, or one with no owner or expiry.

A fresh-context code reviewer runs these five on every change. A worker that hits one in its own diff stops and reports it instead of shipping.

## Red flags

- A new rules file in a brownfield package that duplicates its existing gates.
- A rule with no command behind it, or a number with no reason.
- A threshold relaxed in the same change that was failing it.
- An exception with no owner or an expiry more than a year out.
- A rule added without a past mistake it was proved on (that is a guess, not a mined rule).

## Verification

- [ ] Mode stated (brownfield, greenfield, or non-interactive floor).
- [ ] Brownfield: a gate table from real files, no new file.
- [ ] Greenfield: CONSTRAINTS.md with a reason and a command per row, origin per rule, at least one non-suite check, pointer in AGENTS.md or CLAUDE.md.
- [ ] The guard-the-bar checks ran on the diff and any loosening is named.

## Sources

- [addyosmani/agent-skills `constraint-driven-development`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/constraint-driven-development/SKILL.md) at `aeac41d`, MIT (`THIRD_PARTY_LICENSE-agent-skills`): CONSTRAINTS.md written once, the four-question intake with defaults, floor only in non-interactive runs, external / project / suite check kinds, the five guard-the-bar moves, ratchets, sane defaults. Rewritten, not copied. The brownfield detect-only mode is ours.
- [cursor/plugins pstack `correct`](https://github.com/cursor/plugins/tree/9580c23696ac8d7abed0c166f2f9918800afb819/pstack/skills/correct), MIT: new rules only after a check fails on a real past mistake, and the rule-to-enforcer table.
- Compared, no quality-bar skill (safety rails or verify-before-done only): gstack `careful` / `guard` / `freeze`, mattpocock/skills `git-guardrails-claude-code`, superpowers `verification-before-completion`, compound-engineering `ce-compound`, kunchenguid/no-mistakes.
- Short `[bl: invariant|strong|soft]` tags mark bitter-lesson risk; the analysis is in `BITTER_LESSON.md`.
