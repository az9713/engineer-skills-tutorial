# prototype

Builds throwaway code that answers one specific design question. Routes between two branches: a logic prototype (interactive terminal app for state/business-logic questions) or a UI prototype (multi-variant page with a floating switcher for visual questions).

---

## The problem it solves

Design decisions made on paper or in conversation are often wrong in ways that only appear when you interact with the thing. A state machine that looks correct in a diagram may have impossible transitions when you try to drive it by hand. A UI layout that seems obvious in mockup form may feel wrong when it's live next to real data.

Prototypes are cheap. Real implementations are expensive. This skill formalizes the throwaway prototype as a distinct artifact with a clear lifecycle: build it, answer the question, delete it.

---

## How it works

### Picking the branch

The skill's first step is identifying which question is being answered:

- **"Does this logic / state model feel right?"** → Logic branch (LOGIC.md)
- **"What should this look like?"** → UI branch (UI.md)

If the question is ambiguous and the user isn't reachable, the agent defaults to whichever branch better matches the surrounding code (backend module → logic; page or component → UI) and states the assumption at the top.

Getting this wrong wastes the whole prototype — two fundamentally different artifacts are produced. The skill is explicit about this risk.

### Rules for both branches

1. **Throwaway from day one, clearly marked.** Located close to where it will be used; named so it's obviously a prototype, not production.
2. **One command to run.** `pnpm <name>` or equivalent — no path to remember.
3. **No persistence by default.** State lives in memory.
4. **Skip the polish.** No tests, no error handling beyond runnable, no abstractions.
5. **Surface the state.** After every action (logic) or on every variant switch (UI), the full state is visible.
6. **Delete or absorb when done.** The answer is the only thing worth keeping.

### When done

Capture the answer somewhere durable (commit message, ADR, issue, or `NOTES.md`) along with the question it was answering. Then delete the prototype or fold the validated decision into the real code.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Branch selection, shared rules, lifecycle |
| `LOGIC.md` | Full logic prototype process — TUI structure, state design, anti-patterns |
| `UI.md` | Full UI prototype process — variant generation, switcher implementation, sub-shapes |

### LOGIC.md in detail

**State the question first.** Before writing code, write down what state model and what question you're prototyping. A logic prototype that answers the wrong question is waste.

**Isolate the logic in a portable module.** The key design principle: put the actual logic behind a small, pure interface that could be lifted into the real codebase. The TUI is throwaway; the logic module shouldn't be. Four shapes for the logic module:
- A pure reducer: `(state, action) => state`
- A state machine with explicit states and transitions
- A small set of pure functions over a plain data type
- A class with a clear method surface

**Build the smallest TUI that exposes the state.** Clear-screen loop — every tick renders the current state and keyboard shortcuts. The whole frame should fit on one screen. Bold for state field names, dim for context values like IDs and timestamps.

**Anti-patterns:** Don't add tests. Don't wire to the real database. Don't generalize. Don't blur the logic and TUI together. Don't ship the TUI shell to production.

### UI.md in detail

**Two sub-shapes:**

- **Sub-shape A (preferred):** Adjustment to an existing page. Variants rendered on the same route, gated by a `?variant=` URL param. The existing data fetching stays; only the rendering swaps. Use this by default.
- **Sub-shape B (last resort):** A new throwaway route when the prototype genuinely has no existing page to live inside.

**Generate radically different variants.** Default: 3 variants. Each must be structurally different — different layout, information hierarchy, primary affordance. "Three slightly-tweaked card grids" is not a UI prototype.

**The floating switcher:** A fixed-position bar at bottom-center with left/right arrows and the current variant label. URL-param-driven (shareable, reload-stable). Keyboard navigable (←/→). Hidden in production builds. Never intercepts arrow keys when an input is focused.

**Clean up:** Fold the winner into the page, delete all losing variant components and the switcher.

---

## Mechanism

The branch split is the key design decision. Logic and UI prototypes answer different questions, live in different places, and have different lifecycles. Merging them into one "prototype anything" skill would produce a confused process for both.

The "one command to run" constraint forces the prototype to be immediately usable. A prototype the user can't run in 10 seconds isn't a tool for quick feedback — it's work in progress.

The "capture the answer" requirement closes the prototype loop. Without it, the prototype gets deleted and the knowledge disappears. With it, the insight lives in a commit message, ADR, or issue that survives the prototype.

---

## How to improve

**Add guidance on prototype duration.** The skill says to "delete or absorb when done" but doesn't say how long a prototype should run. A guideline like "if the prototype hasn't answered its question in one or two sessions, either the question is unclear or the prototype is the wrong shape" would help avoid prototype drift.

**Add a "when NOT to prototype" section.** Prototypes are for design questions under genuine uncertainty. When requirements are clear, a prototype delays real implementation. A short note — "skip prototyping when the interface is already defined in a PRD or ADR, or when you're building something similar to existing code in the repo" — would help calibrate usage.

**Link `NOTES.md` template.** The skill mentions leaving a `NOTES.md` next to the prototype for the answer, but there's no template. A minimal structure (Question, Answer, Disposal plan) would make the capture step faster to complete.

**Add guidance on prototype quality threshold for logic modules.** LOGIC.md says "the logic module shouldn't be throwaway" — it should be liftable into the real codebase. But "liftable" is vague. Adding "the logic module should have no I/O, no terminal code, and a clear public interface matching what the real module will expose" makes the bar concrete.
