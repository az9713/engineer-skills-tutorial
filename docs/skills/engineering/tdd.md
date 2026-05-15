# tdd

Test-driven development with a strict red-green-refactor loop, one vertical slice at a time. Enforces behavior-level tests over implementation-detail tests.

---

## The problem it solves

Without a discipline, AI agents write tests in bulk after writing code, producing tests that test the shape of things (data structures, function signatures) rather than the behavior that matters. These tests are both fragile (they break on refactors that don't change behavior) and insensitive (they pass when behavior actually breaks).

The skill enforces vertical slicing: one test at a time, one implementation at a time, each cycle responding to what was learned in the previous one.

---

## How it works

### The core philosophy

The skill establishes two principles before giving any workflow instructions:

**Good tests verify behavior through public interfaces.** A test should survive an internal refactor that doesn't change behavior. If you rename an internal function and a test breaks, the test was testing implementation, not behavior.

**Horizontal slicing produces crap tests.** Writing all tests first, then all implementation (RED all → GREEN all) means tests are written in imagined context, not actual context. The agent "outruns its headlights" and commits to test structure before understanding the implementation.

### Phase 1: Planning

Before any code:
- Confirm with the user which behaviors to test (don't test everything — confirm priorities)
- Identify opportunities for deep modules (small interface, large implementation)
- Design interfaces for testability
- List behaviors to test, not implementation steps
- Get user approval on the plan

The planning phase explicitly references `deep-modules.md` and `interface-design.md` — both of which exist as asset files.

### Phase 2: Tracer bullet

Write ONE test that confirms ONE thing about the system, then write the minimal code to pass it. This proves the test path works end-to-end before adding complexity.

### Phase 3: Incremental loop

For each remaining behavior:
- Write next test → fails (RED)
- Write minimal code to pass → passes (GREEN)

Rules enforced:
- One test at a time
- Only enough code to pass the current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### Phase 4: Refactor

After all tests pass, look for refactor candidates in `refactoring.md`:
- Extract duplication
- Deepen modules
- Apply SOLID principles where natural
- Consider what new code reveals about existing code

**Never refactor while RED.** Get to GREEN first.

### Per-cycle checklist

Five questions to ask after each RED→GREEN cycle:
- Test describes behavior, not implementation
- Test uses public interface only
- Test would survive internal refactor
- Code is minimal for this test
- No speculative features added

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Workflow with philosophy, anti-patterns, and cycle checklist |
| `tests.md` | Good vs bad test examples with TypeScript code |
| `mocking.md` | When to mock (system boundaries only), how to design for mockability |
| `deep-modules.md` | ASCII diagram of deep vs shallow modules, design heuristics |
| `interface-design.md` | Three principles for testable interface design with code examples |
| `refactoring.md` | Six refactor candidates to look for after the GREEN phase |

### tests.md in detail

Two complete TypeScript examples — one good, one bad — for both types of test failure. The "bad test" for bypassing-the-interface is particularly instructive: it shows a test that queries the database directly instead of using the public API. The "good test" version uses `getUser()` instead of a raw SQL query, making the test resilient to schema changes.

### mocking.md in detail

Establishes the rule: mock at **system boundaries only** (external APIs, databases, time/randomness, sometimes the filesystem). Never mock your own modules or internal collaborators.

The "SDK-style interfaces" guidance is an important design pattern: rather than a generic `api.fetch(endpoint)` that requires conditional logic in mocks, create specific functions per operation (`api.getUser`, `api.createOrder`). Each is independently mockable, type-safe, and clear about what it tests.

### interface-design.md in detail

Three principles: accept dependencies rather than creating them (dependency injection), return results rather than producing side effects, and keep a small surface area. Each principle comes with a before/after TypeScript code pair.

---

## Mechanism

The skill prevents the two most common AI-specific testing failures:

1. **Bulk test writing** is prevented by the vertical-slice constraint. Each cycle is one test → one implementation, meaning tests are always written with full knowledge of the code that will make them pass.

2. **Implementation coupling** is prevented by the behavior-level test philosophy and the per-cycle checklist. After every cycle, the agent explicitly asks "would this test survive an internal refactor?"

The asset files serve as progressive disclosure: the main SKILL.md keeps the workflow tight, and deeper content (mock guidelines, deep module theory, concrete examples) lives in linked files that the agent loads when needed.

---

## How to improve

**Add test naming guidance.** The skill defines what to test (behavior, not implementation) and how to structure tests (vertical slices) but says nothing about how to name tests. "user can checkout with valid cart" vs "checkout success" — the first is significantly more readable. A short section in `tests.md` on naming conventions would complete the picture.

**Expand refactoring.md.** The refactor candidates file is six bullet points. It would benefit from the same treatment as `tests.md` — before/after examples for the most common candidates (extract duplication, deepen modules).

**Add guidance on when to skip TDD.** The skill is framed as always-applicable, but there are legitimate cases to skip it: throwaway prototypes (explicitly, for the `/prototype` skill), spike code to explore an unfamiliar API, or infrastructure config. A brief "when NOT to use TDD" section prevents the skill from being applied rigidly where it causes friction.

**Reduce the duplicate "Confirm with user" in planning.** Phase 1 says "Confirm with user what interface changes are needed" and then immediately "Confirm with user which behaviors to test." These could be combined into one conversation checkpoint.
