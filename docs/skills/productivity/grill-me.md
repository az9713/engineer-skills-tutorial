# grill-me

Relentlessly interviews you about a plan or design until every branch of the decision tree is resolved. The base grilling skill — not code-specific.

---

## The problem it solves

When you describe a plan to an agent and ask it to execute, you've often left unstated assumptions that will cause problems mid-implementation. You don't know what you don't know. The grilling session surfaces those gaps before any code is written.

---

## How it works

The SKILL.md is three sentences:

1. Interview me relentlessly about every aspect of this plan until we reach a shared understanding.
2. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.
3. Ask the questions one at a time. If a question can be answered by exploring the codebase, explore the codebase instead.

Three behavioral directives:

**"Relentlessly"** — the agent doesn't stop when it has enough for a guess. It stops when the design tree is fully resolved.

**"Provide your recommended answer"** — the agent commits to a position for each question. This prevents the session from becoming a passive exchange where the human has to decide everything. The agent says "I recommend X because Y — do you agree?" The human either agrees (fast) or redirects (clarifying).

**"One at a time"** — prevents question batches that are hard to answer. Each question gets a response before the next is asked.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Three-sentence instruction |

No companion files.

---

## The difference between grill-me and grill-with-docs

`/grill-me` is general-purpose: it works for any plan, code or not. It doesn't touch `CONTEXT.md` or ADRs.

`/grill-with-docs` is the engineering-flavored version: it adds domain awareness (challenges against the glossary, cross-references with code) and writes to `CONTEXT.md` and `docs/adr/` inline. Use `/grill-me` for non-code plans; use `/grill-with-docs` when the plan involves code.

---

## Mechanism

The skill is minimal by design. The grilling behavior depends entirely on the model's ability to ask good questions — the skill's job is only to activate that behavior with the right framing. "Relentlessly" and "one at a time" are the two constraints that most commonly need to be stated explicitly; without them, agents tend to ask three questions at once and stop too soon.

---

## How to improve

**Add an explicit termination condition.** The skill doesn't say when grilling is done. Adding "end when all branches of the design tree are resolved and no question remains that would affect implementation decisions" would make it clear when to stop.

**Add guidance on scope.** For a large system, "every aspect of this plan" could mean hundreds of questions. A note like "focus on decisions that are hard to reverse or that affect how other parts of the plan will be designed" would help prioritize.

**Add a post-grilling summary step.** After the grilling ends, the agent could produce a one-paragraph summary of the key decisions made and assumptions confirmed. This gives the human a quick way to verify the session produced useful output before moving to implementation.
