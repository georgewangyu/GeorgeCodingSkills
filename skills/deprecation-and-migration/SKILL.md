---
name: deprecation-and-migration
description: "Retire old code and move its callers safely. Code is a liability, Hyrum's law makes removal hard, so decide first (still unique value, who depends on it, is there a replacement), choose advisory or compulsory, build and prove the replacement, announce with a migration guide, migrate callers yourself, then remove only at zero usage. For internal APIs with no outside users, migrate every caller and delete the old API in the same wave instead of keeping a compatibility layer. Includes expand and contract for schema changes and a worked rename with a one-release redirect. Use when replacing or sunsetting a system, API, library, flag, skill or command, consolidating duplicates, renaming something others call, or deciding whether to keep or remove legacy code."
---

# Deprecation and migration

Code is a liability, not an asset. Every line needs tests, patches, dependency bumps and a reader. Removing code that no longer earns its keep is engineering work, and so is moving its users safely.

Standing rules from your user or organization (who may break a contract, release cadence, who approves removal) win over this skill.

## When to use

- Replacing an old system, API, library, config key, command or skill with a new one.
- Renaming anything other code, scripts, docs or people call by name.
- Consolidating two implementations of the same thing.
- Deciding whether to keep, invest in, or remove legacy code nobody owns.
- Designing something new: ask "how would we remove this in three years?"

## 1. Decide before you deprecate [bl: strong]

1. **Does it still give unique value?** If yes, keep it and maintain it properly.
2. **Who depends on it?** Count callers from code search, usage metrics and logs, not memory. With enough users every observable behavior is depended on (Hyrum's law), including bugs and timing.
3. **Is there a replacement?** If not, build it first. Never deprecate into a void.
4. **What does each caller pay to move?** If you can automate it, do it.
5. **What does keeping it cost?** Security, upkeep, and the confusion of two ways to do one thing.

**Zombie code** (callers but no owner, failing tests nobody fixes, stale dependencies) gets an owner or a removal plan. It cannot stay in limbo.

## 2. Advisory or compulsory [bl: strong]

| Type | When | Mechanism |
|---|---|---|
| Advisory | The old path is stable and moving is optional | Warnings, docs, nudges. Callers move on their own time. |
| Compulsory | Security risk, it blocks progress, or upkeep is unsustainable | A removal date, migration tooling, and support. |

Default to advisory. A compulsory deadline without tooling is just an outage on a schedule.

## 3. The process [bl: strong]

1. **Build the replacement and prove it** in real use. It covers the critical cases of the old one.
2. **Announce in the place callers already look**: status, replacement, removal date or "advisory", reason, and a migration guide with concrete before and after.
3. **Migrate callers incrementally**, one at a time: find every touchpoint, switch it, verify behavior matches, remove the old reference. If you own the old thing, you do the migration (the churn rule). Do not announce and walk away.
4. **Remove only at zero usage**, verified by search plus metrics or logs. Remove the code, its tests, docs, config and the deprecation notice. Then the redirect.

## 4. Internal APIs: migrate callers, then delete [bl: strong]

When the old API has no outside users and the project can absorb a coordinated change, do not keep a compatibility layer. In one wave:

- Move every caller to the new API.
- Rewrite tests to assert the new contract. Delete tests that only protected the old implementation.
- Delete the old API.

A shim with no outside users is two ways to do one thing, forever. Keep a redirect only when callers you cannot edit exist (other repos, scripts, people's muscle memory), and give it an end date.

## 5. Patterns [bl: soft]

- **Strangler**: run old and new side by side, shift traffic 0, 10, 50, 100 percent, then remove the old.
- **Adapter**: keep the old interface, delegate to the new implementation, migrate the backend first.
- **Flag**: switch callers one at a time behind a flag; test both states.
- **Schema change, expand and contract** (the data is the one thing a code revert does not undo):
  1. Expand: add the new column, nullable. 2. Dual-write old and new. 3. Backfill in throttled batches. 4. Switch reads, keep writing both, bake. 5. Contract: stop writing the old column, then drop it in a later, separate deploy.
  - Adds are safe in any deploy. Drops and renames ship alone, after no code reads the old shape. Every migration has a tested down path.

## 6. Worked example: rename with a one-release redirect [bl: soft]

Renaming nine short skill names to clearer ones (`spec` to `spec-driven-development`, and so on), where people and other repos still type the old names:

1. **Build the replacement**: create each new skill with the full body.
2. **Move the callers you own in the same change**: the router, the playbook, the README, cross-links, allowlists. Search for every old name, including in prose.
3. **Leave a redirect at the old name**, body only: "Renamed to X. Load X. This redirect stays for one release, then is deleted." Mark it deprecated in frontmatter so tools can list it.
4. **Release.** Callers you could not edit see the redirect and move.
5. **Next release**: search again for the old names, fix stragglers, delete the redirects.

The redirect is the advisory notice and the migration guide in one line. Its end date is in the text, so it does not become zombie code.

## Red flags

- A deprecation with no replacement, or no migration guide.
- "Advisory" for years with no progress, or new features added to the deprecated path.
- Removing code without measuring usage first.
- A compatibility layer kept for callers that are all in your own repo.
- A redirect or shim with no end date.
- A schema change and the code that needs it in the same deploy; a column renamed in place; a migration with no down path.

## Verification

- [ ] Callers counted from search plus usage data, and the count is in the change description.
- [ ] Replacement proven in real use and covers the critical cases.
- [ ] Callers you own moved in the same wave; old tests rewritten to the new contract.
- [ ] Every redirect or shim has an end date.
- [ ] At removal: zero usage verified; code, tests, docs, config and notices gone.

## Sources

- [addyosmani/agent-skills `deprecation-and-migration`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/deprecation-and-migration/SKILL.md) (MIT, see `THIRD_PARTY_LICENSE-agent-skills`): code is a liability, Hyrum's law, the decision questions, advisory vs compulsory, the four steps, the churn rule, zombie code, patterns, expand and contract.
- [pstack](https://github.com/cursor/plugins/tree/main/pstack) `principle-migrate-callers-then-delete-legacy-apis` (cited, see `THIRD_PARTY_LICENSE-pstack`): migrate callers and delete in one wave, rewrite tests to the new contract. `figure-it-out` for very large migrations: frame a falsifiable done, riskiest unit first, a decision log.
- The worked example is from this repo's own skill renames.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
