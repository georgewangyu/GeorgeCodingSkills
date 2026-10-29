---
name: interfaces
description: Design module seams, APIs, config keys, schemas and other contracts so they are deep, hard to misuse and safe to roll out and back. Contract first, every observable behavior is a contract (Hyrum), behavior declared by the injected implementation and never inferred from it, validate at boundaries and trust inside, illegal states unrepresentable, additive changes only while old and new versions run together. Use when the user says "design the interface", "where should the seam go", "what should the API look like", "add a config key", "change this schema", "is this a breaking change", or when a plan slice produces something other slices consume.
---

# Interfaces

A good interface is small, deep and honest: little to learn, a lot behind it, and nothing a caller can observe that you are not willing to keep.

## When to use

- A new API, RPC operation, schema, config key, metric name, CLI flag or module seam.
- Changing any of those where something else already depends on it.
- A `plan` slice whose "produces" line other slices consume.
- A review asks "why does this class guess what to do?" or "is this backward compatible?".

Not for:

- Code with no caller outside its own file. Write it plainly.
- Which approach to take at all: `brainstorm`. What we are building: `spec`. Exact signatures of an existing library: read its source or docs first.

## Words to use [bl: strong]

Use these terms exactly, so reviews and briefs share one vocabulary (from Matt Pocock `codebase-design`).

- **Module:** anything with an interface and an implementation, from a function to a package.
- **Interface:** everything a caller must know to use it: signature, invariants, ordering, error modes, required config, latency. Not only the type.
- **Depth:** behavior per unit of interface learned. Deep is a small interface over a lot of behavior. Shallow is an interface as complex as its body.
- **Seam:** where an interface lives, the place behavior can change without editing callers.
- **Adapter:** a concrete implementation plugged in at a seam.

## Rules

