---
name: bigin-harness-setup
description: "Scaffolds BigIn AI workflow harness into a repo — CLAUDE.md, governance rules, and enforcement gates. MUST use when user says: 'set up harness', 'add AI rules', 'scaffold harness', 'add CLAUDE.md', 'initialize AI workflow', 'set up claude rules', 'thiết lập harness', 'cài harness', 'thêm AI rules', or when onboarding an existing repo for structured AI-assisted development. Supports nuxt, go, nodejs profiles."
---

# bigin-harness-setup

Sets up a standardized AI workflow harness: governance files, scoped rules, and enforcement gates. Idempotent — re-running on an already-set-up repo is safe.

*Thiết lập harness quy trình AI: tài liệu quản trị, rules theo phạm vi, và cổng kiểm soát. Idempotent — chạy lại trên repo đã cài đặt là an toàn.*

---

## Phase 0: Detect Stack Profile / Phát hiện profile

Check for stack indicators:
1. `nuxt.config.ts` or `nuxt.config.js` → profile = `nuxt`
2. `go.mod` → profile = `go`
3. `package.json` with express/fastify/hono/koa in dependencies → profile = `nodejs`
4. None found or ambiguous → ask:

```
Which stack profile?
1. nuxt   — Nuxt 4 SPA, Pinia, VueUse, Nuxt UI, Vitest, Zod, Cloudflare Pages
2. go     — Go REST API backend
3. nodejs — Node.js TypeScript REST API backend

Type 1, 2, or 3.
```

Store result as `PROFILE`. Load `references/profile-{PROFILE}.md` for all template content.

---

## Phase 1: Detect Existing Harness / Phát hiện harness hiện có

Check for existing harness files:
```
CLAUDE.md | AI_TASK_GUIDE.md | AI_REVIEW_CHECKLIST.md | .claude/rules/
```

If any exist, show what was found and ask:
```
Found existing harness files: [list them]

Overwrite all? (yes) / Create missing only? (new) / Cancel? (cancel)
```

- `yes` → overwrite all (show what will be replaced before writing)
- `new` → create only files that don't exist; skip existing ones silently
- `cancel` → stop immediately

Store choice as `INSTALL_MODE`.

---

## Phase 2: Generate CLAUDE.md

Read the content from `references/profile-{PROFILE}.md` → `## CLAUDE.md Template` section.

Write to `CLAUDE.md` in the project root.
Skip if `INSTALL_MODE=new` and `CLAUDE.md` already exists.

---

## Phase 3: Generate .claude/rules/

Create `.claude/rules/` if it doesn't exist.

Generate three files (each: skip if `INSTALL_MODE=new` and already exists):

**conventions.md** — from `references/profile-{PROFILE}.md` → `## conventions.md Template`

**security.md** — from `references/files-shared.md` → `## security.md`

**architecture.md** — from `references/files-shared.md` → `## architecture.md`, then append the profile block from `references/profile-{PROFILE}.md` → `## architecture addendum`

---

## Phase 4: Generate AI Files

**AI_TASK_GUIDE.md** — from `references/files-shared.md` → `## AI_TASK_GUIDE.md`. Write to project root.

**AI_REVIEW_CHECKLIST.md** — from `references/files-shared.md` → `## AI_REVIEW_CHECKLIST.md`. Replace `{COMMANDS}` with the profile's lint/typecheck/test commands (from `references/profile-{PROFILE}.md` → `## Commands`).

Skip each if `INSTALL_MODE=new` and file already exists.

---

## Phase 5: Generate Enforcement / Tạo cơ chế thực thi

### 5-1. Pre-commit hook

Read from `references/hook-guard.md` → `## pre-commit: {PROFILE}`.

Write to `scripts/pre-commit.sh`, then make it executable: `chmod +x scripts/pre-commit.sh`.

### 5-1b. Initialize git + install the hook

The hook lives in `.git/hooks/`, so a git repo must exist first.

1. **Ensure a git repo.** Check with `git rev-parse --is-inside-work-tree 2>/dev/null`.
   - If it fails (not a repo), run `git init` and tell the user a repo was initialized.
   - If it already is a repo, do nothing.

2. **Install the hook** (idempotent — never clobber a foreign hook silently):
   - If `.git/hooks/pre-commit` does not exist, or is already a symlink to `../../scripts/pre-commit.sh` → install/refresh it:
     ```sh
     ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit
     ```
   - If `.git/hooks/pre-commit` exists and is **not** our symlink (a real file or a different target) → do NOT overwrite. Show it and ask:
     ```
     A pre-commit hook already exists at .git/hooks/pre-commit.
     Replace it with the harness hook? (yes / no — leave it and I'll note it in the summary)
     ```

3. Confirm to the user that the hook is installed (or was left untouched).

> Note: `.git/hooks/` is not version-controlled, so each fresh clone still needs this step — that's why Phase 6 keeps it in the README onboarding for teammates.

### 5-2. Bash guard (blocks gate bypass)

