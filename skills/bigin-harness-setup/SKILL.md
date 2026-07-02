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
1. nuxt   — Nuxt 4 fullstack (Cloudflare Pages): Nuxt UI, Pinia + Colada, VueUse, nuxt-auth-utils, Vitest, Zod — BFF proxy layer, no direct DB access
2. go     — Go REST API backend
3. nodejs — Node.js TypeScript REST API backend

Type 1, 2, or 3.
```

Store result as `PROFILE`. Load `references/profile-{PROFILE}.md` for all template content.

---

## Phase 0.5: Nuxt Project Scaffold / Khởi tạo dự án Nuxt

**nuxt profile only.** If `PROFILE = nuxt` **and** the repo has no `nuxt.config.ts`:

**Delegate to the `nuxt-scaffold` skill.** Load and follow `skills/nuxt-scaffold/SKILL.md` (installed as part of this same plugin) from its Phase 1 through Phase 8. It scaffolds the Nuxt 4 BFF app **from scratch** — non-interactive `npm create nuxt@latest` + the BFF preset + config + sample code. **No GitHub template clone, no embedded skill copy.** Do not write any project files yourself while it runs.

Set `SCAFFOLDED = true` when `nuxt-scaffold` returns (the governance overlay reconciles with what the scaffold provides — see Phases 1 and 5).

Skip this phase entirely if `nuxt.config.ts` already exists (onboarding an existing repo) or for the `go` / `nodejs` profiles.

---

## Phase 1: Detect Existing Harness / Phát hiện harness hiện có

If `SCAFFOLDED = true`, the `nuxt-scaffold` skill already brought `nuxt.config.ts`, `app/`, `server/`, `eslint.config.mjs`, `.claude/settings.json` (permissions + a `PostToolUse` lint-fix hook), `.vscode/settings.json`, and a `simple-git-hooks` pre-commit gate. Treat those as pre-existing (do not clobber) and skip straight to adding the BigIn guardrails the scaffold lacks: `bash-guard.py` (+ its `PreToolUse` hook), governance rules, and AI files.

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

(The `nuxt-scaffold` skill does **not** write a `CLAUDE.md` — governance is this skill's job — so for `SCAFFOLDED = true` nuxt repos there is no existing `CLAUDE.md` to preserve; write it fresh.)

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

**First check for an existing git-hook manager.** If the repo already gates commits via `simple-git-hooks` or `husky` (key in `package.json`), a `.husky/` dir, or an existing `.git/hooks/pre-commit` → **do NOT create `scripts/pre-commit.sh`**. The existing mechanism is the gate; skip to 5-2. (This is the case for `SCAFFOLDED = true` nuxt repos — the template uses `simple-git-hooks` → `pnpm lint-staged`.)

Otherwise (go / nodejs, or a nuxt repo without a hook manager): read `references/hook-guard.md` → `## pre-commit: {PROFILE}`. Write to `scripts/pre-commit.sh`, then `chmod +x scripts/pre-commit.sh`, and continue to 5-1b.

### 5-1b. Initialize git + install the hook

Only when 5-1 created `scripts/pre-commit.sh`. The hook lives in `.git/hooks/`, so a git repo must exist first.

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

