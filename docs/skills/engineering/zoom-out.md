# zoom-out

Tells the agent to step back from implementation detail and produce a module map of the relevant area using the project's domain vocabulary. One-line skill for navigating unfamiliar code.

---

## The problem it solves

When working in an unfamiliar part of a codebase, an agent (or human) can get lost in implementation detail without understanding how the module fits into the broader system. "What does this function do?" is a different question from "what calls this function, and why does it exist in this system?"

This skill provides a fast way to get the higher-level view.

---

## How it works

The SKILL.md is a single instruction:

> I don't know this area of code well. Go up a layer of abstraction. Give me a map of all the relevant modules and callers, using the project's domain glossary vocabulary.

The skill is marked `disable-model-invocation: true` — it doesn't use the LLM to process its content, just delivers the instruction directly.

The three directives in the instruction:

1. **"Go up a layer of abstraction"** — shift the agent from implementation-level to architecture-level thinking
2. **"Give me a map of all relevant modules and callers"** — produce a navigation artifact (who calls what, and from where) rather than a description of what the code does
3. **"Using the project's domain glossary vocabulary"** — use `CONTEXT.md` terms rather than file names or function names, which may not be meaningful out of context

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Single instruction (one line) |

No companion files.

---

## Mechanism

The skill's simplicity is intentional. "Zoom out" is a recovery action — something you invoke when you're lost. A complex skill with phases and checklists would be counterproductive. The one-line instruction gives the agent a clear direction and gets out of the way.

The domain vocabulary requirement ensures the map is comprehensible at a domain level, not just a technical level. Without it, the agent might produce "FileRepository → FileController → FileRouter" rather than "the Document intake module → the Publishing module."

---

## How to improve

**Add guidance on what a good map contains.** "A map of all relevant modules and callers" is open-ended. Specifying what "relevant" means — "include the 2–3 levels above and below the current module," "include any module that shares data with this one," "include the entry point that triggers this path" — would make the output more consistently useful.

**Add guidance on depth.** The agent currently decides how far up and out to zoom. For some use cases (navigating a bug), one level up is enough. For others (planning a refactor), a full system view is needed. Adding an optional depth parameter — `/zoom-out deep` for a full system view — would make the skill more precise.

**Clarify the output format.** The skill doesn't specify whether the map should be prose, a bullet-point hierarchy, a dependency list, or a diagram description. Prose tends to bury the structure; a bullet hierarchy or a "Module X → called by A, B, C; calls D, E" list would be more scannable.

**Add a follow-up prompt.** After receiving the map, the most common next action is to zoom into one of the identified modules. A suggested follow-up — "To dive into any module from this map, use `/zoom-out` again with that module name as context" — would make the navigation pattern more fluid.
