# grill-with-docs

A structured interview that stress-tests a plan against the project's existing domain model, sharpens terminology, and updates `CONTEXT.md` and `docs/adr/` inline as decisions crystallize. The engineering-flavored version of `/grill-me`.

---

## The problem it solves

When you start building a feature, two kinds of misalignment exist. First, you and the agent might have different understandings of what to build (the classic "not what I meant" problem). Second, the terms you use might not match the project's established vocabulary — so the agent will name new code inconsistently, making the codebase harder to navigate.

This skill solves both. It runs a grilling session, but during that session it also reads and updates the project's domain documentation. The output is not just a clearer plan — it's a plan expressed in the project's actual language, with new terms captured and decisions recorded.

---

## How it works

The skill has two parts: the instruction block (`<what-to-do>`) and the supporting context block (`<supporting-info>`).

### The interview

The `<what-to-do>` block tells the agent to interview relentlessly, one question at a time, walking down every branch of the design tree. For each question the agent provides its recommended answer — this prevents the session from becoming a passive "what do you think?" exchange and forces the agent to commit to a position the user can accept or push back on.

If a question can be answered by reading the codebase, the agent reads the codebase instead of asking. This keeps the interview focused on genuine ambiguity.

### Domain awareness during the session

The `<supporting-info>` block tells the agent how to behave while grilling:

**Challenge against the glossary.** If the user uses a term that conflicts with an existing entry in `CONTEXT.md`, the agent calls it out immediately. This prevents vocabulary drift.

**Sharpen fuzzy language.** When the user says "account," the agent checks whether they mean `Customer` or `User`. If they're different concepts in this project's domain, the agent surfaces the distinction.

**Stress-test with scenarios.** Domain relationships are probed with concrete edge-case scenarios. "What happens when a Customer cancels an Order that's already been fulfilled?" — this kind of pressure reveals ambiguity that prose descriptions hide.

**Cross-reference with code.** If the user describes how something works, the agent checks whether the code agrees. Contradictions between stated behavior and actual code are surfaced immediately.

**Inline updates.** When a term is resolved, `CONTEXT.md` is updated on the spot. This is the "inline" discipline — the skill explicitly says "capture them as they happen." The resulting document is always current, not a post-session reconstruction.

### ADR creation

ADRs are offered sparingly — only when all three are true: the decision is hard to reverse, surprising without context, and the result of a real trade-off. The `<supporting-info>` block gives this three-condition check explicitly. This prevents ADR sprawl (where every small choice gets a document) while ensuring genuinely important decisions are recorded.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Interview instruction + domain awareness rules |
| `CONTEXT-FORMAT.md` | Format spec for `CONTEXT.md` — structure, rules, single vs multi-context repos |
| `ADR-FORMAT.md` | Format spec for ADRs — template, optional sections, when to offer one |

Both format files are referenced by other skills (`improve-codebase-architecture` links to `CONTEXT-FORMAT.md`). They function as shared standards for domain documentation across the skill collection.

### CONTEXT-FORMAT.md in detail

The format requires:
- A one-line context description
- A `## Language` section with bold terms, one-sentence definitions, and `_Avoid_` aliases
- A `## Relationships` section showing cardinality between concepts
- A `## Example dialogue` section — a short conversation demonstrating the terms in use
- A `## Flagged ambiguities` section for terms that were ambiguous and have now been resolved

The example dialogue requirement is the most important design choice here. It forces the format to include not just definitions but *how* the terms behave when used in actual conversation — which is what an agent needs to use them correctly.

### ADR-FORMAT.md in detail

ADRs are intentionally minimal: a title, 1–3 sentences of context/decision/reason, and optional sections only when they add genuine value. The format explicitly rejects the heavyweight Nygard template (Status, Context, Decision, Consequences) in favor of a single paragraph. The reasoning: most decisions don't need a full document — they need a sentence that tells the next engineer why things are the way they are.

---

## Mechanism

The power of this skill is the combination of two things happening in the same session: alignment and documentation. Most grilling sessions produce clarity in the agent's memory but leave no artifact. This skill externalizes that clarity into `CONTEXT.md` and `docs/adr/` as it happens, so future sessions (and future engineers) benefit.

The inline-update discipline is the key. If updates were batched at the end of the session, the agent would be less precise (reconstructing decisions from memory) and the user would be less engaged (reviewing a draft instead of decisions made 10 minutes ago).

---

## How to improve

**Add an explicit termination condition.** The skill says "interview relentlessly" but never says when the session is done. Adding a closing criterion — "end when all branches of the design tree have been resolved and CONTEXT.md accurately reflects the current plan" — would make it clearer when to stop.

**Specify the minimum viable session.** For smaller changes, a full deep-dive is overkill. A note like "for changes that touch only one existing concept, a 3–5 question session is sufficient" would help calibrate the depth of grilling to the scope of the change.

**Add guidance on the order of questions.** Currently the agent walks "down each branch" without guidance on which branch to prioritize. Suggesting an ordering (data model first, then API contracts, then UI/UX, then edge cases) would produce more consistent session structure.

**Make the cross-reference step more specific.** "Check whether the code agrees" is vague. Specifying what to look for — type definitions for domain concepts, function names that use the term, test names — would make this step produce more actionable output.
