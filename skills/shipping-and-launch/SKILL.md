---
name: shipping-and-launch
description: "Launch a change so it is reversible, observable and incremental. A short pre-launch checklist, a rollback path that still works after the change (not just before it), the alarms and dashboards live before release, testing at the target load and not below it, measurable rollback criteria per stage, a worst case that includes losing one zone, feature flags that separate deploy from release, staged rollout against a baseline, and a watched first hour. Use when preparing a production deploy or launch, writing or reviewing a rollout or change plan, a capacity or config change, enabling a feature flag for users, or asking what must be in place before shipping."
---

# Shipping and launch

A launch is done when the change is in front of users, the evidence says it is healthy, and you can still undo it. Every launch should be reversible, observable and incremental.

Your organization's change process (approvals, change windows, two-person review on production) wins over this skill. Follow it.

## When to use

- First production deploy of a feature, or a significant behavior change.
- A capacity, config, schema or infrastructure change in production.
- Turning a flag on for users, or opening a beta.
- Writing or reviewing a rollout or change plan.

## 1. Pre-launch checklist [bl: strong]

Short on purpose. The quality gates already ran in CI (`ci-cd-and-automation`); this is what CI cannot check.

- [ ] **Rollback written down**, with the trigger, the steps and how long it takes (section 2).
- [ ] **Monitoring live**: the alarms, dashboards and log queries this launch relies on are deployed and pointing at the right resources, not just merged (section 3).
- [ ] **Tested at the target**: load, size or density at the end state and at the worst case, not only at today's level (section 4).
- [ ] **Rollback criteria are numbers** per stage (section 5).
- [ ] **Worst case covered**, including losing one zone or one dependency at peak (section 6).
- [ ] **Flag or kill switch** for anything users see, owned and with an expiry.
- [ ] **Config is code**: nothing done from memory in a console.
- [ ] **Docs and runbooks** updated (`documentation-and-adrs`); the people who will be paged know it is coming.
- [ ] **Not into a freeze**: no Friday-evening launches, and respect any error budget or change window.

## 2. A rollback that still works after the change [bl: invariant]

Write the rollback before the first deploy, and check it against the state **after** the change, not before it:

- **Will the old version still fit?** A change that removes capacity, migrates data or drops a field can make the old version unrunnable. If rolling back needs capacity or data you just removed, the first rollback step is "restore N units", with its lead time.
- **Every step of a multi-step plan has its own rollback.** A plan whose rollback only works from the start is not reversible halfway.
- **Data changes have a tested down path** (`deprecation-and-migration`, expand and contract).
- Write the time to roll back: flag off in a minute, redeploy in minutes, data restore longer.

```markdown
## Rollback
Trigger: <metric> above <threshold> for <duration> at any stage
Steps: 1. <restore capacity or flag off>  2. <redeploy previous version>  3. <verify>
Time to roll back: <minutes>
Still valid after the final step? yes, because <reason>
```

## 3. Alarms and their changes live before release [bl: invariant]

- The alarms, thresholds and dashboards the plan depends on are **deployed** before the stage that needs them. A pending change to an alarm is not an alarm.
- Check the thresholds against the **new** normal. A change that raises steady-state load can make an existing alarm fire at ordinary peak, or a new one never fire.
- List the alarms you **expect** to fire during the change, so a real one is not dismissed as expected.

## 4. Test at the target, not below it [bl: strong]

- Test at the end-state load, size or density, and at the worst case (section 6). Data that stops below the target does not support the target.
- A "qualified limit" needs a link to the run that qualified it.
- If you cannot test at the target, cap the change at the tested range and say so.

## 5. Measurable rollback criteria [bl: strong]

Each stage gets a metric, a threshold, a duration and an action. Compare against a baseline taken before the change.

| Signal | Advance | Hold and look | Roll back |
|---|---|---|---|
| Error rate | Within 10% of baseline | 10% to 2x | Over 2x |
| Latency p99 | Within 20% | 20% to 50% | Over 50% |
| New error types | None | Rare | Spreading |
| Saturation (CPU, queue age, capacity left) | Under plan | Near the limit | Over the limit |

- Alert on change from baseline, not on absolutes. Ignore a single blip; act when it persists across two checks.
- "More than a few" is not a criterion. A unit, a number and a window are.

## 6. Worst case includes losing a zone [bl: strong]

For any capacity or density change, compute the load per unit with one zone (or one replica group, or one dependency) gone, at daily peak. If that exceeds the limit, the plan is not safe even if the steady state is. Add it as its own row in the plan's worst-case section.

## 7. Flags and staged rollout [bl: strong]

- **Deploy dark, release with a flag**: code in production with the flag off, then team, then a small percent, then wider.
- **Stages**: staging, production with flag off, internal users, canary (about 5%), 25, 50, 100. Bake at each stage long enough to see a daily peak.
- **Canary against a baseline**: compare canary to the rest, not to a guess.
- **Small steps for big moves.** A jump that doubles load per unit gets an intermediate stage.
- **Flags have an owner and an expiry.** Remove the flag and the dead path soon after full rollout; test both states while it exists.

## 8. The first hour [bl: strong]

Someone watches. Health check, error rate, latency, the critical flow end to end, logs flowing. Confirm the rollback is still ready. A rollback triggered by an alarm is evidence: investigate it before retrying (`debugging-and-error-recovery`).

## Red flags

- No written rollback, or one that assumes capacity or data the change removed.
- An alarm the plan relies on still in review.
- A target load above the highest tested point.
- Rollback criteria in words ("if it looks bad"), or no worst case with a zone lost.
- A big-bang release, a flag with no owner, nobody watching the first hour.
- Retrying a rolled-back deploy without knowing why it rolled back.

## Verification

- [ ] Rollback written, valid from every step and from the end state, with time to roll back.
- [ ] Alarms and dashboards deployed and thresholds checked against the new normal.
- [ ] Test evidence at the target and the worst case, linked.
- [ ] Every stage has numeric criteria and an action.
- [ ] Zone-loss case computed at peak.
- [ ] First hour watched; flag cleanup scheduled.

## Sources

- [addyosmani/agent-skills `shipping-and-launch`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/shipping-and-launch/SKILL.md) (MIT, see `THIRD_PARTY_LICENSE-agent-skills`): pre-launch checklist, flag lifecycle, staged rollout, decision thresholds, rollback plan, error budget, first-hour checks.
- [garrytan/gstack](https://github.com/garrytan/gstack) `ship`, `land-and-deploy`, `canary` and `landing-report` (see `THIRD_PARTY_LICENSE-gstack`): baseline before deploy, alert on changes, persist across two checks, never hide missing evidence, an approved rollback.
- [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin) `ce-commit-push-pr` (see `THIRD_PARTY_LICENSE-compound-engineering-plugin`): ship as one explicit, reviewable step.
- Sections 2 to 6 are lessons from the author's review of a real capacity release plan: the rollback did not fit after capacity was removed, alarm changes were not live, tests stopped below the target, criteria had no units, and no zone-loss row.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
