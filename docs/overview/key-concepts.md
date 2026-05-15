# Key concepts

Definitions for every term used across skill docs. Terms specific to a single skill are defined in that skill's doc; terms that appear across multiple skills are defined here.

---

## Skills and structure

**Skill** — a Markdown file (plus optional companion files) that gives an agent a process to follow for one specific task. Invoked with a `/name` command. The agent receives the SKILL.md content as context.

**Bucket** — a folder grouping related skills. The three public buckets are `engineering/`, `productivity/`, and `misc/`. Skills in `personal/`, `in-progress/`, and `deprecated/` are not published.

**Frontmatter** — the YAML block at the top of SKILL.md. The `name` field is the slash command; `description` is what the agent sees when deciding which skill to load.

**Asset file** — a companion file in the same folder as SKILL.md (e.g. `CONTEXT-FORMAT.md`, `LANGUAGE.md`). Loaded on demand by the skill, not automatically.

**Trigger** — a condition in the `description` field that tells the agent to load the skill. Example: "Use when user says 'diagnose this' / reports a bug".

---

## Domain documentation

**CONTEXT.md** — a project-level glossary file. Contains precise definitions of domain terms specific to the project. Skills use it to name things consistently. Written and maintained by `/grill-with-docs`.

**CONTEXT-MAP.md** — a root-level file that appears in multi-context repos (monorepos). It lists each context, where it lives, and how contexts relate to each other.

**ADR (Architecture Decision Record)** — a short document recording a technical decision, why it was made, and what alternatives were considered. Lives in `docs/adr/`. Created during `/grill-with-docs` grilling sessions.

**Domain language** — the shared vocabulary that developers and domain experts agree on. The antidote to overloaded terms. Captured in CONTEXT.md.

---

## Architecture vocabulary (from `improve-codebase-architecture`)

These terms are used precisely across multiple skills. See [LANGUAGE.md](../../skills/engineering/improve-codebase-architecture/LANGUAGE.md) for full definitions.

**Module** — anything with an interface and an implementation. Scale-agnostic: applies to a function, class, package, or tier-spanning slice. Avoid "component", "service", "unit".

**Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, configuration. More than just the type signature.

**Depth** — the leverage a module provides: how much behavior sits behind how small an interface. A deep module has a large implementation behind a small interface.

**Seam** — where an interface lives; a place where behavior can be altered without editing in place. Avoid "boundary" (overloaded with DDD's bounded context).

**Adapter** — a concrete thing that satisfies an interface at a seam.

**Shallow module** — a module where the interface is nearly as complex as the implementation. Sign of a pass-through that earns no leverage.

**Deletion test** — imagine deleting a module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.

---

## Issue tracking

**Issue tracker** — where issues live for a repo. Configured per-repo by `/setup-matt-pocock-skills`. Can be GitHub Issues, GitLab Issues, or local markdown files.

**Triage state** — the current workflow state of an issue. Five canonical states: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`.

**Agent brief** — a structured comment posted on an issue when it moves to `ready-for-agent`. The authoritative specification an AFK agent works from. Defined in `skills/engineering/triage/AGENT-BRIEF.md`.

**AFK agent** — an agent that runs without human interaction. An issue is `ready-for-agent` when it has an agent brief that is complete enough for the agent to implement and merge without asking questions.

**HITL (Human in the Loop)** — a task or step that requires human interaction. Opposite of AFK. Used in `/to-issues` to flag slices that need human judgment.

**Vertical slice** — an issue or task that cuts through all integration layers end-to-end (schema, API, UI, tests), delivering a narrow but complete piece of working software. The unit of work in `/to-issues` and `/tdd`.

---

## Testing

**Tracer bullet** — the first test in a TDD cycle. Proves one thing about the system end-to-end before adding more tests.

**Red-green-refactor** — the TDD cycle: write a failing test (red), write minimal code to pass it (green), then clean up the code (refactor). Never refactor while red.

**Integration-style test** — a test that exercises real code paths through public APIs, not mocked internals. The preferred test style in `/tdd`.

**Deep module** — a module with a small interface and a large implementation. Easy to test because the test surface is small. From _A Philosophy of Software Design_ (Ousterhout).

---

## Prototyping

**Throwaway prototype** — code written to answer one question, then deleted. Two branches: a logic prototype (terminal app for state/business-logic questions) or a UI prototype (multi-variant page with a switcher).

**Logic prototype** — a tiny interactive terminal app that lets the user drive a state model. Used to validate state machines and data shapes before committing to an implementation.

**UI prototype** — several structurally different UI variants on one route, switchable via URL param. Used to pick a design before building the real thing.
