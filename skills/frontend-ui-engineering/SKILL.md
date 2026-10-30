---
name: frontend-ui-engineering
description: "Build or change UI: an app's screens (dashboards, settings, editors) and the HTML artifacts an agent writes for a person to review. Read the project's own UI rules first, name the mode (operate, read, persuade), keep one owner per piece of state, use the design system's tokens and not AI defaults, hold the accessibility floor (keyboard, focus only on user action, labels, contrast, reduced motion), and verify in the running app with a screenshot you look at. Charts go through a dataviz skill. Use when touching components, a theme, a dialog, a layout, an HTML artifact, or UI copy."
---

# Frontend

UI that reads like a careful engineer built it for this product, not like a template. Rules first, then the pixels, then the proof on screen.

## Scope

- **App UI:** screens people use to get work done (dashboards, settings, editors, chat panes).
- **HTML artifacts:** reports, plans, comparisons and explainers an agent writes for a person to read or annotate.
- The project's own conventions always win over the defaults below.

Not for: backend code, CLI text output, slide decks. A chart, graph or stat tile goes through your dataviz skill first.

## 1. Read the project's UI rules [bl: invariant]

UI code has places that fail silently. Read the project's frontend doc and agent file (`AGENTS.md`, `CLAUDE.md`, `docs/FRONTEND.md` or similar) before the first edit. Look for these traps:

- **A committed bundle.** If the app ships a built bundle from the repo, rebuild and commit it after a source edit. Otherwise nobody sees the change and CI stays green.
- **Vendored upstream code.** Change it only through the seams the project gives (theme registration, string overrides, extension points), never by editing the vendored copy.
- **CSS specificity.** A theme block can lose to a core default with the same specificity. Check which rule actually wins on the root element.
- **Generated files.** Regenerate them with their command. Never hand-edit them.
- **One owner per contract.** When README, a guidance file and a skill all describe one behavior, update the owner, not a copy.

## 2. Name the mode first [bl: strong]

The mode is what a win looks like for the person on this screen. Name it in one line before you judge a pixel.

| Mode | Where | Rules |
|---|---|---|
| Operate | dashboards, settings, editors, a review viewer's chrome | Finish a task. Calm surfaces, dense but readable, few colors, one accent, minimal chrome. Headings say what the area is or does ("Wake-ups", "Plan status"). No card mosaics. |
| Read | reports, explainers, comparisons | Understand something. One column of 65 to 75 characters, headings closer to what follows, wayfinding (where am I, what is next). No hero. |
| Persuade | a launch page, a demo | Decide and act. Write a short design plan (four to six named colors, type roles, layout, principles), check it against the brief for defaults, then build. |

A mixed page is classified per section.

## 3. Structure and state [bl: strong]

- Composition over configuration. A component does one thing. Split past about 200 lines.
- Keep data fetching apart from rendering. Every async view has loading, empty, error and success states. An empty screen says what to do next.
- Use the simplest state that works: local, lifted, context, URL, then a server cache. Do not drill props more than three levels.
- **One owner per piece of state.** If the server owns a value, the UI does not set it optimistically where a late event can lose to it. [bl: invariant]
- In TypeScript, model view states as discriminated unions, use `unknown` at the boundary, and no `as` casts.

## 4. Tokens, look and copy [bl: strong]

- Use the design system's tokens: CSS variables, its spacing scale, its type scale. No raw hex. No off-scale pixel values.
- A theme is a brief. If the product has a deliberate look, follow it exactly inside that theme.
- Where the look is open, avoid AI defaults:
  - purple or indigo gradients, rounded-everything, the same soft shadow under every card;
  - a grid of identical cards as the layout;
  - an ALL-CAPS eyebrow over every heading, one accented word in a headline, middle-dot meta strings;
  - a `→` appended to buttons and links, monospace for small labels.
  Each is fine when the brief asks for it. Do not reach for one by default.