For **nuxt**:
- **If `SCAFFOLDED = true`**: the `nuxt-scaffold` skill already wrote `.claude/settings.json` with `permissions.allow` + a `PostToolUse` `pnpm lint --fix` hook. Merge **only** the `PreToolUse` `bash-guard.py` hook (and any missing `permissions.allow` entries). Do **not** re-add `PostToolUse` — it is already present. Merge per-event; show additions before writing.
- **Otherwise** (onboarding an existing nuxt repo): read the full template from `references/profile-nuxt.md` → `## settings.json Template`. If `.claude/settings.json` exists, merge the `hooks` block + missing `permissions.allow` entries (per-event, never drop the user's); if not, write fresh.

For **go** / **nodejs**: read the template from `references/profile-{PROFILE}.md` → `## settings.json Template`. If the file exists, merge the `hooks` block + missing `permissions.allow` entries (per-event); otherwise write fresh.

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

## Phase 5.5: Knowledge Bundle (optional) / Gói tri thức (tùy chọn)

Ask:
```
Add the Knowledge Bundle convention? (yes/no)
Structured domain knowledge under knowledge/ — concept files with frontmatter, linked from an index, validated by a script. See references/knowledge-bundle.md for the spec.
```

If yes, set `KNOWLEDGE_BUNDLE = true`. Read all templates from `references/knowledge-bundle.md`. Replace `{DATE}` with today's date in ISO 8601 (`YYYY-MM-DD`) in every template before writing.

1. **Rule file** — `## knowledge.md` → write to `.claude/rules/knowledge.md`. Skip if `INSTALL_MODE=new` and it exists.
2. **Starter bundle** — write each (skip existing under `INSTALL_MODE=new`):
   - `## knowledge/meta/knowledge-bundle-spec.md` → `knowledge/meta/knowledge-bundle-spec.md`
   - `## knowledge/index.md` → `knowledge/index.md`
   - `## knowledge/contracts/openapi-contract.md` → `knowledge/contracts/openapi-contract.md`
   - `## knowledge/constraints/agent-rules.md` → `knowledge/constraints/agent-rules.md`
   - `## knowledge/log.md` → `knowledge/log.md`
3. **Validator** — `## tools/knowledge_validate.py` → `tools/knowledge_validate.py`, then `chmod +x tools/knowledge_validate.py`.
4. **Wire into the enforcement gate.** If `scripts/pre-commit.sh` exists (created in Phase 5-1), append a step running `uv run tools/knowledge_validate.py`. If the repo instead uses `simple-git-hooks`/`husky` (Phase 5-1 skipped creating our script), add the same command to that existing hook config rather than creating a second script.
5. **Wire into CLAUDE.md.** Append two lines under the profile's `CLAUDE.md` (written in Phase 2):
   ```
   For system/domain context, read knowledge/index.md before non-trivial changes.
   At sprint end, run the sprint-distill skill to fold merged work into knowledge/ and bigin-skills.
   ```
6. **Wire into AI_REVIEW_CHECKLIST.md.** Append one line to the `## Scope` section (written in Phase 4): `- [ ] Behavior-changing PR → related knowledge/ concept updated?`
7. If Phase 5.6 generates new CI config in this same run, it includes the validator step automatically (see Phase 5.6). If the repo already has **foreign** CI config (not generated by this skill), do **not** edit it automatically — note in the Phase 7 summary that `uv run tools/knowledge_validate.py` should also be added as a CI job/step there.

If no, leave `KNOWLEDGE_BUNDLE` unset and skip everything above — no other phase depends on it.

---

## Phase 5.6: CI Config (optional) / Cấu hình CI (tùy chọn)

Ask:
```
Add CI config? (github/gitlab/both/no)
Generates a workflow that runs {LINT} && {TYPECHECK} && {TEST} on push to main and on merge/pull requests.
```

Store choice as `CI_PROVIDER`. Skip everything below if `no`.

Read templates from `references/ci.md`.

1. **GitHub** (if `CI_PROVIDER` is `github` or `both`): if `.github/workflows/ci.yml` already exists, treat like any other idempotency check — under `INSTALL_MODE=new` skip it silently; under `yes` show it and confirm before overwriting. Otherwise write `## github: {PROFILE}` to `.github/workflows/ci.yml`.
2. **GitLab** (if `CI_PROVIDER` is `gitlab` or `both`): same existence check for `.gitlab-ci.yml`. Otherwise write `## gitlab: {PROFILE}` to `.gitlab-ci.yml`.
3. **If `KNOWLEDGE_BUNDLE = true`** (decided in Phase 5.5, which runs first): before writing each file above, merge in `## knowledge-validate step: github` / `## knowledge-validate step: gitlab` respectively, so the generated CI file validates the knowledge bundle in the same run — no separate manual step needed.

This phase only ever writes CI files it generates itself. It never edits a pre-existing, hand-written CI config — see Phase 5.5 step 7 for that case.

---

## Phase 6: Update README / Cập nhật README

Check for `README.md`. If found, check whether it already contains `## AI Onboarding`.

If not present, append the following block (replace `{LINT}`, `{TYPECHECK}`, `{TEST}` with profile commands):

```markdown
## AI Onboarding

1. Clone the repo and install dependencies.
2. Run `claude` in the repo root and accept the workspace trust dialog — this repo ships a `.claude/settings.json` with pre-approved permissions, which Claude Code only applies after you trust the folder. (If the dialog doesn't appear, or you're on a headless/non-interactive setup, set `hasTrustDialogAccepted: true` for this path in `~/.claude.json`.)
3. Install git hook:
   ```sh
   ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit && chmod +x scripts/pre-commit.sh
   ```
4. Verify gates pass: `{LINT} && {TYPECHECK} && {TEST}`
5. Read `CLAUDE.md` → `AI_TASK_GUIDE.md`.
6. Do one scoped task end-to-end through all gates to confirm the setup works.
```

If no `README.md` exists: skip this phase (do not create one).

---

## Phase 7: Summary / Tóm tắt

Print a short summary of what was created and what's next:

```
BigIn harness setup complete for profile: {PROFILE}

[if SCAFFOLDED] Scaffolded the Nuxt 4 BFF app via the `nuxt-scaffold` skill.

Created:
  AI_TASK_GUIDE.md
  AI_REVIEW_CHECKLIST.md
  .claude/rules/security.md
  .claude/rules/architecture.md
  .claude/guards/bash-guard.py
  .claude/settings.json [created/merged]
  CLAUDE.md [created]
  .claude/rules/conventions.md
  scripts/pre-commit.sh [skipped if a hook manager already exists]
  [.claude/agents/code-reviewer.md] (if opted in)
  [Knowledge Bundle: .claude/rules/knowledge.md, knowledge/*, tools/knowledge_validate.py] (if opted in)
  [.github/workflows/ci.yml] (if CI_PROVIDER is github/both)
  [.gitlab-ci.yml] (if CI_PROVIDER is gitlab/both)

Enabled:
  git repo [initialized/already present]
  pre-commit gate [scripts/pre-commit.sh hook | existing simple-git-hooks/husky]
  [knowledge bundle validation wired into the pre-commit gate] (if opted in)
  [knowledge bundle validation wired into generated CI] (if opted in and CI_PROVIDER != no)
  [sprint-distill available — run it at sprint end to fold merged work into knowledge/ and bigin-skills] (if opted in)

Next steps:
  1. First `claude` run here: accept the workspace trust dialog, or the 19 permissions.allow entries in .claude/settings.json are ignored (Claude Code will print a warning naming this path).
  2. {LINT} && {TYPECHECK} && {TEST}
  3. Read CLAUDE.md + AI_TASK_GUIDE.md
  4. One scoped task through all gates — confirm the harness works.
  [5. Add `uv run tools/knowledge_validate.py` to your existing CI — this skill only wires it into CI it generated itself.] (if opted in and CI_PROVIDER=no but foreign CI config detected)
```

---

## Idempotency Rules / Tính bền vững

- Check existence before writing every file.
- `INSTALL_MODE=yes` → overwrite. `INSTALL_MODE=new` → skip existing.
- `.claude/settings.json` — always merge (never full overwrite if file exists).
- `README.md` — append only; never overwrite; check for `## AI Onboarding` first.
- `git init` — only if not already a repo (never re-init).
- pre-commit hook — skip if a hook manager (simple-git-hooks/husky) or hook already exists; otherwise install only if absent or already ours, confirming before replacing a foreign hook.
- Nuxt scaffold (Phase 0.5) — only if `PROFILE=nuxt` and no `nuxt.config.ts`; delegates to the `nuxt-scaffold` skill (no clone, no embedded copy into the target). When `SCAFFOLDED`, do not overwrite the scaffold's `.vscode/settings.json` or pre-commit — overlay additively.
- Knowledge Bundle (Phase 5.5) — opt-in only (`KNOWLEDGE_BUNDLE`); skip entirely if declined. Never edit unknown CI config automatically — only note it's needed.
- CI Config (Phase 5.6) — opt-in only (`CI_PROVIDER`); skip entirely if `no`. Only ever writes/overwrites CI files this skill generated; never edits pre-existing, hand-written CI config.
- Never delete files not part of the harness.

---

## Output Checklist

- [ ] **nuxt + empty repo** — `nuxt-scaffold` skill executed (Phase 0.5); `nuxt.config.ts` now present
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
- [ ] **if opted in** — Knowledge Bundle: `.claude/rules/knowledge.md`, `knowledge/{meta,contracts,constraints}/*.md`, `knowledge/index.md`, `knowledge/log.md`, `tools/knowledge_validate.py` (executable), wired into the pre-commit gate, `CLAUDE.md` + `AI_REVIEW_CHECKLIST.md` each get one added line
- [ ] **if CI_PROVIDER = github/both** — `.github/workflows/ci.yml` runs lint + typecheck + test (+ knowledge validator if opted in)
- [ ] **if CI_PROVIDER = gitlab/both** — `.gitlab-ci.yml` runs lint + typecheck + test (+ knowledge validator if opted in)

---

## References

- `references/profile-nuxt.md` — templates for nuxt profile
- `references/profile-go.md` — templates for go profile
- `references/profile-nodejs.md` — templates for nodejs profile
- `references/files-shared.md` — shared files: security, architecture, AI task guide, review checklist, code-reviewer agent
- `references/hook-guard.md` — bash-guard.py script + pre-commit scripts per profile
- `references/knowledge-bundle.md` — optional Knowledge Bundle: rule file, spec, starter concept files, validator script
- `references/ci.md` — optional CI config: GitHub Actions + GitLab CI templates per profile, plus the knowledge-validate step
