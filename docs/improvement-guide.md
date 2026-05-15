# Improvement guide

Patterns for improving existing skills and writing new ones, drawn from dissecting the skills in this repo.

---

## The three skill quality dimensions

Every skill can be evaluated on three dimensions:

**Trigger precision** — does the `description` reliably cause the agent to load the skill at the right times and not load it at the wrong times? A vague description causes missed triggers and false positives.

**Process completeness** — does the skill define every decision point, phase transition, and termination condition? A skill with gaps causes the agent to improvise, which produces inconsistent results.

**Asset leverage** — are the companion files the right size and shape? Too much in SKILL.md makes it hard to scan; too little means the agent lacks detail when it needs it.

---

## Pattern 1: Make every gate explicit

The best skills in this collection define gates between phases — conditions that must be true before the next phase starts. `diagnose` does this explicitly: "Do not proceed to Phase 2 until you have a loop you believe in."

Without explicit gates, an agent under time pressure will skip phases. With gates, skipping requires an explicit "skip this gate because X," which forces a conscious decision.

**How to apply:** Review every skill you maintain. For each phase transition, ask: what must be true for the agent to move on? If the answer isn't stated, add it.

---

## Pattern 2: Write the termination condition

Skills that say "interview relentlessly" or "explore organically" without saying when to stop produce sessions that either end too early (first plausible answer) or too late (exhausting every possible question).

The `diagnose` skill has clear termination: Phase 6 has a checklist — when all boxes are checked, you're done. The `grill-me` skill doesn't — "relentlessly" is undefined.

**How to apply:** Add a "Done when:" line or closing checklist to any skill whose termination is currently defined by feel.

---

## Pattern 3: Use the vocabulary layer

Three skills establish shared vocabulary: `LANGUAGE.md` in `improve-codebase-architecture`, `CONTEXT-FORMAT.md` and `ADR-FORMAT.md` in `grill-with-docs`, and `CONTEXT.md` (the project-level glossary) maintained by `grill-with-docs` and read by most other skills.

When a skill uses consistent vocabulary across sessions, the agent's output is consistent across sessions. When vocabulary drifts ("module" becomes "service" becomes "component"), the agent's ability to navigate and discuss the codebase degrades.

**How to apply:** If you're building a skill that introduces technical concepts specific to its domain, write a vocabulary file (`LANGUAGE.md` or similar) with definitions and `_Avoid_` aliases. Reference it from the main SKILL.md.

---

## Pattern 4: Make the description do real work

The `description` frontmatter field is what determines whether a skill fires. Compare:

| Skill | Description quality |
|-------|-------------------|
| `diagnose` | Excellent — lists exact trigger phrases ("diagnose this", "debug this"), describes what the user says, not what the skill does |
| `zoom-out` | Good — specific trigger condition ("unfamiliar with a section") |
| `handoff` | Adequate — describes what it does but trigger conditions are vague ("compact the current conversation") |
| `migrate-to-shoehorn` | Good — specific library name as trigger |

The key insight: descriptions that list what the user says trigger more reliably than descriptions that describe what the skill does. "Use when user says 'diagnose this'" is better than "Use when debugging."

**How to apply:** For each skill, add 3–5 exact phrases a user might say that should trigger it. Include negative examples if the skill name is ambiguous ("not for general refactoring — only when the user asks about codebase architecture").

---

## Pattern 5: Progressive disclosure via companion files

The best skills keep SKILL.md scannable by moving deep content to companion files. Compare:

| Skill | SKILL.md lines | Companion files |
|-------|--------------|----------------|
| `tdd` | ~110 | 5 files (tests, mocking, modules, interface, refactoring) |
| `improve-codebase-architecture` | ~72 | 3 files (language, deepening, interface design) |
| `zoom-out` | 8 | 0 |
| `grill-me` | 7 | 0 |

The pattern: skills that are always fully executed (every step, every session) keep everything in SKILL.md. Skills that have reference content used only sometimes split it out.

**How to apply:** For each skill, identify content that is "reference" (looked up occasionally) vs "process" (always executed). Move reference content to companion files. Link to them inline from the relevant step.

---

## Pattern 6: Self-demonstrating content

The `caveman` skill is written in caveman mode. This is the best possible documentation: the skill demonstrates its output format by example rather than describing it.

The `AGENT-BRIEF.md` asset in the `triage` skill includes a good brief, a bad brief, and an annotation of what makes the bad one bad. The contrast is more instructive than any rule.

**How to apply:** Wherever possible, write the skill in the style it produces. Add good/bad examples for format-sensitive output. The SKILL.md for a JSON-output skill should show example JSON.

---

## Pattern 7: Inline updates as decisions crystallize

`grill-with-docs` and `improve-codebase-architecture` both specify inline updates: when a term is resolved during grilling, update `CONTEXT.md` right there, not at the end of the session. This is the difference between a live document and a retrospective document.

The reason: inline capture is more accurate (the decision is fresh) and produces more engaged review (the human sees the term added immediately, not reconstructed later).

**How to apply:** For any skill that produces documentation artifacts, add "update inline as decisions are made, not at the end."

---

## Common weaknesses across this skill collection

Based on the per-skill analysis, these gaps appear repeatedly:

| Weakness | Affected skills |
|----------|----------------|
| Missing termination condition | `grill-me`, `grill-with-docs`, `zoom-out`, `improve-codebase-architecture` |
| No output format specification | `zoom-out`, `handoff` |
| Update/maintenance path missing | `setup-matt-pocock-skills`, `git-guardrails-claude-code` |
| Missing "when NOT to use" | `tdd`, `prototype` |
| Calibration guidance absent | `to-prd` (PRD length), `to-issues` (slice size) |

Addressing these would make the skill collection more consistent and predictable across sessions.

---

## Adding a new skill: checklist

```
[ ] Description includes exact trigger phrases
[ ] SKILL.md is under 100 lines
[ ] Every phase transition has an explicit gate or condition
[ ] Termination condition is stated
[ ] Reference content is in companion files, not SKILL.md
[ ] At least one good/bad example for format-sensitive output
[ ] No time-sensitive information (version numbers, current dates)
[ ] Terminology is consistent — define it if it's novel
[ ] "When NOT to use this skill" is addressed if the trigger is broad
```
