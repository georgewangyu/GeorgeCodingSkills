---
name: git-workflow-and-versioning
description: "Keep every change reviewable, reversible and safe to publish. Atomic commits whose message says why, separate concerns, size the change, save points before risky steps then squash before review, one change per checkout (no git worktrees; clone again if parallel work truly needs isolation), push safety (fetch first, rebase on the fresh base, never force a shared branch), guardrail hooks for destructive commands, redact before anything leaves the machine, an explicit finish step with checks green, semver and a changelog for anything with consumers, and bisect to find the commit that broke it. Use when committing, splitting a messy tree into commits, branching, rebasing, pushing, opening or finishing a pull request, cutting a release or tag, writing a changelog, or hunting the commit that introduced a bug."
---

# Git workflow and versioning

Git is the safety net. Commits are save points, history is documentation, and every push is a publication. Agents write code fast, so the discipline that keeps changes small, reviewable and reversible matters more, not less.

Standing rules from your user or organization (who may merge, what needs review, when to publish, which repos allow a direct push, which identity to use) win over this skill. It does not decide them; follow them.

## When to use

- Committing, or turning a messy working tree into clean commits.
- Branching, rebasing, pushing, opening or finishing a pull request.
- Cutting a release, a tag or a changelog entry.
- Finding the commit that introduced a bug.

## 1. Commits that do one thing [bl: strong]

1. **Atomic.** One logical change per commit: it builds, its tests pass, and it can be reverted alone.
2. **Separate concerns.** Formatting, refactoring and behavior are different commits, ideally different pull requests. A rename in passing is fine at the reviewer's discretion; a reformat of the file is not.
3. **Size it.** About 100 changed lines is easy to review, 300 is fine for one logical change, past about 1000 split it. Splitting comes before review, not after.
4. **The message says why.** The subject names the outcome (what is now fixed or possible), not the files touched. The body, when needed, gives the motivation and the trade-off a reader cannot get from the diff. Follow the repo's convention first; else the recent log; else conventional commits (`feat`, `fix`, `refactor`, `test`, `docs`, `chore`).
   - Bad: `update auth.ts`, `fix stuff`, `address review comments`.
   - Good: `fix: reject expired tokens before the cache lookup`.
5. **Stage named files only.** Never `git add -A` or `git add .`. Build output, symlinks, local config and scratch files ride in that way. Read `git diff --staged` before every commit: no secrets, no generated files the repo does not expect, nothing outside the change.
6. **Credit carried work.** When a commit carries someone else's work (a ported patch, a cherry-pick from their branch), keep their name: a `Co-authored-by:` trailer copied exactly from their commits, never a guessed email.
7. **Commit when asked.** If your user's rules say commit only on request, finish the change and say it is ready.

## 2. Save points, then squash before review [bl: strong]

- **Commit before a risky step**: a rebase, a large refactor, a codemod, a dependency bump, a conflict resolution. A save point means you never lose more than one step of work.
- **Step back safely.** Prefer `git reset --keep <save>` (it refuses to discard uncommitted changes) or `git revert`. `git reset --hard` destroys uncommitted work; only use it on your own work after `git status` shows nothing you would lose.
- **Squash before review.** Save points and fixups are for you, not the reviewer. Before review, reshape the branch into what the review tool expects: one commit per change for snapshot-style review, or a short stack of atomic commits. Without an interactive rebase, `git reset --soft <base>` then recommit in groups.
- Squash the noise, not the narrative: two genuinely separate logical changes stay two commits, or better, two pull requests.

## 3. One change per checkout, no worktrees [bl: strong]

- **Short-lived branches** off the default branch, merged within days. A long-lived branch accumulates merge risk every day it lives. Prefer a flag over a branch for unfinished work.
- **One change per checkout.** Two agents, or an agent and a human, never share one working tree.
- **No git worktrees.** If parallel work truly needs isolation, clone the repo again into a fresh directory, work and push from there, and delete the clone once the change lands. A clone has no shared index, no shared stash and no worktree bookkeeping to clean up.
- **A dirty checkout that is not yours** is not yours to clean. Do not stash, reset or commit someone else's edits to get a clean tree; clone fresh instead.

