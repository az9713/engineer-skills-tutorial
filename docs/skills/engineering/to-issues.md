# to-issues

Breaks a plan, spec, or PRD into independently-grabbable issues on the project issue tracker using vertical slices (tracer bullets). Each issue is a thin, complete path through all integration layers.

---

## The problem it solves

Plans get broken into tasks in one of two problematic ways: either in large chunks ("implement authentication") that are too big for a single agent session, or in horizontal layers ("write the database schema," "write the API handlers," "write the UI") that can't be verified until all layers are complete.

This skill enforces vertical slicing: each issue is thin but complete, cutting through every layer end-to-end, deliverable and demoable on its own.

---

## How it works

### Phase 1: Gather context

Work from whatever is already in the conversation. If the user passes an issue reference, fetch it from the tracker and read the full body and comments.

### Phase 2: Explore the codebase

If the codebase hasn't been explored, do so now. Issue titles and descriptions must use the project's domain vocabulary (from `CONTEXT.md`) and respect ADRs.

### Phase 3: Draft vertical slices

Each issue is a **tracer bullet**: a thin vertical slice that cuts through ALL integration layers end-to-end. Not a horizontal slice of one layer.

Three rules enforced:

| Rule | Meaning |
|------|---------|
| Complete path | Each slice delivers a narrow but complete path through every layer (schema, API, UI, tests) |
| Demoable | A completed slice is demoable or verifiable on its own |
| Thin | Prefer many thin slices over few thick ones |

Each slice is classified as **AFK** (can be implemented and merged without human interaction) or **HITL** (requires human interaction — architectural decision, design review, external access).

### Phase 4: Quiz the user

Present the breakdown as a numbered list showing:
- **Title** — short descriptive name
- **Type** — HITL or AFK
- **Blocked by** — which other slices (if any) must complete first
- **User stories covered** — which user stories from the PRD this addresses

Ask the user: Is the granularity right? Are dependency relationships correct? Should any slices be merged or split? Are HITL/AFK classifications correct?

Iterate until the user approves.

### Phase 5: Publish in dependency order

Publish issues in dependency order (blockers first) so real issue identifiers can be referenced in the "Blocked by" field.

Each issue uses a fixed template:

```
## Parent
Reference to the parent issue (if applicable)

## What to build
Concise description of this vertical slice.
Describe end-to-end behavior, not layer-by-layer implementation.

## Acceptance criteria
- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by
- Reference to blocking ticket (or "None - can start immediately")
```

The instruction to describe end-to-end behavior rather than layer-by-layer implementation is the critical constraint. An issue that says "add `schedule` column to database" is a horizontal slice. An issue that says "users can see their next scheduled run on the dashboard" is vertical.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Process + vertical slice rules + issue template |

No companion files. The template is embedded in SKILL.md.

---

## Mechanism

Vertical slicing solves the integration problem. A horizontal slice ("write the database schema") can't be tested or demonstrated until the layers above it are built. A vertical slice ("users can view their profile") is testable the moment it's merged, because it goes all the way to the user-visible surface.

The AFK/HITL classification ensures that AFK agents don't get assigned issues requiring judgment calls they can't make. HITL issues get queued for the next human session rather than being handed to an agent that will stall or guess.

Publishing in dependency order with real issue references means each issue's "Blocked by" field contains an actual tracker link, not a description — making the dependency graph actionable rather than informational.

---

## How to improve

**Add guidance on issue title format.** The template says "short descriptive name" but doesn't specify format. Should titles be user-story format ("Users can view profile"), imperative ("Add user profile view"), or noun ("User profile view")? A consistent format makes the issue list scannable. Imperative format aligns with the tracer-bullet metaphor.

**Define "too small" and "too large" for a slice.** The skill says "prefer many thin slices over few thick ones" but doesn't define bounds. A guideline like "a slice should be implementable in one agent session (roughly 2–4 hours of work)" would calibrate slice sizing.

**Add guidance on what belongs in acceptance criteria vs what to build.** "What to build" describes the end-to-end behavior; "Acceptance criteria" describes how to verify it. The distinction is subtle. An example showing the same feature expressed as both would clarify the difference.

**Add a note about not modifying the parent issue.** The skill says "Do NOT close or modify any parent issue" at the very end. This is an important constraint that's easy to miss. Moving it to the beginning of Phase 5 would make it more prominent.
