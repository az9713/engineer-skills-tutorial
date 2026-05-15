# scaffold-exercises

Creates AI Hero course exercise directory structures — sections with problems, solutions, and explainers — that pass the `pnpm ai-hero-cli internal lint` validator.

---

## The problem it solves

AI Hero course exercises follow a strict directory and file convention enforced by a linter. Hand-creating these structures is tedious and error-prone. This skill automates the creation from a plan, then validates with the linter before committing.

> **Note:** This skill is specific to AI Hero course repositories that use `ai-hero-cli`. It is not general-purpose.

---

## How it works

### Directory naming

- **Sections:** `XX-section-name/` inside `exercises/` (e.g. `01-retrieval-skill-building`)
- **Exercises:** `XX.YY-exercise-name/` inside a section (e.g. `01.03-retrieval-with-bm25`)

### Exercise variants

Each exercise has at least one subfolder:
- `problem/` — student workspace with TODOs
- `solution/` — reference implementation
- `explainer/` — conceptual material, no TODOs

Default when stubbing: `explainer/` unless the plan specifies otherwise.

### Required files

Each subfolder needs a `readme.md` that is non-empty (even a single title line) with no broken links. If the subfolder has code, it also needs a `main.ts` (>1 line).

### Workflow

1. **Parse the plan** — extract section names, exercise names, and variant types
2. **Create directories** — `mkdir -p` for each path
3. **Create stub readmes** — minimal `readme.md` with a title per variant folder
4. **Run lint** — `pnpm ai-hero-cli internal lint`
5. **Fix any errors** — iterate until lint passes

### Moving/renaming

Use `git mv` (not plain `mv`) to preserve git history when renumbering exercises.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Naming rules, required files, lint rules summary, complete workflow with example |

No companion files.

---

## Mechanism

The skill is a deterministic recipe for generating files that satisfy a known linter. Its value is encoding the lint rules into the creation process — rather than creating files and iterating on lint errors, the skill generates files that will pass lint on the first attempt (or close to it).

The "lint then fix" loop at the end is the safety net. It catches any edge cases the recipe missed.

---

## How to improve

**Add a note about the ai-hero-cli requirement.** The skill is only useful in AI Hero course repos. A prominent note at the top — "This skill requires `ai-hero-cli` installed as a dev dependency" — would prevent confusion when running in other repos.

**Add guidance on section numbering gaps.** When inserting a new section between `04` and `06`, the skill doesn't say whether to renumber `06+` or leave a gap at `05`. A convention note would prevent inconsistent numbering across the course.

**Add the lint rule for `speaker-notes.md`.** The lint rules summary includes "No `speaker-notes.md` files" — but there's no guidance on what to do if a speaker notes file exists in the plan. A note about how to handle speaker notes (where they should go, if anywhere) would prevent the linter from blocking a valid structure.
