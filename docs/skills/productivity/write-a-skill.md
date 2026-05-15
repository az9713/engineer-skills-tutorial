# write-a-skill

Creates new skills with proper structure, a good description for reliable triggering, and companion files when needed.

---

## The problem it solves

Skills are only as good as their description and structure. A skill with a vague description won't trigger reliably — the agent can't tell when it's relevant. A skill stuffed into one file becomes hard to navigate. This skill encodes the rules for building skills that work well.

---

## How it works

### Phase 1: Gather requirements

Ask the user:
- What task/domain does the skill cover?
- What specific use cases should it handle?
- Does it need executable scripts or just instructions?
- Any reference materials to include?

### Phase 2: Draft the skill

Create:
- `SKILL.md` with concise instructions
- Additional reference files if content exceeds 500 lines
- Utility scripts if deterministic operations are needed

### Phase 3: Review with user

Present the draft and ask:
- Does this cover your use cases?
- Anything missing or unclear?
- Should any section be more or less detailed?

---

## Skill structure

```
skill-name/
├── SKILL.md           ← main instructions (required)
├── REFERENCE.md       ← detailed docs if needed
├── EXAMPLES.md        ← usage examples if needed
└── scripts/           ← utility scripts if needed
    └── helper.js
```

---

## The description: the most important field

The `description` frontmatter field is the only thing the agent sees when deciding which skill to load. It needs to convey two things:
1. What capability this skill provides
2. When/why to trigger it (specific keywords, contexts, file types)

Constraints:
- Max 1024 characters
- Written in third person
- First sentence: what it does
- Second sentence: "Use when [specific triggers]"

**Good:** "Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when user mentions PDFs, forms, or document extraction."

**Bad:** "Helps with documents."

---

## When to add scripts

Add utility scripts when:
- The operation is deterministic (validation, formatting)
- The same code would be generated repeatedly
- Errors need explicit handling

Scripts save tokens and improve reliability compared to regenerating the same code each session.

---

## When to split files

Split into separate files when:
- SKILL.md exceeds 100 lines
- Content has distinct domains
- Advanced features are rarely needed

The 100-line limit keeps `SKILL.md` scannable. Deep content (format specs, advanced options, examples) lives in linked files loaded on demand.

---

## Review checklist

- [ ] Description includes triggers ("Use when...")
- [ ] SKILL.md under 100 lines
- [ ] No time-sensitive info
- [ ] Consistent terminology
- [ ] Concrete examples included
- [ ] References one level deep (no nested links)

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Process, structure rules, description requirements, when to split |

No companion files.

---

## Mechanism

The description requirement is the skill's core contribution. Skills.sh-style skill systems route by description matching, not by command name. A precise description with specific triggers is what makes a skill reliable — the agent can decide between 50 installed skills in milliseconds if the descriptions are precise.

The 100-line SKILL.md limit enforces progressive disclosure: the main file stays tight, deep content lives in linked files. This mirrors the "deep module" philosophy from the engineering skills — a small, clear interface with depth behind it.

---

## How to improve

**Add a lint rule for the 100-line limit.** The skill says SKILL.md should be under 100 lines but this is only enforced by review. Adding a script that validates SKILL.md length would make this a hard gate rather than a soft guideline.

**Add a description evaluation prompt.** The skill defines good vs bad descriptions but doesn't tell the agent how to evaluate one it's written. Adding "after drafting the description, ask yourself: could an agent distinguish this from a similar skill based solely on this description?" would make the evaluation step explicit.

**Add guidance on skill versioning.** Skills change over time as users discover they work better with different instructions. There's no guidance on how to handle backwards compatibility when modifying an existing skill — whether to update in place or create a new version.

**Add examples of the skills in this repo as illustrations.** The best examples of good skill structure are the skills themselves. Pointing to `/grill-me` (minimal, three sentences) and `/diagnose` (complex, six phases with companion files) as examples of the spectrum would ground the guidelines in concrete cases.
