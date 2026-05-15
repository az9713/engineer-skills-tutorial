# diagnose

A six-phase discipline for hard bugs and performance regressions. The defining constraint: do not hypothesize without a feedback loop. Every other phase serves the loop.

---

## The problem it solves

Hard bugs resist casual debugging. The typical failure mode is "stare at code, form a hypothesis, apply a fix, test manually, repeat until something sticks." This is slow and produces fixes that mask symptoms rather than eliminating causes.

The skill replaces that loop with a structured process: build a fast, deterministic feedback signal first, then use that signal to bisect toward the cause systematically. The feedback loop is the skill — everything else is mechanical once you have it.

---

## How it works

### Phase 1: Build a feedback loop (the core discipline)

The skill opens with "This is the skill. Everything else is mechanical." The agent is directed to spend disproportionate effort here and to be "aggressive, creative, refuse to give up."

Ten strategies for constructing a loop are listed in order of preference:

1. Failing test at whatever seam reaches the bug
2. Curl/HTTP script against a running dev server
3. CLI invocation with a fixture, diffing against a known-good snapshot
4. Headless browser script (Playwright/Puppeteer)
5. Replay a captured trace (network request, event log)
6. Throwaway harness spinning up a minimal subset of the system
7. Property/fuzz loop for "sometimes wrong output" bugs
8. Bisection harness for bugs that appeared between two known states
9. Differential loop comparing old vs new version
10. HITL bash script (human-driven loop using the bundled template) — last resort

Once a loop exists, the skill asks the agent to improve it: make it faster, sharpen the signal, and increase determinism. "A 30-second flaky loop is barely better than no loop. A 2-second deterministic loop is a debugging superpower."

For non-deterministic bugs, the goal isn't a clean repro but a higher reproduction rate — run 1000×, add stress, narrow timing windows.

If no loop can be built, the skill says to stop and ask for artifacts or production access. Do not proceed to Phase 2 without a loop.

### Phase 2: Reproduce

Run the loop. Confirm three things: the failure matches what the user described (not a different nearby failure), it's reproducible across multiple runs, and the exact symptom is captured.

### Phase 3: Hypothesize

Generate 3–5 ranked hypotheses before testing any. Each hypothesis must be falsifiable — the prediction is stated as "If X is the cause, then changing Y will make the bug disappear." Single-hypothesis generation anchors on the first plausible idea; generating 3–5 first forces a survey of the space.

Show the ranked list to the user before testing — they often have domain knowledge that re-ranks instantly. "Don't block on it — proceed with your ranking if the user is AFK."

### Phase 4: Instrument

Each probe maps to a specific hypothesis prediction. Change one variable at a time. Tool preference: debugger/REPL first, targeted logs second, never "log everything and grep."

Debug logs are tagged with a unique prefix (`[DEBUG-a4f2]`) so they can be removed with a single grep at the end.

For performance regressions, logs are usually wrong — use a timing harness and bisect.

### Phase 5: Fix and regression test

Write the regression test before the fix — but only at a "correct seam" (one where the test exercises the real bug pattern as it occurs at the call site).

If no correct seam exists, that itself is the finding. It signals that the codebase architecture is preventing the bug from being locked down. This is passed to `/improve-codebase-architecture` after the fix.

### Phase 6: Cleanup and post-mortem

Required before declaring done:
- Original repro no longer reproduces
- Regression test passes (or absence of seam is documented)
- All debug instrumentation removed
- Throwaway prototypes deleted
- The correct hypothesis is stated in the commit/PR message

The post-mortem question: "What would have prevented this bug?" If the answer involves architectural change, hand off to `/improve-codebase-architecture` with specifics.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Six-phase process with checklists and tool preferences |
| `scripts/hitl-loop.template.sh` | Template bash script for human-in-the-loop reproduction loops |

### hitl-loop.template.sh in detail

The template provides two helpers: `step "instruction"` (shows instruction, waits for Enter) and `capture VAR "question"` (reads user input into a variable). The script ends by printing all captured values as `KEY=VALUE` pairs for the agent to parse.

This solves a specific problem: some bugs can only be reproduced through UI interaction. Rather than having the agent and human communicate in prose about what happened, the template structures the interaction into a machine-parseable loop.

---

## Mechanism

The six phases prevent the most common failure modes in debugging:

- **Feedback loop first** prevents the "fix things randomly" failure mode
- **Multiple hypotheses** prevents anchoring on the first plausible idea
- **One variable at a time** makes the probe-outcome relationship interpretable
- **Tagged debug logs** makes cleanup deterministic
- **Post-mortem handoff** closes the loop with the architecture skill

The ordering is not arbitrary. Each phase is a gate — the skill explicitly says "do not proceed to Phase 2 until you have a loop" and "do not proceed until you reproduce the bug." These gates prevent the agent from skipping to Phase 5 because it "looks like" it knows the cause.

---

## How to improve

**Add a visual flow diagram.** The six phases are described in prose with checklists. A simple numbered flowchart showing the phases and their gates ("cannot proceed without loop," "cannot proceed without repro") would make the structure clearer at a glance.

**Make Phase 5's "correct seam" concept concrete.** The concept of a "correct seam" is explained in abstract terms. Adding an example — "a unit test of `getDiscount()` is NOT a correct seam if the bug only manifests when `getDiscount()` is called from `checkout()` with a specific cart state" — would make the gate actionable.

**Add explicit guidance for wrong-hypothesis recovery.** The skill doesn't say what to do when a hypothesis is tested and disproven. Adding a "when your hypothesis fails" step (go back to Phase 3, cross off the disproved hypothesis, rank the remaining ones, update based on what the failed probe revealed) would make the loop more explicit.

**Cross-link the hitl script more prominently.** `hitl-loop.template.sh` is mentioned in Phase 1 as a last resort but isn't linked. Adding a direct link `[hitl-loop.template.sh](scripts/hitl-loop.template.sh)` makes it easy to copy.

**Add a performance regression specialization.** Phase 4 mentions "Perf branch" in a sub-section, but performance debugging is different enough from correctness debugging to warrant a more prominent split. A "When the bug is a perf regression, start here" callout at the top would help.
