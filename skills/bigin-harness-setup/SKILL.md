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

Write to `scripts/pre-commit.sh`. Make it executable: `chmod +x scripts/pre-commit.sh`.

Tell the user (do not run automatically):
```
Install hook: ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit
```

### 5-2. Bash guard (blocks gate bypass)

Read from `references/hook-guard.md` → `## bash-guard.py`.

Write to `.claude/guards/bash-guard.py`.

### 5-3. .claude/settings.json

Read the template from `references/profile-{PROFILE}.md` → `## settings.json Template`.

If `.claude/settings.json` already exists:
- **Merge**: add `hooks` block and any missing `permissions.allow` entries. Never remove existing entries.
- Show the additions before writing.

If it doesn't exist: write fresh.

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

Next steps:
  1. ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit
  2. {LINT} && {TYPECHECK} && {TEST}
  3. Read CLAUDE.md + AI_TASK_GUIDE.md
  4. One scoped task through all gates — confirm the harness works.
```

---

## Idempotency Rules / Tính bền vững

- Check existence before writing every file.
- `INSTALL_MODE=yes` → overwrite. `INSTALL_MODE=new` → skip existing.
- `.claude/settings.json` — always merge (never full overwrite if file exists).
- `README.md` — append only; never overwrite; check for `## AI Onboarding` first.
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
- [ ] `.claude/settings.json` — hook wired + profile permissions
- [ ] `README.md` — AI Onboarding section appended (if README existed)

---

## References

- `references/profile-nuxt.md` — templates for nuxt profile
- `references/profile-go.md` — templates for go profile
- `references/profile-nodejs.md` — templates for nodejs profile
- `references/files-shared.md` — shared files: security, architecture, AI task guide, review checklist, code-reviewer agent
- `references/hook-guard.md` — bash-guard.py script + pre-commit scripts per profile
