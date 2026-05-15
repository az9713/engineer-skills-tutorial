# What is this?

A collection of composable agent skills for real software engineering — not vibe coding. Each skill encodes a specific engineering discipline as a repeatable process an AI agent can follow.

---

## The problem these skills solve

AI coding agents fail in four predictable ways:

1. **Misalignment** — the agent builds the wrong thing because you and it never reached a shared understanding of the plan.
2. **Verbose language** — the agent uses 20 words where 1 will do, because it has no access to your project's domain vocabulary.
3. **No feedback loops** — the agent produces code that "looks right" but hasn't been tested against real behavior.
4. **Accelerated entropy** — because agents can write code fast, they also accumulate technical debt fast. A codebase can become a ball of mud in days.

These skills address each failure mode directly. They don't own the whole process — they're small, focused, and composable. You stay in control.

---

## Design philosophy

**Small and hackable.** Every skill is a Markdown file you can read and edit. There's no framework to learn, no schema to comply with. If a skill doesn't fit your workflow, change it.

**Based on engineering classics.** The skills draw on _The Pragmatic Programmer_, _A Philosophy of Software Design_, _Domain-Driven Design_, and _Extreme Programming Explained_. The concepts are battle-tested; the skills adapt them for AI-assisted development.

**Composable.** Skills are designed to chain. A typical session might be: `/grill-with-docs` (align) → `/to-prd` (document) → `/to-issues` (plan) → `/tdd` (build) → `/diagnose` (debug). Each skill hands off cleanly to the next.

**Explicit process over autonomy.** Rather than letting the agent decide how to approach a task, each skill defines the process. This makes the agent's behavior predictable and the process debuggable when something goes wrong.

---

## How skills work

A skill is a Markdown file in a named folder:

```
skills/
└── engineering/
    └── diagnose/
        ├── SKILL.md              ← main instructions the agent receives
        └── scripts/
            └── hitl-loop.template.sh   ← bundled resource
```

When you invoke `/diagnose`, the agent receives the content of `SKILL.md` as additional context. The frontmatter `description` field controls when the skill auto-triggers.

Skills can reference other files in the same folder. Those files are loaded lazily — only when the agent needs them. This keeps the initial context small while making depth available on demand.

---

## The three layers every skill uses

**Layer 1: Trigger** — the frontmatter `description` field tells the agent when to invoke the skill. A good description includes both what the skill does and the specific signals that mean it should activate ("Use when user says 'diagnose this', reports a bug...").

**Layer 2: Process** — the body of SKILL.md defines a workflow with phases, checklists, and explicit decision points. The agent follows these steps in order. Where human input is needed, the skill says so explicitly.

**Layer 3: Resources** — companion files (format specs, vocabulary files, script templates, example outputs) give the agent what it needs without bloating the main SKILL.md. These files are linked from SKILL.md and only loaded when relevant.

---

## The shared infrastructure

Five skills form the infrastructure the rest depend on:

- **[setup-matt-pocock-skills](../skills/engineering/setup-matt-pocock-skills.md)** — writes the per-repo config (issue tracker, triage labels, domain doc layout) that all engineering skills consume. Run once per repo.
- **[grill-with-docs](../skills/engineering/grill-with-docs.md)** — builds and maintains `CONTEXT.md` (the project glossary) and `docs/adr/` (decision records) that other skills read.
- **`CONTEXT.md`** — the shared language file. Agents use it to name things consistently with the project vocabulary.
- **`docs/adr/`** — the decision record. Skills check this before suggesting changes, to avoid re-litigating past decisions.
- **`docs/agents/`** — machine-readable config written by `setup-matt-pocock-skills`. Skills like `triage`, `to-issues`, and `diagnose` read from here.

---

## Typical skill lifecycle

For a complete feature, the typical sequence is:

```
1. /grill-with-docs       → Resolve the plan, update CONTEXT.md
2. /to-prd                → Publish a PRD to the issue tracker
3. /to-issues             → Break the PRD into vertical-slice issues
4. /tdd                   → Build one slice at a time with tests
5. /diagnose              → Fix any bugs that emerge
6. /improve-codebase-architecture → Refactor toward better design
```

Not every task needs all six. A bug report might only need `/diagnose`. A new feature in a familiar area might skip `/grill-with-docs` and go straight to `/tdd`.
