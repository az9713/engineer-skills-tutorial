# migrate-to-shoehorn

Migrates test files from `as` type assertions to `@total-typescript/shoehorn`, a library that provides type-safe alternatives for passing partial or intentionally wrong data in tests.

---

## The problem it solves

TypeScript's `as` assertion in tests has two problems. `as Type` requires specifying the full type and providing all required fields even when only one field matters for the test. `as unknown as Type` for intentionally wrong data loses autocomplete and type checking. Both encourage writing verbose test fixtures or disabling type safety.

Shoehorn provides three targeted functions that solve each case without disabling TypeScript.

---

## How it works

The skill has a workflow phase plus a reference section with migration patterns.

### Migration patterns

| Pattern | When to use | Example |
|---------|------------|---------|
| `fromPartial({...})` | Large objects where only some properties matter | Pass `{ body: { id: "123" } }` where the type requires 20+ fields |
| `fromAny({...})` | Intentionally wrong data (for error testing) | Pass `{ body: { id: 123 } }` where `id` should be a string |
| `fromExact({...})` | Force full object (useful for later switching to fromPartial) | Validate that the full object is correctly typed |

### Workflow

1. **Gather requirements** — ask the user which test files have `as` assertions, whether they deal with large objects where only some properties matter, and whether they need to pass intentionally wrong data.

2. **Install and migrate:**
   - Install: `npm i @total-typescript/shoehorn`
   - Find test files: `grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   - Replace `as Type` with `fromPartial()`
   - Replace `as unknown as Type` with `fromAny()`
   - Add imports
   - Run type check to verify

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Why shoehorn, three function descriptions, migration patterns with before/after, workflow |

No companion files.

---

## Mechanism

The skill is a focused codemod recipe — it answers exactly one question (how do I replace `as` assertions in tests?) with concrete before/after examples for each case. The three-function API maps directly to three migration patterns, making the decision tree simple: if it's `as Type`, use `fromPartial`; if it's `as unknown as Type`, use `fromAny`.

---

## How to improve

**Add a note that this is test-code only.** The skill includes "Test code only. Never use shoehorn in production code." but this is in the "Why shoehorn?" section which a reader skimming for the workflow might miss. Moving it to the workflow as a prominent warning would make the boundary clearer.

**Add the `fromExact` use case.** The three functions are listed in the table but `fromExact` only appears in the table header — it has no before/after example. Adding an example would complete the migration guide.

**Add guidance on nested `as` assertions.** Some tests have `as` inside nested objects or function calls: `{ user: { role: "admin" as Role } }`. These require a different approach (replace the inner `as` or restructure with `fromPartial` at the outer level). A note about this case would handle a common migration edge case.
