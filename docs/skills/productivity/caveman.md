# caveman

Ultra-compressed communication mode. Drops articles, filler words, pleasantries, and hedging while keeping full technical accuracy. Triggered by "caveman mode" / "less tokens" / `/caveman`.

---

## The problem it solves

Agent responses often pad technical content with verbal overhead: "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..." This costs tokens and slows reading. Caveman mode eliminates the padding while preserving the substance.

---

## How it works

The SKILL.md is itself written in caveman mode — it's a self-demonstrating skill.

### What gets dropped

- Articles: a, an, the
- Filler: just, really, basically, actually, simply
- Pleasantries: sure, certainly, of course, happy to
- Hedging (excessive qualifiers)
- Conjunctions that can be cut
- Verbose forms: "in order to" → "to", "implement a solution for" → "fix"

### What stays exact

- Technical terms
- Code blocks (unchanged)
- Error messages (quoted exactly)

### The response pattern

`[thing] [action] [reason]. [next step].`

Instead of: "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..."  
Say: "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix:"

### Persistence

Active for every response once triggered. Does not revert after many turns. Off only when user says "stop caveman" or "normal mode."

### Auto-Clarity Exception

Caveman drops temporarily for:
- Security warnings
- Irreversible action confirmations
- Multi-step sequences where fragment order risks misread
- When the user asks for clarification or repeats a question

Resume caveman after the clear part is done.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Rules for what to drop, what to keep, persistence, and the auto-clarity exception |

No companion files.

---

## Mechanism

The skill's self-demonstrating format (written in caveman mode) serves two purposes. First, it shows the agent exactly what the mode looks like in practice rather than describing it abstractly. Second, it signals that the skill is itself subject to the mode — establishing that caveman applies to skill content, not just conversation content.

The persistence rule ("never revert after many turns") solves the drift problem: agents naturally drift back toward verbose patterns as the conversation continues. Making persistence explicit prevents this.

The Auto-Clarity Exception solves the safety problem: some content (destructive actions, security warnings) needs to be understood precisely, not just efficiently. The exception is narrow and specific — it doesn't create a loophole for verbose responses in general, just for content where misreading has consequences.

---

## How to improve

**Add intensity levels to the trigger.** The SKILL.md defines one mode, but there's a spectrum from "slightly more terse" to "extreme abbreviation." Some users might want to drop pleasantries but keep articles. Adding "caveman lite" (drop pleasantries only) and "caveman ultra" (maximum compression including abbreviations like DB/auth/req/res) would give more control.

**Add an explicit list of standard abbreviations.** The skill says "Abbreviate common terms (DB/auth/config/req/res/fn/impl)" but this list is short. A more complete list of approved abbreviations would make the compression more consistent across sessions.

**Add a counter-example gallery.** The skill has two examples (React re-render and database pooling) but these cover simple explanations. Adding examples for more complex outputs — a multi-step process, a comparison table, a bug diagnosis — would demonstrate that caveman works for all response types, not just one-liners.
