# to-prd

Synthesizes the current conversation context and codebase understanding into a Product Requirements Document, then publishes it to the issue tracker. No interview — it works from what's already been discussed.

---

## The problem it solves

After a grilling session with `/grill-with-docs`, you have a plan in your head and in the conversation history. But conversation context doesn't persist across sessions and isn't searchable. This skill captures the plan as a structured, permanent artifact in the issue tracker, ready for agents to work from.

The "no interview" constraint is intentional: by the time you run `/to-prd`, you've already discussed the feature. The skill synthesizes what's been said rather than asking again.

---

## How it works

### Phase 1: Explore the codebase

If the agent hasn't already explored the codebase, it does so now, using the project's `CONTEXT.md` domain vocabulary throughout. This ensures the PRD uses the project's actual language rather than generic terms.

### Phase 2: Sketch modules and check in

The agent sketches the major modules it will need to build or modify, actively looking for opportunities to extract deep modules (small interface, large implementation). It checks with the user that these modules match expectations, and asks which modules should have tests written for them.

This is the only conversation checkpoint in the skill — everything else is synthesis and publishing.

### Phase 3: Write and publish the PRD

The PRD uses a fixed template with six sections:

| Section | Content |
|---------|---------|
| **Problem Statement** | The problem from the user's perspective |
| **Solution** | The solution from the user's perspective |
| **User Stories** | Extensive numbered list in "As a X, I want Y, so that Z" format |
| **Implementation Decisions** | Modules to build/modify, interfaces, architectural decisions, schema changes |
| **Testing Decisions** | What makes a good test, which modules to test, prior art |
| **Out of Scope** | Explicit exclusions |

The PRD is published to the issue tracker with the `ready-for-agent` triage label. No further triage needed.

### What goes in Implementation Decisions

Modules, interface shapes, technical clarifications, architectural decisions, schema changes, API contracts, specific interactions. **Not** file paths or code snippets — they go stale. Exception: if a `/prototype` session produced a snippet that encodes a decision more precisely than prose can (a state machine, reducer, schema, type shape), inline it with a note that it came from a prototype.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Process description + PRD template with section-by-section guidance |

No companion files. The skill is self-contained.

---

## Mechanism

The skill is a synthesis tool, not an exploration tool. Its value is converting implicit knowledge (in the conversation) into explicit knowledge (in the tracker). The strict "no interview" constraint prevents it from becoming a second `/grill-with-docs` session — that work should already be done.

The "active look for deep modules" step in Phase 2 introduces architectural intent into the PRD. Rather than just describing features, the PRD captures the module structure the implementation will target. This makes the resulting issues (from `/to-issues`) more coherent because they can be sliced around module boundaries.

The `ready-for-agent` label on publish is an implicit assertion: by the time this skill runs, the feature is specified well enough that an agent could implement it. If it's not, run `/grill-with-docs` first.

---

## How to improve

**Add guidance on PRD length.** The template calls for "a LONG, numbered list of user stories" and "extremely extensive" coverage, but gives no guidance on what a complete vs minimal PRD looks like. A note like "a typical PRD for a medium feature should have 10–20 user stories and 5–8 implementation decisions" would calibrate expectations.

**Standardize the Implementation Decisions format.** The section is described as "a list of implementation decisions" but each item can be a module, an interface shape, an architectural decision, a schema change, or an API contract. These are quite different. Giving each type a consistent sub-format — "**Module**: OrderProcessor — encapsulates X, takes Y as input, returns Z" — would make the PRD more scannable.

**Add a validation step before publishing.** The skill jumps directly from module sketch → PRD write → publish. A brief self-check — "does this PRD have enough in Implementation Decisions for an agent to start without asking questions?" — would catch underspecified PRDs before they land in the tracker as `ready-for-agent`.

**Link back to the grilling session.** When a PRD is published after a `/grill-with-docs` session, it's useful to link the PRD to the context that produced it. The agent could add a "Context" section to the PRD referencing the CONTEXT.md version and any ADRs created during the grilling session.