## 4. Push safety [bl: invariant]

The point of the tool is to not lose anyone's work. Refuse and surface a problem rather than attempt a clever recovery.

1. **Fetch first.** Rebase onto the freshly fetched remote base (`origin/<base>`), never a stale local copy of it.
2. **Never force-push a shared branch.** A rejected push means the remote moved: fetch, look at what arrived, integrate it. Do not force over it.
3. **On your own branch**, a rewrite after rebase uses `--force-with-lease=<branch>:<sha>` anchored to the commit you last observed on the remote, not one read just before pushing (that check always passes and protects nothing). Never a bare `--force`.
4. **Never amend or rebase commits that may already be on a remote** others use. Fetch before deciding a commit is local-only; automation may have pushed it.
5. **Do not add remotes or push repo data somewhere new** without the user's direction.

## 5. Guardrails as hooks, not prose [bl: strong]

A rule in prose is skipped under pressure; a hook is not. Set these up once per machine or repo:

- **Agent hook** (a pre-tool hook in the coding agent): read the command, match destructive patterns, exit non-zero with a message saying the user has not authorized it. Block at least: `push --force` and `-f` (also `+refspec`), `reset --hard`, `clean -f`, `branch -D`, `checkout .` and `restore .`, `stash drop` and `stash clear`, `add -A` and `add .`. Add plain `push` to protected branches if your rules say a human publishes.
- **Git hooks**: `pre-commit` for lint and a secret scan on the staged diff; `pre-push` for the redaction scan below and a refusal to push to protected branches.
- Test the hook with a sample command; it must exit non-zero and print why. The way past a block is the user running the command, or approving that one action, never the agent editing the hook.

## 6. Redact before publish [bl: invariant]

Everything that leaves the machine is published: commit messages, branch and tag names, PR titles and bodies, review comments, release notes, uploaded artifacts.

- **Home paths** become `~` (`/Users/<name>`, `/home/<name>`, `C:\Users\<name>`).
- **Secrets** never: keys, tokens, passwords, connection strings, cookies. A secret that reached a remote is compromised: rotate it first, then purge history.
- **Internal names** never go to a public repo: hostnames, account ids, ticket ids, internal tool and team names, private package names. Keep a marker list outside the repo and scan with it in `pre-push`.
- **Identity**: the author and committer must be the identity that repo expects (personal for personal repos, work for work repos). Check `git config user.email` in the repo before the first commit.
- Redact at one boundary that every publishing path goes through (the pre-push scan plus the PR body), not at each call site; a new path otherwise leaks.

## 7. Finish deliberately [bl: strong]

1. **Integrate, then verify.** Rebase or merge the fresh base first, then run the full checks on that integrated tree. A green run only proves the tree it ran on.
2. **Write the change summary**: what changed, what you deliberately did not touch, open concerns. The "did not touch" list shows scope discipline.
3. **Choose the finish**, or ask if your user's rules do not decide it:
   - merge locally into the base, re-run checks on the merged result, delete the branch;
   - push and open a pull request;
   - keep the branch as is;
   - discard, only when the user asks in so many words and confirms after seeing the branch and commits that will be lost.
4. **Pull request body**: the smallest view that makes the point (pseudocode, a call tree, a file tree, a diff sketch), evidence before and after (a failing then passing test, output, a screenshot for UI), and merge danger: one-way or two-way door, and the blast radius.
5. **Clean up** after it lands: delete the merged branch and any temporary clone. Not before; review fixes happen there.

## 8. Versions and changelogs (anything with consumers) [bl: strong]

- **Semver** once anyone depends on you: MAJOR breaks consumers, MINOR adds compatibly, PATCH fixes compatibly. Anything a consumer can observe is part of the contract; when unsure whether a change breaks, treat it as major.
- **Tag the release** (`git tag -a v1.4.0 -m "Release 1.4.0"`, then push the tag) and derive the version from the tag, so the artifact, tag and changelog cannot disagree.
- **Changelog for humans**, newest first, grouped Added, Changed, Fixed, Deprecated, Removed, Security, phrased by user impact. Write the entry in the same change, not at release time. Breaking changes carry a migration note.
- Apps deployed by a pipeline with no outside consumers usually need none of this; the deploy history is the version.

