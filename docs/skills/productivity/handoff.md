# handoff

Compacts the current conversation into a handoff document so another agent session can continue the work without losing context.

---

## The problem it solves

Agent sessions have limited context windows. When a session ends mid-task, the next session starts cold — it doesn't know what was tried, what was decided, or what's left to do. Reconstructing this context wastes tokens and time.

This skill captures the current session's state as a durable artifact that a new session can read at startup.

---

## How it works

The SKILL.md instruction:

> Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save it to a path produced by `mktemp -t handoff-XXXXXX.md` (read the file before you write to it).

Three constraints:

1. **`mktemp -t handoff-XXXXXX.md`** — creates a temp file with a random suffix to avoid naming conflicts. The agent reads it first (to satisfy the read-before-write requirement) then writes the handoff content.
2. **Don't duplicate existing artifacts.** If a PRD, ADR, issue, commit, or diff already captures something, reference it by path or URL rather than restating it.
3. **Suggest skills for the next session.** If the work has an obvious next step (e.g. "the PRD is written, next session should run `/to-issues`"), say so.

If the user passed arguments (`/handoff what-the-next-session-will-focus-on`), the document is tailored to that focus rather than being a general summary.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Single instruction with three constraints |

No companion files.

---

## Mechanism

The "don't duplicate" constraint is the key design choice. A handoff document that restates the entire PRD is noise — it inflates context for the next session without adding value. By referencing existing artifacts instead, the handoff stays focused on what's novel: what was tried, what was decided in conversation, and what's left to do.

The `mktemp` approach gives the document a predictable naming pattern (`handoff-XXXXXX.md` in the system temp directory) that's easy to find and share, without requiring the user to specify a path.

---

## How to improve

**Add a template structure.** The skill says "summarising the current conversation" but doesn't define what that summary should contain. A handoff document for a debugging session looks very different from one for a planning session. A minimal template — Current state, What was tried, Key decisions, What's left, Suggested next skill — would produce more consistently useful output.

**Add guidance on what "continuing the work" means.** The handoff is written for a fresh agent — but agents have different context window sizes and different capabilities. A note about what assumptions a receiving agent can make (it can access the same codebase, same issue tracker, same tools) would make the handoff more precise.

**Support a structured path option.** `mktemp` generates a path like `/tmp/handoff-a4f2.md` which is hard to find after the session. An alternative like `docs/handoffs/YYYY-MM-DD-description.md` would make handoffs easier to review and version-control.

**Add a "skills to load at startup" note.** The skill says to "suggest the skills to be used," but this is vague. Adding a specific format — "Start with: `/grill-with-docs` (resume the domain alignment session) then `/to-prd` (capture the plan)" — would give the next agent a clearer startup sequence.
