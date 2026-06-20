# Fullstack MVP Stack Spec
# Đặc tả stack Fullstack MVP

## Stack Overview

| Layer | Technology |
|-------|-----------|
| Framework | Nuxt v4 |
| UI Library | Nuxt UI (latest) |
| CSS | Tailwind CSS v4 |
| Font | Google Sans (default, loaded via Google Fonts) |
| Theme Primary | Blue |
| Theme Neutral | Slate |
| State | Pinia + Pinia Colada |
| Utilities | VueUse |
| Runtime | Nitro (cloudflare-pages preset) |
| Deployment | Cloudflare Pages |
| Config | Wrangler (manual, wrangler.toml) |
| Local Dev | nitro-cloudflare-dev |

## Optional Cloudflare Services

| Service | Binding Name | When to Enable |
|---------|-------------|----------------|
| D1 (SQLite) | `DB` | Needs persistent relational data |
| R2 (Object Storage) | `STORAGE` | Needs file/asset uploads |
| KV (Key-Value) | `KV` | Needs cache, sessions, or config store |

Ask the user which optional services to enable during Phase 3.

---

## nuxt.config.ts (canonical baseline)

```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  future: { compatibilityVersion: 4 },

  modules: [
    '@nuxt/ui',
    '@pinia/nuxt',
    '@vueuse/nuxt',
  ],

  ui: {
    theme: {
      colors: ['primary', 'error'],
    },
  },

  ssr: false,

  nitro: {
    preset: 'cloudflare-pages',
  },

  css: ['~/assets/css/main.css'],
})
```

## app.config.ts (canonical baseline)

```typescript
export default defineAppConfig({
  ui: {
    colors: {
      primary: 'blue',
      neutral: 'slate',
    },
  },
})
```

## assets/css/main.css

```css
@import url('https://fonts.googleapis.com/css2?family=Google+Sans:wght@400;500;700&display=swap');

@import "tailwindcss";
@import "@nuxt/ui";

:root {
  font-family: 'Google Sans', sans-serif;
}
```

---

## wrangler.toml (manual config)

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

# [[kv_namespaces]]
# binding = "KV"
# id = "YOUR_KV_NAMESPACE_ID"
```

## package.json scripts

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "deploy": "wrangler pages deploy .output/public",
    "cf-dev": "nitro-cloudflare-dev"
  }
}
```

---

## Nitro API Routes

API routes live in `server/api/`. Example:

```typescript
// server/api/hello.get.ts
export default defineEventHandler(async (event) => {
  const { DB } = event.context.cloudflare.env  // D1 binding
  return { message: 'hello' }
})
```

Bindings are accessed via `event.context.cloudflare.env`.

---

## D1 Conventions (if enabled)

- Schema files in `server/database/schema.sql`
- Migrations in `server/database/migrations/`
- Seed files in `server/database/seed.sql`
- Use `drizzle-orm` with `drizzle-orm/d1` driver (preferred) or raw `env.DB.prepare()`
- All table names: snake_case
- Primary keys: `id INTEGER PRIMARY KEY AUTOINCREMENT`

---

## R2 Conventions (if enabled)

- Upload via `event.context.cloudflare.env.STORAGE.put(key, body)`
- Public URLs via Cloudflare R2 public bucket or custom domain
- Key naming: `{type}/{uuid}.{ext}` (e.g. `avatars/abc123.jpg`)

---

## KV Conventions (if enabled)

- Access via `event.context.cloudflare.env.KV.get(key)` / `.put(key, value)`
- Use for: session tokens, feature flags, rate limit counters, cached responses
- TTL on all temporary keys

---

## Local Dev with nitro-cloudflare-dev

```bash
# Install
npm install -D nitro-cloudflare-dev

# Run (simulates Cloudflare bindings locally)
npm run cf-dev
```

`nitro-cloudflare-dev` reads `wrangler.toml` and injects local D1/R2/KV simulators.  
Local D1 data lives in `.wrangler/state/`.

---

## Deploy

```bash
# Build
npm run build

# Deploy to Cloudflare Pages
npm run deploy

# First-time setup (create project in Cloudflare):
wrangler pages project create your-app-name
```

---

## Folder Structure

```
project/
├── app/
│   ├── components/
│   ├── composables/
│   ├── layouts/
│   ├── pages/
│   ├── stores/          ← Pinia stores
│   └── assets/
│       └── css/
│           └── main.css
├── server/
│   ├── api/
│   ├── middleware/
│   └── database/
│       ├── schema.sql
│       └── migrations/
├── public/
├── nuxt.config.ts
├── app.config.ts
├── wrangler.toml
└── package.json
```
