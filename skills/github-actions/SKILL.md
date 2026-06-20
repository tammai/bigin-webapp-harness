---
name: github-actions
description: "GitHub Actions CI/CD workflow templates for Nuxt v4 + Cloudflare Pages projects. Use when setting up CI pipelines, deploy workflows, type-check jobs, preview deployments, or GitHub secrets configuration."
---

# GitHub Actions — CI/CD Conventions

Reference for GitHub Actions workflows in bigin web projects (Nuxt v4 + Cloudflare Pages stack).

---

## File Structure

```
.github/
  workflows/
    ci.yml        ← lint, typecheck, build (runs on every PR)
    deploy.yml    ← deploy to Cloudflare Pages (runs on push to main)
```

---

## CI Workflow (PR checks)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]

jobs:
  check:
    name: Typecheck & Build
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Typecheck
        run: pnpm vue-tsc --noEmit

      - name: Build
        run: pnpm build
        env:
          NUXT_PUBLIC_API_URL: ${{ vars.NUXT_PUBLIC_API_URL }}
```

> **Note:** `vars.*` are GitHub Actions variables (non-sensitive). Use `secrets.*` for tokens and keys.

---

## Deploy Workflow (push to main)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy to Cloudflare Pages
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build
        env:
          NUXT_PUBLIC_API_URL: ${{ vars.NUXT_PUBLIC_API_URL }}

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=${{ vars.CF_PAGES_PROJECT }}
```

---

## D1 Migration Job (optional)

Add this job to `deploy.yml` **before** the deploy step when the project uses D1:

```yaml
  migrate:
    name: Run D1 migrations
    runs-on: ubuntu-latest
    needs: []            # run independently; deploy can depend on this

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Apply D1 migrations
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
        run: |
          for f in server/database/migrations/*.sql; do
            echo "Applying $f"
            npx wrangler d1 execute ${{ vars.D1_DATABASE_NAME }} --file="$f" --remote
          done
```

Then make deploy depend on migrate:

```yaml
  deploy:
    needs: [migrate]
    ...
```

---

## Required GitHub Secrets & Variables

### Secrets (sensitive — set in repo Settings > Secrets and variables > Actions)

| Secret | Description |
|--------|-------------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with Pages + D1 edit permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Your Cloudflare account ID |

### Variables (non-sensitive — same Settings panel, Variables tab)

| Variable | Example value | Description |
|----------|--------------|-------------|
| `CF_PAGES_PROJECT` | `my-app` | Cloudflare Pages project name |
| `NUXT_PUBLIC_API_URL` | `https://api.example.com` | Public API base URL |
| `D1_DATABASE_NAME` | `my-app-db` | D1 database name (if using D1) |

---

## Creating the Cloudflare API Token

1. Go to Cloudflare Dashboard → My Profile → API Tokens
2. Create Token → Use template: **Edit Cloudflare Workers**
3. Add permission: **Cloudflare Pages — Edit**
4. Add permission: **D1 — Edit** (if using D1)
5. Copy token → paste into GitHub secret `CLOUDFLARE_API_TOKEN`

---

## Caching Strategy

The `cache: 'pnpm'` option in `actions/setup-node` automatically caches `~/.pnpm-store` keyed by `pnpm-lock.yaml`. This typically saves 30–60 seconds per run.

No additional cache configuration needed.

---

## Branch Strategy

| Branch | Trigger | Destination |
|--------|---------|-------------|
| `main` | push → deploy.yml | Cloudflare Pages production |
| any PR | PR opened/updated → ci.yml | No deploy; checks only |
| preview | Connect repo to Cloudflare Pages dashboard | Automatic preview URLs per PR (no extra CI config) |

For preview deployments on PRs, enable Cloudflare Pages Git integration via the Dashboard — it handles previews automatically without extra workflow config.

---

## Concurrency (prevent duplicate deploys)

Add to `deploy.yml` to cancel in-flight deploys when a new push arrives:

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true
```

Place this at the top level of the workflow (same level as `on:`).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `pnpm: command not found` | Ensure `pnpm/action-setup` runs before `setup-node` |
| Wrangler auth fails | Verify `CLOUDFLARE_API_TOKEN` secret is set; token must have Pages Edit permission |
| Build passes locally, fails in CI | Check all `NUXT_PUBLIC_*` env vars are set as GitHub variables |
| Cache miss every run | Ensure `pnpm-lock.yaml` is committed and not in `.gitignore` |
| D1 migration fails: database not found | Check `D1_DATABASE_NAME` variable matches name in `wrangler.toml` exactly |
| `wrangler pages deploy` 404 | Project must be created first: `wrangler pages project create <name>` |
