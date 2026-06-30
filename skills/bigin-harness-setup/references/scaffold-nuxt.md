# Nuxt Project Scaffold (in-place)

Used by SKILL.md Phase 0.5 when the nuxt profile is chosen and the repo has **no `nuxt.config.ts`**. Lays the full Nuxt 4 fullstack app from `tammai/nuxt-fullstack-template` into the **current** repo (not a subfolder), then the harness overlay is applied on top.

The template already ships: `nuxt.config.ts` (modules + cloudflare-pages + stylistic eslint), `eslint.config.mjs`, `@nuxt/eslint`, `nuxt-auth-utils`, Pinia/Colada/VueUse/Nuxt UI/Zod, Drizzle + Wrangler, Vitest + Playwright, `simple-git-hooks` + `lint-staged`, `.vscode/settings.json`, and its own `CLAUDE.md`.

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

3. **Customize** (per the `nuxt-fullstack-scaffold` skill, steps 3–5):
   - `package.json` `name`, `wrangler.toml` `name` + `database_name` → the project name (kebab-case; ask if unknown).
   - Theme in `app/app.config.ts` (primary / neutral / size) — ask, else keep defaults.
   - Cloudflare D1 / KV IDs in `wrangler.toml` — ask, else leave placeholders.

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
