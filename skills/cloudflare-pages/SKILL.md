---
name: cloudflare-pages
description: "Cloudflare Pages deployment conventions for Nuxt v4 fullstack projects. Use when configuring wrangler.toml, D1/R2/KV bindings, environment variables, CI/CD deploy pipelines, or debugging Cloudflare-specific runtime issues."
---

# Cloudflare Pages — Deployment Conventions

Reference for deploying Nuxt v4 Fullstack MVP projects to Cloudflare Pages using Wrangler.

---

## Project Setup

### 1. Install Wrangler

```bash
pnpm add -D wrangler nitro-cloudflare-dev
```

### 2. Configure nuxt.config.ts

```typescript
export default defineNuxtConfig({
  ssr: false,
  nitro: {
    preset: 'cloudflare-pages',
  },
})
```

### 3. wrangler.toml (canonical)

```toml
name = "your-app-name"
compatibility_date = "2025-01-01"
pages_build_output_dir = ".output/public"

# Uncomment as needed:

# [[d1_databases]]
# binding = "DB"
# database_name = "your-db-name"
# database_id = "YOUR_DATABASE_ID"

# [[r2_buckets]]
# binding = "STORAGE"
# bucket_name = "your-bucket-name"

# [[r2_buckets.preview_bucket_name]]
# bucket_name = "your-bucket-name-preview"

# [[kv_namespaces]]
# binding = "KV"
# id = "YOUR_KV_NAMESPACE_ID"
# preview_id = "YOUR_KV_PREVIEW_ID"
```

---

## First-Time Project Creation

```bash
# Create the Pages project in Cloudflare (once only)
wrangler pages project create your-app-name

# Build
pnpm build

# Deploy
wrangler pages deploy .output/public
```

---

## Deploy Script (package.json)

```json
{
  "scripts": {
    "build": "nuxt build",
    "deploy": "wrangler pages deploy .output/public",
    "cf-dev": "nitro-cloudflare-dev"
  }
}
```

---

## Local Development with Bindings

`nitro-cloudflare-dev` simulates Cloudflare bindings (D1, R2, KV) locally:

```bash
pnpm cf-dev
```

Local state is stored in `.wrangler/state/` — add to `.gitignore`:

```
.wrangler/
.output/
```

---

## Cloudflare Services

### D1 (SQLite)

```bash
# Create database
wrangler d1 create your-db-name

# Run migration
wrangler d1 execute your-db-name --file=server/database/migrations/001_init.sql

# Local migration
wrangler d1 execute your-db-name --local --file=server/database/migrations/001_init.sql
```

Access in Nitro:

```typescript
// server/api/example.get.ts
export default defineEventHandler(async (event) => {
  const { DB } = event.context.cloudflare.env
  const result = await DB.prepare('SELECT * FROM users').all()
  return result.results
})
```

### R2 (Object Storage)

```bash
# Create bucket
wrangler r2 bucket create your-bucket-name

# List objects
wrangler r2 object list your-bucket-name
```

Access in Nitro:

```typescript
const { STORAGE } = event.context.cloudflare.env

// Upload
await STORAGE.put(`avatars/${uuid}.jpg`, body, {
  httpMetadata: { contentType: 'image/jpeg' }
})

// Get
const obj = await STORAGE.get(`avatars/${uuid}.jpg`)

// Delete
await STORAGE.delete(`avatars/${uuid}.jpg`)
```

### KV (Key-Value)

```bash
# Create namespace
wrangler kv namespace create YOUR_KV
```

Access in Nitro:

```typescript
const { KV } = event.context.cloudflare.env

// Write (with optional TTL in seconds)
await KV.put('session:abc123', JSON.stringify(data), { expirationTtl: 3600 })

// Read
const raw = await KV.get('session:abc123')
const data = raw ? JSON.parse(raw) : null

// Delete
await KV.delete('session:abc123')
```

---

## Environment Variables

### Production (via Cloudflare Dashboard or Wrangler)

```bash
# Set secret (not in wrangler.toml — use for sensitive values)
wrangler pages secret put MY_SECRET

# Set via Dashboard: Pages > your-project > Settings > Environment variables
```

### wrangler.toml vars (non-sensitive, committed)

```toml
[vars]
PUBLIC_API_URL = "https://api.example.com"
```

### Access in Nitro

```typescript
const { MY_SECRET, PUBLIC_API_URL } = event.context.cloudflare.env
```

### Access in Nuxt (public runtime config)

```typescript
// nuxt.config.ts
runtimeConfig: {
  public: {
    apiBase: process.env.NUXT_PUBLIC_API_URL || 'https://api.example.com',
  },
}
```

Set `NUXT_PUBLIC_API_URL` in Cloudflare Pages environment variables for production.

---

## CI/CD — GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
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

      - run: pnpm install --frozen-lockfile

      - run: pnpm build
        env:
          NUXT_PUBLIC_API_URL: ${{ secrets.NUXT_PUBLIC_API_URL }}

      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy .output/public --project-name=your-app-name
```

Required GitHub secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`.

---

## Preview Deployments

Every PR automatically gets a preview URL when using Cloudflare Pages Git integration (connect repo via Dashboard). No extra CI config needed — Cloudflare handles it.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `event.context.cloudflare` is undefined | Ensure `preset: 'cloudflare-pages'` in nitro config and `ssr: false` |
| Local bindings not working | Run `pnpm cf-dev` not `pnpm dev`; check `.wrangler/state/` exists |
| D1 binding not found | Verify `binding` name in `wrangler.toml` matches code exactly (case-sensitive) |
| Build output wrong path | `pages_build_output_dir = ".output/public"` must match Nitro output |
| KV preview_id missing | Create a separate KV namespace for preview: `wrangler kv namespace create YOUR_KV --preview` |
| Wrangler auth fails in CI | Use `CLOUDFLARE_API_TOKEN` env var, not `wrangler login` |

---

## Binding Name Conventions

| Service | Binding Name | Description |
|---------|-------------|-------------|
| D1 | `DB` | Primary database |
| R2 | `STORAGE` | File/asset storage |
| KV | `KV` | Cache, sessions, config |

These names are project defaults. If multiple D1 databases are needed, use `DB_PRIMARY`, `DB_ANALYTICS`, etc.
