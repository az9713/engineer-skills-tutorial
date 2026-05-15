# setup-matt-pocock-skills

One-time per-repo scaffold that writes the configuration all other engineering skills consume. Run this first, before using `triage`, `to-issues`, `to-prd`, `tdd`, `diagnose`, `improve-codebase-architecture`, or `zoom-out`.

---

## The problem it solves

The engineering skills need to know three things about a repo that can't be inferred from code alone: where issues live, what label strings the issue tracker uses for triage states, and how domain documentation is laid out. Without this context, skills either fail silently or produce output that doesn't match the project's actual workflow.

This skill asks the maintainer those three questions once and writes the answers to durable files that all other skills can read.

---

## How it works

The skill is marked `disable-model-invocation: true` — it's a prompt-driven conversation, not an automated script. The agent explores, asks, confirms, and writes.

### Phase 1: Explore

The agent reads the repo to understand its current state without assuming anything:

- `git remote -v` and `.git/config` — detect if GitHub or GitLab is in use
- `AGENTS.md` and `CLAUDE.md` at root — find whether one already exists and whether it already has an `## Agent skills` section
- `CONTEXT.md`, `CONTEXT-MAP.md`, `docs/adr/`, `docs/agents/`, `.scratch/`

This prevents overwriting prior setup or creating duplicate configuration.

### Phase 2: Walk through three decisions

The agent presents each decision one at a time, with a plain-English explainer before each choice. The three decisions are:

**A. Issue tracker** — GitHub Issues (via `gh` CLI), GitLab Issues (via `glab` CLI), local markdown files under `.scratch/`, or a custom workflow described by the user.

**B. Triage label vocabulary** — the five canonical triage states (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) mapped to the actual label strings used in the issue tracker. If the repo uses `bug:triage` instead of `needs-triage`, the mapping goes here.

**C. Domain doc layout** — single-context (one `CONTEXT.md` at root + `docs/adr/`) or multi-context (`CONTEXT-MAP.md` pointing to per-context directories, typical of monorepos).

### Phase 3: Confirm and write

The agent shows a draft of all changes before writing:
- The `## Agent skills` block to add to `CLAUDE.md` or `AGENTS.md`
- Contents of `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, `docs/agents/domain.md`

The user can edit the draft before writing. Once confirmed, the agent writes the files.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Main process with all three sections and file-selection rules |
| `issue-tracker-github.md` | Seed template for GitHub issue tracker config |
| `issue-tracker-gitlab.md` | Seed template for GitLab issue tracker config |
| `issue-tracker-local.md` | Seed template for local markdown issue tracker config |
| `triage-labels.md` | Seed template for label vocabulary mapping |
| `domain.md` | Seed template for domain doc consumer rules |

The seed templates are starting points. The agent adapts them based on what the user provides.

---

## Mechanism

The skill solves the "environment mismatch" problem that makes skills fail silently. Without `docs/agents/`, a skill like `/triage` doesn't know whether to call `gh issue list` or look in `.scratch/`. With it, each skill reads a small, specific file that answers exactly the question it needs to answer.

The three `docs/agents/` files are machine-readable prose — not structured config. This means the agent can read nuance ("issues are tracked in Linear, but we use GitHub for public-facing bugs") rather than being forced into a rigid schema.

The `## Agent skills` block in `CLAUDE.md` acts as a pointer. Skills that run later see "See `docs/agents/issue-tracker.md`" and read the full details there, rather than polluting the main agent config file with verbose config.

---

## How to improve

**Show an example of the completed output.** The skill describes what it will write but doesn't show a complete before/after example. Adding a concrete example of a finished `## Agent skills` block and a sample `docs/agents/issue-tracker.md` would reduce uncertainty about what the setup produces.

**Add an update path.** The skill says "re-running is only necessary if you want to switch issue trackers" — but it doesn't explain what "re-running" looks like. Does the agent overwrite or merge? A short "Updating an existing setup" section would make this less ambiguous.

**Handle the `docs/agents/` directory creation.** The skill writes files to `docs/agents/` but doesn't explicitly mention creating the directory if it doesn't exist. A small note or an explicit `mkdir -p docs/agents/` would prevent failure when the directory is missing.

**Add a verification step.** After writing, the agent could read one of the files back to confirm it was written correctly — the same way `diagnose` verifies the fix works before declaring done.