## 9. Git for debugging [bl: soft]

- `git bisect start; git bisect bad; git bisect good <sha>`, then `git bisect run <test-cmd>` to find the first bad commit automatically. Atomic commits are what make this work.
- `git log -S'<string>'` (when a string appeared or vanished) and `-G'<regex>'`, `git log --grep`, `git blame -w`, `git log -p --follow <file>`.
- Hand the result to `debugging-and-error-recovery`: the first bad commit is evidence, not the root cause.

## Red flags

- Large uncommitted changes piling up, or one commit mixing a reformat with a behavior change.
- Messages like `fix`, `wip`, `misc` in the history under review.
- `git add -A`, a staged build artifact, symlink or `.env`.
- Two sessions in one working tree, or a worktree created for parallel work.
- A force-push to a shared branch, or a lease read just before the push.
- A home path, token or internal hostname in a commit message or PR body.
- "Tests passed earlier" as evidence for the merged tree.
- A breaking change under a patch bump, or a release with no tag or changelog entry.

## Verification

Per commit: one logical change; message says why; staged diff read; no secrets or artifacts; right identity.
Per push: fetched and rebased on the fresh base; no force on a shared branch; redaction scan clean.
Per finish: checks green on the integrated tree; summary written; the finish chosen by the user's rules or the user.
Per release: bump matches the change; tagged; changelog entry by impact.

## Related

- `incremental-implementation` (one slice per commit or PR), `code-review-and-quality` (sizing and splitting a change, receiving review), `security-and-hardening` (secrets), `debugging-and-error-recovery` (after bisect), `quality-constraints` (hooks as gates).

## Sources

- [addyosmani/agent-skills `git-workflow-and-versioning`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/git-workflow-and-versioning/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): atomic commits, why-first messages, separate concerns, sizing, save points, change summaries, pre-commit hygiene, bisect, semver, tags, changelog. Its worktree section is deliberately not adopted (clone again instead), and its "do not squash" advice is narrowed to save points versus logical commits.
- [obra/superpowers `finishing-a-development-branch`](https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/finishing-a-development-branch/SKILL.md) at `2ce8aa1`, MIT (see `THIRD_PARTY_LICENSE-superpowers`): verify then choose merge, PR, keep or discard; discard only on request with confirmation; a rejected push is not a reason to force.
- [mattpocock/skills `git-guardrails-claude-code`](https://github.com/mattpocock/skills/blob/5beb35b040e3de97e54fa9e386d7e42d15c295c2/skills/misc/git-guardrails-claude-code/SKILL.md) and [`pr`](https://github.com/mattpocock/skills/blob/5beb35b040e3de97e54fa9e386d7e42d15c295c2/skills/engineering/pr/SKILL.md) at `5beb35b`, MIT (see `THIRD_PARTY_LICENSE-mattpocock-skills`): guardrails as hooks, the PR body (summary view, evidence, merge danger).
- [EveryInc/compound-engineering-plugin `ce-commit`](https://github.com/EveryInc/compound-engineering-plugin/blob/6500180a95ee132d612c58ceb6cb2fffe40766e6/skills/ce-commit/SKILL.md) at `6500180`, MIT (see `THIRD_PARTY_LICENSE-compound-engineering-plugin`): outcome-first subjects, named-file staging, co-author credit.
- [garrytan/gstack `ship`](https://github.com/garrytan/gstack/blob/576839cb8ce73c08a05a71efa91e80ca19fa630b/ship/SKILL.md) at `576839c`, MIT (see `THIRD_PARTY_LICENSE-gstack`): merge the base before verifying, bisectable commits, never force-push.
- [kunchenguid/no-mistakes](https://github.com/kunchenguid/no-mistakes/tree/50ea98d8157e49a003d65e31f3807a8f29077f9a/.agents/skills) `branch-sync-and-push-safety` and `pr-publication-safety` at `50ea98d`, MIT: refuse over clever recovery, rebase on the fresh remote, a lease anchored to the observed head, redaction at one publication boundary. Ideas only, no text copied.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