Read from `references/hook-guard.md` → `## bash-guard.py`. Write to `.claude/guards/bash-guard.py`.

> nuxt auto-format needs no script — it's a `PostToolUse` hook in the nuxt settings.json template that runs `pnpm lint --fix` (the Nuxt ESLint module) after every Write/Edit.

### 5-3. .claude/settings.json

Read the template from `references/profile-{PROFILE}.md` → `## settings.json Template`.

If `.claude/settings.json` already exists:
- **Merge**: add the `hooks` block (for nuxt this includes **both** the `PreToolUse` bash guard and the `PostToolUse` ESLint formatter) and any missing `permissions.allow` entries. Never remove existing entries.
- If the file already has a `hooks` block, merge per-event: append our hook entries without dropping the user's.
- Show the additions before writing.

If it doesn't exist: write fresh.

### 5-3b. .vscode/settings.json (nuxt only)

Editor format-on-save via ESLint. Read `references/profile-nuxt.md` → `## .vscode/settings.json Template`.

- If `.vscode/settings.json` exists: **merge** the keys in (never overwrite; show additions first).
- If not: write fresh.

Other profiles: skip.

### 5-4. Optional: code-reviewer agent

Ask:
```
Add a read-only code-reviewer agent? (yes/no)
```

If yes: read from `references/files-shared.md` → `## code-reviewer agent`. Write to `.claude/agents/code-reviewer.md`.

---

## Phase 6: Update README / Cập nhật README

Check for `README.md`. If found, check whether it already contains `## AI Onboarding`.

If not present, append the following block (replace `{LINT}`, `{TYPECHECK}`, `{TEST}` with profile commands):

```markdown
## AI Onboarding

1. Clone the repo and install dependencies.
2. Install git hook:
   ```sh
   ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit && chmod +x scripts/pre-commit.sh
   ```
3. Verify gates pass: `{LINT} && {TYPECHECK} && {TEST}`
4. Read `CLAUDE.md` → `AI_TASK_GUIDE.md`.
5. Do one scoped task end-to-end through all gates to confirm the setup works.
```

If no `README.md` exists: skip this phase (do not create one).

---

## Phase 7: Summary / Tóm tắt

Print a short summary of what was created and what's next:

```
BigIn harness setup complete for profile: {PROFILE}

Created:
  CLAUDE.md
  AI_TASK_GUIDE.md
  AI_REVIEW_CHECKLIST.md
  .claude/rules/conventions.md
  .claude/rules/security.md
  .claude/rules/architecture.md
  .claude/guards/bash-guard.py
  .claude/settings.json [created/merged]
  scripts/pre-commit.sh
  [.claude/agents/code-reviewer.md] (if opted in)

Enabled:
  git repo [initialized/already present]
  pre-commit hook [installed/left untouched]

Next steps:
  1. {LINT} && {TYPECHECK} && {TEST}
  2. Read CLAUDE.md + AI_TASK_GUIDE.md
  3. One scoped task through all gates — confirm the harness works.
```

---

## Idempotency Rules / Tính bền vững

- Check existence before writing every file.
- `INSTALL_MODE=yes` → overwrite. `INSTALL_MODE=new` → skip existing.
- `.claude/settings.json` — always merge (never full overwrite if file exists).
- `README.md` — append only; never overwrite; check for `## AI Onboarding` first.
- `git init` — only if not already a repo (never re-init).
- pre-commit hook — install only if absent or already ours; confirm before replacing a foreign hook.
- Never delete files not part of the harness.

---

## Output Checklist

- [ ] `CLAUDE.md` — profile-specific, ≤30 lines
- [ ] `.claude/rules/conventions.md` — profile-specific patterns
- [ ] `.claude/rules/security.md` — shared security rules
- [ ] `.claude/rules/architecture.md` — shared base + profile addendum
- [ ] `AI_TASK_GUIDE.md` — spec gate + task workflow
- [ ] `AI_REVIEW_CHECKLIST.md` — profile commands filled in
- [ ] `scripts/pre-commit.sh` — lint + typecheck + test for profile, executable
- [ ] `.claude/guards/bash-guard.py` — blocks `--no-verify` and force-push to main
- [ ] `.claude/settings.json` — guards wired (nuxt also gets a PostToolUse `pnpm lint --fix` auto-format hook) + profile permissions
- [ ] **nuxt only** — `.vscode/settings.json` with ESLint format-on-save (Prettier disabled), merged if it existed
- [ ] git repo initialized (if it wasn't one) and `.git/hooks/pre-commit` installed (or foreign hook left untouched with confirmation)
- [ ] `README.md` — AI Onboarding section appended (if README existed)

---

## References

- `references/profile-nuxt.md` — templates for nuxt profile
- `references/profile-go.md` — templates for go profile
- `references/profile-nodejs.md` — templates for nodejs profile
- `references/files-shared.md` — shared files: security, architecture, AI task guide, review checklist, code-reviewer agent
- `references/hook-guard.md` — bash-guard.py script + pre-commit scripts per profile
