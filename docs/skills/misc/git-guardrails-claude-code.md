# git-guardrails-claude-code

Installs a Claude Code `PreToolUse` hook that intercepts and blocks dangerous git commands before the agent executes them.

---

## The problem it solves

Claude Code can execute git commands autonomously. Some git commands are destructive and irreversible: `git push --force`, `git reset --hard`, `git clean -f`. Without guardrails, an agent running in auto-approve mode can push to the wrong branch, hard-reset working changes, or clean untracked files — without any human confirmation.

This skill installs a bash hook that intercepts these commands before they run and exits with a "BLOCKED" message that the agent can read.

---

## How it works

### Phase 1: Ask scope

Project-level (`.claude/settings.json`) or global (`~/.claude/settings.json`)?

### Phase 2: Copy the hook script

Copy `scripts/block-dangerous-git.sh` to the target location:
- Project: `.claude/hooks/block-dangerous-git.sh`
- Global: `~/.claude/hooks/block-dangerous-git.sh`

Make it executable with `chmod +x`.

### Phase 3: Add hook to settings

Add a `PreToolUse` hook to the settings file pointing to the script. If the settings file already exists, merge into the existing `hooks.PreToolUse` array — don't overwrite other settings.

The hook intercepts `Bash` tool calls, meaning it runs before any shell command Claude Code executes.

### Phase 4: Ask about customization

Ask if the user wants to add or remove patterns from the blocked list.

### Phase 5: Verify

Test the hook:

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

Should exit with code 2 and print a BLOCKED message to stderr.

---

## What gets blocked

- `git push` (all variants, including `--force`)
- `git reset --hard`
- `git clean -f` and `git clean -fd`
- `git branch -D`
- `git checkout .` and `git restore .`

When blocked, the agent sees: "Claude Code does not have authority to run this command."

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Five-step process with JSON hook config snippets |
| `scripts/block-dangerous-git.sh` | The bash hook script that does the actual blocking |

### block-dangerous-git.sh

Reads the JSON input from Claude Code's tool call, extracts the command, and pattern-matches it against the blocked list. Exits 0 (allow) or 2 (block) with a human-readable message.

---

## Mechanism

Claude Code's `PreToolUse` hook is a bash script that runs before any tool call. It receives the tool input as JSON on stdin. If the script exits with a non-zero code, the tool call is blocked and the exit message is shown to the agent.

The hook is registered under the `Bash` matcher — it only fires for `Bash` tool calls. This means non-shell tools (Read, Write, Edit) are unaffected.

---

## How to improve

**Handle missing settings.json.** The skill's settings JSON snippets assume the file exists. If it doesn't, the agent needs to create it first. A note — "if the file doesn't exist, create it with the JSON structure shown" — would prevent a common failure case.

**Add guidance on combining with other hooks.** If the user already has other `PreToolUse` hooks, the merge instruction ("merge into existing array") is correct but brief. Showing the merged JSON structure explicitly would prevent array nesting mistakes.

**Add custom pattern documentation.** Phase 4 asks about customization but doesn't explain what format custom patterns take. Is it a regex? A prefix match? Showing the pattern format in the hook script's comments (it currently just lists specific commands) would make customization straightforward.

**Add a global uninstall path.** The skill explains installation but not removal. A "Remove guardrails" section with the reverse steps (remove hook from settings, delete script file) would make the skill complete.
