# Nuxt Project Scaffold (in-place)

Used by SKILL.md Phase 0.5 when the nuxt profile is chosen and the repo has **no `nuxt.config.ts`**. Lays the full Nuxt 4 fullstack app from `tammai/nuxt-fullstack-template` into the **current** repo (not a subfolder), then the harness overlay is applied on top.

The template already ships: `nuxt.config.ts` (modules + cloudflare-pages + stylistic eslint), `eslint.config.mjs`, `@nuxt/eslint`, `nuxt-auth-utils`, Pinia/Colada/VueUse/Nuxt UI/Zod, Vitest + Playwright, `simple-git-hooks` + `lint-staged`, `.vscode/settings.json`, and its own `CLAUDE.md`.

> The template also includes Drizzle + Wrangler D1/KV bindings from its fullstack roots — these are **removed during scaffold** (step 3) because the Nuxt app is a BFF proxy layer; the backend owns data persistence.

> The `nuxt-fullstack-scaffold` skill is the canonical scaffolder, but it creates a **new subdirectory**. Here we scaffold **in place**. Reuse its customization steps (project name, theme, Cloudflare IDs) — do not duplicate its logic beyond the in-place clone below.

## Steps

1. **Confirm.** This writes many files into the current directory. Confirm with the user before proceeding:
   ```
   No nuxt.config.ts found. Scaffold a full Nuxt 4 fullstack app from
   tammai/nuxt-fullstack-template into this repo? (yes / no)
   ```
   If no → skip scaffold; continue as an onboarding run (governance only).

2. **Clone in place** (preserve the repo's existing `.git`):
   ```sh
   tmp=$(mktemp -d)
   git clone --depth 1 git@github.com:tammai/nuxt-fullstack-template.git "$tmp"
   rm -rf "$tmp/.git"
   cp -R "$tmp/." .          # includes dotfiles; overlays template onto cwd
   rm -rf "$tmp"
   ```
   If the repo is not yet a git repo, Phase 5-1b will `git init` later.

3. **Customize + strip DB layer:**
   - `package.json` `name` and `wrangler.toml` `name` → the project name (kebab-case; ask if unknown).
   - Theme in `app/app.config.ts` (primary / neutral / size) — ask, else keep defaults.
   - Remove Drizzle and D1/KV bindings (BFF doesn't access the DB directly):
     - `pnpm remove drizzle-orm drizzle-kit @libsql/client` (or equivalent Drizzle deps in the template)
     - Remove `server/db/` directory and any `drizzle.config.ts`
     - Remove `[[d1_databases]]` and `[[kv_namespaces]]` blocks from `wrangler.toml`
     - Remove any `db:*` scripts from `package.json`

4. **Install + hooks:**
   ```sh
   pnpm install
   pnpm simple-git-hooks
   ```

5. **Hand off to the governance overlay.** Continue with Phase 1. The template already provides CLAUDE.md, `.vscode/settings.json`, eslint, and the `simple-git-hooks` pre-commit gate — the overlay must reconcile (see SKILL.md Phase 1 + 5):
   - Do **not** overwrite the template's `CLAUDE.md` — append a short pointer to `AI_TASK_GUIDE.md` / `.claude/rules/` instead.
   - Do **not** write `scripts/pre-commit.sh` — the template's `simple-git-hooks` (`pnpm lint-staged` → `eslint --fix`) is the gate.
   - **Merge** `.vscode/settings.json` (don't overwrite).
   - **Add** the BigIn guardrails the template lacks: `.claude/guards/bash-guard.py`, `.claude/settings.json` (permissions + PreToolUse bash-guard + PostToolUse `pnpm lint --fix`), `AI_TASK_GUIDE.md`, `AI_REVIEW_CHECKLIST.md`, `.claude/rules/{security,architecture}.md`.

## Notes
- Playwright browsers aren't installed automatically — `pnpm exec playwright install chromium` when needed.
- The template's login endpoint has a hardcoded credential check — replace with a real DB lookup before shipping.
