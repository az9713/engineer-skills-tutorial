# setup-pre-commit

Sets up Husky pre-commit hooks with lint-staged (Prettier on all staged files), type checking, and tests in a Node.js project. Run once per repo.

---

## The problem it solves

Code quality checks that only run in CI catch problems too late — after a PR is open and reviewers are waiting. Pre-commit hooks catch formatting, type errors, and test failures before the commit lands, without requiring any manual discipline.

---

## How it works

The skill is a deterministic seven-step recipe.

### Step 1: Detect package manager

Check for lockfiles: `package-lock.json` (npm), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), `bun.lockb` (bun). Use whichever is present.

### Step 2: Install dependencies

Install as devDependencies: `husky lint-staged prettier`.

### Step 3: Initialize Husky

```bash
npx husky init
```

Creates `.husky/` and adds `prepare: "husky"` to `package.json`.

### Step 4: Create `.husky/pre-commit`

```
npx lint-staged
npm run typecheck
npm run test
```

Adapts `npm` to the detected package manager. If the repo has no `typecheck` or `test` script in `package.json`, omits those lines and tells the user.

### Step 5: Create `.lintstagedrc`

```json
{ "*": "prettier --ignore-unknown --write" }
```

Runs Prettier on all staged files regardless of extension. `--ignore-unknown` skips files Prettier can't parse (images, etc.).

### Step 6: Create `.prettierrc` (if missing)

Only if no Prettier config exists. Uses sensible defaults (2-space indent, double quotes, trailing commas, 80-char print width).

### Step 7: Verify and commit

Run `npx lint-staged` to verify the setup works. Stage all created files and commit with the message `Add pre-commit hooks (husky + lint-staged + prettier)`. This commit runs through the new pre-commit hooks, providing a smoke test.

---

## Associated assets

| File | What it contains |
|------|-----------------|
| `SKILL.md` | Seven-step recipe with exact file contents and package manager detection |

No companion files.

---

## Mechanism

The skill's effectiveness comes from the self-validating final step: the commit that installs the hooks is the first commit that runs through them. If the hooks are misconfigured, the commit fails immediately — catching the problem before the user has to debug "why isn't my pre-commit hook running."

The `--ignore-unknown` flag on Prettier is an important detail: without it, Prettier will error on file types it doesn't recognize (`.env`, images, binary files), breaking the pre-commit hook for any commit that touches those files.

---

## How to improve

**Add guidance on emergency commit bypass.** Pre-commit hooks can be skipped with `git commit --no-verify`. The skill doesn't mention this. A brief note — "To skip hooks in emergencies: `git commit --no-verify`. Avoid this habitually." — would be useful.

**Handle the case where `typecheck` or `test` scripts are named differently.** The skill omits lines when scripts don't exist, but many repos use names like `tsc`, `check`, `lint`, or `jest` instead of `typecheck` and `test`. A step that checks the `scripts` section of `package.json` for similar names would make the hook more complete.

**Add a note about Husky v9 shebang behavior.** The skill notes "Husky v9+ doesn't need shebangs in hook files." For users familiar with older Husky, this is a common point of confusion. Making this more prominent (e.g. "If you're upgrading from Husky v8, remove the `#!/bin/sh` line from hook files") would prevent a common setup issue.