1. **Contract first.** Write the interface before the body: types, error modes, invariants, ordering, timeouts, config keys and their defaults. In a series it goes in the plan's "produces" line so later slices can rely on it. [bl: strong]
2. **Every observable behavior is a contract (Hyrum's law).** Log lines, metric names and dimensions, error text, ordering and timing all get depended on. Metric names and log lines feed alerts, dashboards and log scans, so renaming one, or changing what it counts, is a breaking change. Search for the alerts and dashboards that read it before the change, and list them in the PR. [bl: invariant]
3. **Deep, not shallow.** Apply the deletion test: if deleting the module just moves its code into callers, it was a pass-through. One adapter is a hypothetical seam, two adapters (production and test, or two modes) make it real. Do not add a seam for a variation that does not exist yet. [bl: strong]
4. **Behavior is declared by the implementation, never inferred from it.** Inject the strategy at the composition root and let it say what it does. No `if (impl == X)`, no mode flag that shadows the injected object, no type-switch dispatch. New behavior is a new implementation of the same contract, not a branch in the shared code. [bl: strong]
5. **Validate at boundaries, trust inside.** CLI args, config files, network input, external and third-party responses, and anything an agent reads from mail or a web page get parsed into domain types once, at the edge. Inside, trust the types: no re-validation, no defensive null checks deep in call chains. Do not re-export wire, storage or framework types through the public surface. Your own stored data counts as a boundary when an older schema or a buggy version may have written it: a checkpoint read back after a deploy or rollback is parsed and checked like external input. [bl: strong]
6. **Make illegal states unrepresentable.** A state machine or enum over two booleans that must stay in sync. A distinct type over a bare string when two strings mean different things (`OrderId` vs `UserId`). Exhaust variants so a new one fails compile. Derive types from the authoritative schema (the IDL, the migration, the config class) rather than a hand-kept twin. [bl: strong]
7. **One error strategy per surface.** Every operation on a surface fails the same way. For RPC services that means modeled exception types in the IDL (HTTP APIs: one status mapping and one error body); mark which ones are retryable and model a new failure as a new exception, not new message text. Never mix throw, return null and return an error object. [bl: strong]
8. **Additive changes only, because old and new run together.** During a rollout, and after a rollback, both versions serve and read the same data and config. So add optional fields and new keys whose default is today's behavior; never change a field's type or meaning in place; old readers must ignore what they do not know. A new path ships beside the old one so rollback is config, not a revert. A removal is expand, migrate callers, then contract. [bl: invariant]
9. **Evolve by addition, not by a second version.** The one-version rule: a new major version of a library or API is a fork every consumer must migrate to, and dependency trees holding both conflict. Keep extending the current major version additively; cut a new one only for a change that cannot be additive, with the migration of callers planned. [bl: strong]
10. **Pick the update shape on purpose.** A partial update (PATCH style) fits when fields are independent. When fields are coupled, or the caller holds the whole object, a full replace with a version or condition check is clearer and safer. Neither is the default. [bl: soft]
11. **State-changing calls say what a retry does.** Either idempotent (key derived from the intent, claimed atomically, same key with a different payload fails loudly) or documented unsafe to retry. Every remote call has three outcomes: success, failure and unknown. A timeout is unknown, never failure. [bl: invariant]
12. **Timeouts and deadlines are part of the contract.** State the caller's timeout, the server's expected latency and what happens at expiry. Example: a client with a fixed 200 ms timeout sees a server p100 of 500 ms as errors, even when nothing is lost. A latency change is an interface change for every caller with a fixed timeout. [bl: strong]
13. **State ordering and concurrency guarantees.** Say what order a caller can rely on (per key, per partition, none), whether calls may run concurrently, and which operations are safe to repeat out of order. A system that keeps order per key but not across keys has a latent bug in every caller that assumes more. [bl: strong]
14. **Async and event contracts are contracts too.** For datagrams, queues and events, write down delivery (at most once, at least once), duplicates, ordering, the payload schema and how it evolves, and what the consumer does with a message it does not understand. Lost or late is part of the contract for UDP. [bl: strong]
15. **Say what a caller gets under overload.** Backpressure is behavior: throttle with a retryable error, queue with a bound, shed, or block. Name it, name the limit, and name the signal (error type, metric) so callers can back off instead of retrying harder. [bl: strong]
16. **Design it twice for one-way doors.** For a public API, a persisted format or a cross-team contract, sketch two or three radically different interfaces (fewest entry points, most flexible, best for the common caller), compare on depth, locality and seam placement, then pick. Subagents can draft them in parallel. [bl: soft]

Rules 12 to 15 are what upstream api-and-interface-design leaves out; they are where our incidents come from.

## Schema and IDL quirks [bl: strong]

Generated code and schema languages each have rules that look like ordinary types but are not. Before adding a shape, read a neighboring shape in the same schema and copy its form. Check, for your IDL: whether enums and unions are real shapes or a base type plus an annotation, which version introduced unions and where they may appear, and whether a declared default is applied at runtime or only validated. When in doubt, model the field optional and apply the default in code. Where the IDL has no union, model a variant in code as a sealed interface, or as an enum plus a payload.

## Domain names [bl: soft]

- Name modules and types after the domain concept the project already uses, not after a step in a pipeline (loader, processor, handler). Execution order is not ownership.
- When a term is fuzzy or two names mean one thing, settle it in a glossary or the design doc's terminology section and use it everywhere.
- Record a decision (an ADR) only when it is hard to reverse, surprising without context, and the result of a real tradeoff.

## Verify

- [ ] The interface is written down (types, errors, invariants, config keys with defaults) before the body.
- [ ] No branch in shared code depends on which implementation is injected.
- [ ] External input is parsed once at the edge; internal code does not re-validate.
- [ ] New fields and keys are optional with today's behavior as the default; nothing changed type or meaning in place.
- [ ] Both versions can run at once and rollback is a config change, or the plan says why not.
- [ ] Renamed, removed or redefined metrics, log lines and error text are listed with the alarms and dashboards that read them.
- [ ] Timeouts, ordering, concurrency, delivery and overload behavior are written down for every remote or async surface.
- [ ] Stored data an older version may have written is parsed like external input.
- [ ] Tests exercise the interface, not internals behind it.

## Red flags

- `isX() { return impl != DEFAULT; }` or any identity check that decides behavior.
- A second boolean that must stay in sync with the first.
- The same validation at three layers, or none at the edge.
- A schema default relied on without checking that it is applied.
- A config key whose default changes today's behavior in the same PR that adds it.
- A metric or log line renamed, or its meaning changed, with no search for the alerts or dashboards that read it.
- A timeout treated as a failure and retried as if nothing happened.
- A latency or ordering change with no note to callers.
- A seam with one adapter and no second one planned.

## Worked example

A service that runs many independent workers and handles a fatal error in one of them. Today a fatal error halts the whole host; the goal is to restart only the failing worker.

- **Before.** A `FatalHelper` held a `Listener` and decided the policy from its identity: `isolatesWorker() { return listener != ABORT; }`. Review: this guesses behavior from which object it holds; extract the handling into a strategy and inject it.
- **After (rule 4).** Each listener declares its behavior, `interface Listener { onWorkerFailed(id, reason, cause); boolean isolatesWorker(); }`. The old mode injects `ABORT` (halt the host, as today), tests inject a log-only listener, the new mode later injects its monitor. The helper only classifies, reports and throws.
- **Rule 8.** The first PR landed the helper with no callers. The second routed every worker-scoped fatal site through it, still set to `ABORT`: same log line, same counters, rollback-safe. Only a later PR changed behavior, behind a feature flag, so rollback is config.
- **Rule 2.** Routing kept the existing log line and counter names, so nothing that reads them had to change.
- **Rule 2, a different change.** A PR that made a health check trip at a percentage of degraded workers also changed what the existing degraded-workers count metric means. The alerts that read it were searched for first and listed in the PR before it went out.

## Related

- `plan` (interfaces per slice), `spec` (design contract for a series).
- pstack `principle-boundary-discipline` and `principle-type-system-discipline`: the short principles behind rules 5 and 6.

## Sources

- [addyosmani/agent-skills `api-and-interface-design`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/api-and-interface-design/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): Hyrum's law, the one-version rule, contract first, consistent errors, validate at boundaries, addition over modification, idempotency keys, the checklist.
- [mattpocock/skills `codebase-design`](https://github.com/mattpocock/skills/blob/5beb35b040e3de97e54fa9e386d7e42d15c295c2/skills/engineering/codebase-design/SKILL.md) (with `DEEPENING.md` and `DESIGN-IT-TWICE.md`) and [`domain-modeling`](https://github.com/mattpocock/skills/blob/5beb35b040e3de97e54fa9e386d7e42d15c295c2/skills/engineering/domain-modeling/SKILL.md) at `5beb35b`, MIT (see `THIRD_PARTY_LICENSE-mattpocock-skills`): the vocabulary, the deletion test, one adapter vs two, design it twice, glossary terms, ADRs only when hard to reverse, surprising and a real tradeoff.
- [cursor/plugins pstack](https://github.com/cursor/plugins/tree/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills) `principle-boundary-discipline`, `principle-type-system-discipline` and `principle-model-the-domain` at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): boundaries, branded types, parse at the edge, state machines over booleans, ownership over execution order.
- [garrytan/gstack `plan-devex-review`](https://github.com/garrytan/gstack/blob/576839cb8ce73c08a05a71efa91e80ca19fa630b/plan-devex-review/SKILL.md) at `576839c`, MIT (see `THIRD_PARTY_LICENSE-gstack`): compared, the caller's experience as a review lens; not adopted as a separate pass.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
