# GeorgeCodingSkills

Agent skills for coding work, each tagged by how likely it is to survive better models.

## Skills

| Skill | What it does |
|---|---|
| [`start-playbook`](skills/start-playbook/SKILL.md) | Start here. Maps the task to the right skill in lifecycle order (define, plan, build, verify, review, ship) and sets the always-on behaviors. Adapted from [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) `using-agent-skills`. |
| [`grill-me`](skills/grill-me/SKILL.md) | Interviews you until the intent, or every decision in a plan, is settled. Rounds with a recommended answer per question, facts looked up by the agent, explicit-yes gate. Combines [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) `interview-me` and [mattpocock/skills](https://github.com/mattpocock/skills) `grilling`. |
| [`principles`](skills/principles/SKILL.md) | Index of the evaluation methods for a claim, a number, a design, or scaffolding. |
| [`principles-scientific-method`](skills/principles-scientific-method/SKILL.md) | Hypothesis, prediction before acting, one change at a time, baseline, try to prove yourself wrong, confidence labels. |
| [`principles-first-principles`](skills/principles-first-principles/SKILL.md) | Fermi bounds from cores, bandwidth, memory, latency, queueing and OS limits, compared against the claim. |
| [`principles-bitter-lesson`](skills/principles-bitter-lesson/SKILL.md) | Sorts agent scaffolding by Sutton's [bitter lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html) ("would we still need this if the model were 10x smarter?") and ships the tag checker. Formerly `bitter-lesson-check`. |

## Bitter-lesson tags

Each skill marks its parts with a short inline tag:

- `[bl: invariant]`: a rule of the game (a feedback signal, an eval, or a real invariant). Always follow.
- `[bl: strong]`: follow it. It should become a hard check or a learned example over time.
- `[bl: soft]`: guidance for weaker models. A strong model may skip it. Candidate to expire.

A `BITTER_LESSON.md` next to each `SKILL.md` holds the full analysis (why, review date, delete-when, ablation result). Agents only read it when evaluating skills, not at session start.

```sh
python3 skills/principles-bitter-lesson/scripts/bitter_lesson.py check  --root .   # every tagged item has a row, and the reverse
python3 skills/principles-bitter-lesson/scripts/bitter_lesson.py report --root .   # the cleanup list
python3 skills/principles-bitter-lesson/scripts/bitter_lesson.py ablate --root .   # soft items to test with and without
```

Principle skills (`principles*`) are durable reasoning methods and carry no tags.

Expire by evidence: run the skill's eval with and without a soft item on a new model. If nothing changes, delete it. A skill whose items have all expired goes away.

## Install

Copy a skill folder into your agent's skills directory (for example `~/.claude/skills/`).

## License

MIT. `start-playbook` and `grill-me` adapt MIT-licensed text by Addy Osmani (`THIRD_PARTY_LICENSE-agent-skills`). `grill-me` also adapts MIT-licensed text by Matt Pocock (`THIRD_PARTY_LICENSE-mattpocock-skills`).
