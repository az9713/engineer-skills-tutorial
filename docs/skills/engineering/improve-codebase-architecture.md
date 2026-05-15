# improve-codebase-architecture

Surface architectural friction in a codebase, propose deepening opportunities (refactors that turn shallow modules into deep ones), and guide the user through implementing the chosen improvement. Informs its analysis using the project's `CONTEXT.md` glossary and `docs/adr/` decisions.

---

## The problem it solves

AI agents accelerate software entropy. Because code can be written fast, technical debt accumulates fast too. The typical result: a codebase full of shallow pass-through modules, tangled callers, no good test seams, and behavior spread across too many places.

This skill applies the "deep modules" philosophy from Ousterhout's _A Philosophy of Software Design_ to find and fix these patterns. Rather than "refactor because it's messy," it has a specific direction: deepen interfaces to increase leverage for callers and locality for maintainers.

---

## How it works

### Phase 1: Explore

Read `CONTEXT.md` and any ADRs in the relevant area first. Then use an Explore sub-agent to walk the codebase organically, looking for friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules shallow — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called?
- Where do tightly-coupled modules leak across their seams?
- Which parts are untested or hard to test through their current interface?

The **deletion test** is applied throughout: imagine deleting a module. If complexity vanishes, it was a pass-through and earns nothing. If complexity reappears across N callers, it was earning its keep.

### Phase 2: Present candidates

Numbered list of deepening opportunities. For each:

| Field | What it contains |
|-------|-----------------|
| **Files** | Which files/modules are involved |
| **Problem** | Why the current architecture causes friction |
| **Solution** | Plain English: what would change |
| **Benefits** | In terms of locality, leverage, and test improvement |

Vocabulary discipline: use `CONTEXT.md` terms for domain concepts ("the Order intake module") and `LANGUAGE.md` terms for architecture ("deepening the seam," "shallow module"). Never say "component," "service," "API," or "boundary."

If a candidate contradicts an existing ADR, surface it explicitly: "contradicts ADR-0007 — but worth reopening because…"

The agent does NOT propose interfaces yet. It presents candidates and asks the user to pick.

### Phase 3: Grilling loop

Once the user picks a candidate, the agent runs a grilling session to design the deepened module. Decisions crystallize as the conversation progresses.

**Inline side effects during the session:**

- New domain concept named during design → add to `CONTEXT.md` immediately
- User rejects a candidate with a load-bearing reason → offer an ADR ("Want me to record this so future architecture reviews don't re-suggest it?")
- Need to explore alternative interfaces → invoke the parallel sub-agent pattern from `INTERFACE-DESIGN.md`

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Three-phase process with vocabulary discipline and ADR conflict handling |
| `LANGUAGE.md` | Shared vocabulary for architecture — module, interface, depth, seam, adapter, leverage, locality |
| `DEEPENING.md` | How to deepen a cluster of shallow modules given its dependency type |
| `INTERFACE-DESIGN.md` | Parallel sub-agent pattern for exploring alternative interface designs |

### LANGUAGE.md in detail

The vocabulary file is the skill's most important asset. It defines eight terms precisely, with `_Avoid_` aliases for each, plus four principles and a "Rejected framings" section that explains why alternative definitions were rejected.

The `_Avoid_` aliases are as important as the definitions. "Avoid 'service'" tells the agent not to say "the Order service" — say "the Order module." This prevents the vocabulary from drifting back to informal usage during long sessions.

The "Rejected framings" section records why Ousterhout's depth-as-ratio-of-lines was rejected in favor of depth-as-leverage. This prevents future sessions from reverting to the rejected definition.

### DEEPENING.md in detail

Classifies dependencies into four categories, which determines how the deepened module is tested:

| Category | Example | Testing strategy |
|----------|---------|-----------------|
| In-process | Pure computation, in-memory state | No adapter needed — test through new interface directly |
| Local-substitutable | Postgres with PGLite stand-in | Test with the stand-in running in the test suite |
| Remote but owned | Internal microservice | Ports & Adapters — port at seam, HTTP adapter for prod, in-memory for tests |
| True external | Stripe, Twilio | Port + mock adapter |

The "one adapter = hypothetical seam, two adapters = real seam" rule is here. Don't introduce a port unless at least two adapters are justified.

### INTERFACE-DESIGN.md in detail

Implements Ousterhout's "Design It Twice" — your first interface idea is unlikely to be best. The skill spawns 3+ sub-agents in parallel, each designing a radically different interface under a different constraint:

- Agent 1: Minimize the interface (1–3 entry points max)
- Agent 2: Maximize flexibility (many use cases, extension points)
- Agent 3: Optimize for the most common caller
- Agent 4 (if applicable): Design around ports and adapters for cross-seam dependencies

Each agent produces the interface, a usage example, what the implementation hides, the dependency strategy, and trade-offs. The coordinator then compares them by depth, locality, and seam placement, gives a recommendation, and proposes a hybrid if elements from different designs would combine well.

---

## Mechanism

The skill works because it's grounded in a specific architectural direction (deepen modules) with a specific metric (leverage and locality). Vague "clean up this code" prompts produce vague improvements. "Deepen this module so behavior is concentrated behind a smaller interface" produces a specific, verifiable change.

The vocabulary discipline enforces consistency across sessions. When the agent always says "seam" instead of "boundary," and always uses `CONTEXT.md` terms for domain concepts, both the grilling conversation and the resulting code use the same language.

---

## How to improve

**Add prioritization guidance for candidates.** The skill surfaces candidates but doesn't say which to tackle first. Heuristics like "prioritize modules with the highest change frequency, lowest test coverage, or most callers" would help when there are 10+ candidates and the user needs to pick.

**Add an exploration heuristic toolkit.** Phase 1 says "explore organically" but the organic approach is only as good as the explorer's intuition. Adding heuristics — "look at files changed most frequently in git log," "look at files imported by 10+ other files," "look for functions with 5+ parameters" — gives the Explore sub-agent more systematic starting points.

**Add an exit criterion.** The skill doesn't say when the architecture is "improved enough." A closing question — "Does the chosen module now pass the deletion test?" or "Can the critical path be tested through a single interface?" — would give the session a concrete done condition.

**Make the ADR conflict handling clearer.** The skill says to surface candidates that contradict ADRs "only when the friction is real enough to warrant reopening." This is a judgment call without much guidance. Adding a threshold — "if the friction causes you to miss bugs or write brittle tests, it's worth reopening" — would make the judgment more consistent.