- Spend boldness in one place. Before you finish, remove one accessory.
- When the look is open and matters (a launch page, a demo, a new product page), borrow from a dedicated design skill instead of growing this one: [Impeccable](https://github.com/pbakaus/impeccable) (polish, bolder, critique), [taste-skill](https://github.com/Leonxlnx/taste-skill) (art direction, reference images), [UI/UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) (styles, palettes, font pairings), and [Anthropic frontend-design](https://github.com/anthropics/skills/tree/main/skills/frontend-design).
- Copy is utility language in sentence case. A button says what happens ("Save changes", not "Submit"). An action keeps its name through the flow ("Publish" then "Published"). Errors say what happened and how to fix it, with no apology.
- Charts, sparklines, meters and stat tiles: your dataviz skill. Where the destination renders live charts, hand it the rows, not a picture.

## 5. Accessibility floor [bl: invariant]

Every UI change holds this floor. It is not a later pass.

- Every control is reachable by Tab and works with Enter and Space. Prefer a native `<button>` to a `div` with a role. Focus is visible.
- **Focus moves only on a user action.** A reload, restore or live update never steals focus. Closing an overlay returns focus to where it came from. An `aria-modal` dialog keeps Tab inside it.
- Icon-only buttons get an accessible name. Inputs get visible labels, not placeholder-only labels.
- Text contrast is at least 4.5:1 (3:1 for large text). Color is never the only signal.
- Status changes are announced (`role="status"` or `aria-live`).
- `prefers-reduced-motion` is honored. Motion shows what changed. It does not decorate.
- The layout works at the narrowest window the product supports and at a wide one.

## 6. Verify in the running app [bl: invariant]

Tests prove the logic. Only the screen proves the UI.

- Run the project's unit tests for the logic, then drive the real app (Playwright or the project's end-to-end harness) on a throwaway profile, and save named screenshots. CI on the PR is the build gate.
- For focus and keyboard behavior, assert the exact behavior in a real browser (for example, Tab from the last control wraps to the first).
- **Look at the screenshot yourself.** Look first, then pick selectors from what rendered. Name the first three things your eye goes to and check they are the ones that matter. Check you can tell at once where you are and what you can do here.
- Stop and show the user after five fixes, after any revert, or once a fix touches files outside the change.

## 7. Doubt the risky ones [bl: soft]

A new flow, a new state owner, or a change to focus or keyboard handling gets a `doubt-driven-review` pass with the design lens before it stands.

## Worked example: a review viewer's lockout and focus-steal

A local HTML review viewer had two bugs a user hit in one live session.

- **State had two owners.** After a send, the viewer's chrome set the agent's presence to "working" itself. A "listening" event could arrive first and lose to that write, and the send button then dropped the next batch without a word. Rule 3: the server owns presence. The optimistic write went, and sending now gates only on the session having ended.
- **Focus moved on a restore.** On live reload, reopening a saved annotation card focused its textarea and pulled focus into the artifact iframe, so keystrokes meant for the message box landed in the card. Rule 5: autofocus only on an open the user started, never on a restore.
- **Verify:** regression tests at each seam (the autofocus predicate, back-to-back sends while the agent works, presence returning to listening on re-attach).

## Red flags

- Editing vendored UI code, or a source edit without the rebuilt bundle the app ships.
- A UI value set by both the client and the server.
- Focus moved by anything other than the user.
- A div with a click handler, an icon button with no name, a placeholder as the only label.
- Raw hex or off-scale pixel values in a themed surface.
- The template look on a surface that did not ask for it.
- "Done" with no screenshot, or a screenshot nobody looked at.

## Related

- `doubt-driven-review` for risky UI decisions.

## Sources

- [addyosmani/agent-skills `frontend-ui-engineering`](https://github.com/addyosmani/agent-skills/blob/aeac41d4d005999432bb7312f7ff06383a91d59d/skills/frontend-ui-engineering/SKILL.md) at `aeac41d`, MIT (see `THIRD_PARTY_LICENSE-agent-skills`): composition, container and presentation, the state ladder, the four async states, tokens and spacing scale, the AI aesthetic table, WCAG keyboard, labels, focus and contrast, the 200-line flag.
- [anthropics/skills `frontend-design`](https://github.com/anthropics/skills/blob/dbd4588f9e1033efb41dad4bef2f7947c8993d44/skills/frontend-design/SKILL.md) and [`webapp-testing`](https://github.com/anthropics/skills/blob/dbd4588f9e1033efb41dad4bef2f7947c8993d44/skills/webapp-testing/SKILL.md) at `dbd4588`, Apache-2.0 (see `THIRD_PARTY_LICENSE-anthropic-skills`): the calibration list of generated-design tells, plan then review against the brief, spend boldness in one place, remove one accessory, motion only for what changed, the writing-in-design rules; look before picking selectors.
- [garrytan/gstack `design-review`](https://github.com/garrytan/gstack/blob/576839cb8ce73c08a05a71efa91e80ca19fa630b/design-review/SKILL.md) at `576839c`, MIT (see `THIRD_PARTY_LICENSE-gstack`): name the mode first (operate, read, persuade), the app UI and read rules, the first-impression and trunk tests, fix-loop self-regulation.
- [cursor/plugins pstack `principle-experience-first`](https://github.com/cursor/plugins/blob/10d7b8a647caedfca8143ca8335d9e063c027227/pstack/skills/principle-experience-first/SKILL.md) and `typescript-best-practices` at `10d7b8a`, MIT (see `THIRD_PARTY_LICENSE-pstack`): ship less, ship better, get the details right, the TypeScript rules.
- Rewritten here, not copied. The `[bl: ...]` tags are analysed in `BITTER_LESSON.md`.
