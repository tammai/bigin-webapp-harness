# Project Scaffold Templates
# Mẫu khởi tạo dự án

Write these files verbatim when scaffolding an empty repo. Do NOT run `pnpm install` or any package install — leave that to the user for speed.

*Ghi các file này khi khởi tạo repo trống. KHÔNG chạy `pnpm install` — để user tự cài cho nhanh.*

---

## Detection — Is the repo empty?

Check for any of these files. If **none** exist, the repo is empty and needs scaffolding:
- `package.json`
- `nuxt.config.ts`
- `go.mod`

If any exist, skip scaffold entirely — assume the project is already bootstrapped.

---

## Type 1: Fullstack MVP (Nuxt v4 + Cloudflare)

### Files to create

#### `package.json`
```json
{
  "name": "my-app",
  "private": true,
  "packageManager": "pnpm@9.15.4",
  "type": "module",
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "preview": "nuxt preview",
    "typecheck": "vue-tsc --noEmit",
    "deploy": "wrangler pages deploy .output/public",
    "cf-dev": "nitro-cloudflare-dev"
  },
  "dependencies": {
    "@nuxt/ui": "latest",
    "@pinia/colada": "latest",
    "@pinia/nuxt": "latest",
    "@vueuse/nuxt": "latest",
    "nuxt": "latest"
  },
  "devDependencies": {
    "@nuxt/devtools": "latest",
    "nitro-cloudflare-dev": "latest",
    "typescript": "latest",
    "vue-tsc": "latest",
    "wrangler": "latest"
  }
}
```

#### `nuxt.config.ts`
```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  future: { compatibilityVersion: 4 },

  ssr: false,

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

  nitro: {
    preset: 'cloudflare-pages',
  },

  css: ['~/assets/css/main.css'],
})
```

#### `app.config.ts`
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

#### `assets/css/main.css`
```css
@import url('https://fonts.googleapis.com/css2?family=Google+Sans:wght@400;500;700&display=swap');

@import "tailwindcss";
@import "@nuxt/ui";

:root {
  font-family: 'Google Sans', sans-serif;
}
```

#### `.npmrc`
```ini
shamefully-hoist=true
strict-peer-dependencies=false
```

#### `.gitignore`
```
node_modules/
.output/
.nuxt/
.wrangler/
dist/
*.local
.env
.env.*
!.env.example
```

#### `wrangler.toml`
```toml
name = "my-app"
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
# preview_id = "YOUR_KV_PREVIEW_ID"
```

#### `app/app.vue`
```vue
<template>
  <UApp>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </UApp>
</template>
```

#### `app/layouts/default.vue`
```vue
<template>
  <div class="min-h-screen bg-neutral-50 dark:bg-neutral-950">
    <slot />
  </div>
</template>
```

#### `app/pages/index.vue`
```vue
<template>
  <UContainer class="py-16">
    <h1 class="text-3xl font-bold text-neutral-900 dark:text-neutral-100">
      Hello World
    </h1>
  </UContainer>
</template>
```

### Directories to create (empty, add `.gitkeep`)
```
app/
  components/
  composables/
  layouts/
  pages/
  stores/
  assets/css/
server/
  api/
  middleware/
  database/
    migrations/
public/
```

### After scaffold — tell the user:
```
✅ Project scaffolded. To start developing:

  pnpm install
  pnpm cf-dev    # with Cloudflare bindings
  # or
  pnpm dev       # without bindings

Now generating your agent team...
```

---

## Type 2: SPA Frontend (Nuxt v4, SSR false)

### Files to create

#### `package.json`
```json
{
  "name": "my-app",
  "private": true,
  "packageManager": "pnpm@9.15.4",
  "type": "module",
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "preview": "nuxt preview",
    "typecheck": "vue-tsc --noEmit"
  },
  "dependencies": {
    "@nuxt/ui": "latest",
    "@pinia/colada": "latest",
    "@pinia/nuxt": "latest",
    "@vueuse/nuxt": "latest",
    "nuxt": "latest"
  },
  "devDependencies": {
    "@nuxt/devtools": "latest",
    "typescript": "latest",
    "vue-tsc": "latest"
  }
}
```

#### `nuxt.config.ts`
```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  future: { compatibilityVersion: 4 },

  ssr: false,

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

  css: ['~/assets/css/main.css'],

  runtimeConfig: {
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_URL || 'http://localhost:3001',
    },
  },
})
```

#### `app.config.ts`
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

#### `assets/css/main.css`
```css
@import url('https://fonts.googleapis.com/css2?family=Google+Sans:wght@400;500;700&display=swap');

@import "tailwindcss";
@import "@nuxt/ui";

:root {
  font-family: 'Google Sans', sans-serif;
}
```

#### `.npmrc`
```ini
shamefully-hoist=true
strict-peer-dependencies=false
```

#### `.gitignore`
```
node_modules/
.output/
.nuxt/
dist/
*.local
.env
.env.*
!.env.example
```

#### `.env.example`
```
NUXT_PUBLIC_API_URL=http://localhost:3001
```

#### `app/app.vue`
```vue
<template>
  <UApp>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </UApp>
</template>
```

#### `app/layouts/default.vue`
```vue
<template>
  <div class="min-h-screen bg-neutral-50 dark:bg-neutral-950">
    <slot />
  </div>
</template>
```

#### `app/pages/index.vue`
```vue
<template>
  <UContainer class="py-16">
    <h1 class="text-3xl font-bold text-neutral-900 dark:text-neutral-100">
      Hello World
    </h1>
  </UContainer>
</template>
```

### Directories to create (empty, add `.gitkeep`)
```
app/
  components/
  composables/
  layouts/
  pages/
  stores/
  assets/css/
public/
```

### After scaffold — tell the user:
```
✅ Project scaffolded. To start developing:

  pnpm install
  pnpm dev

Now generating your agent team...
```

---

## Type 3: Backend (Go)

### Files to create

#### `go.mod` (replace `your-module-name` with actual name)
```
module your-module-name

go 1.23
```

#### `cmd/server/main.go`
```go
package main

import (
	"log"
	"net/http"

	"github.com/go-chi/chi/v5"
	"github.com/go-chi/chi/v5/middleware"
)

func main() {
	r := chi.NewRouter()
	r.Use(middleware.Logger)
	r.Use(middleware.Recoverer)

	r.Get("/health", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte(`{"status":"ok"}`))
	})

	log.Println("Server starting on :8080")
	if err := http.ListenAndServe(":8080", r); err != nil {
		log.Fatal(err)
	}
}
```

#### `Makefile`
```makefile
.PHONY: run build test tidy

run:
	go run ./cmd/server

build:
	go build -o bin/server ./cmd/server

test:
	go test ./...

tidy:
	go mod tidy
```

#### `.gitignore`
```
bin/
*.env
.env.*
!.env.example
```

#### `.env.example`
```
PORT=8080
```

### Directories to create (empty, add `.gitkeep`)
```
cmd/server/
internal/
  handler/
  service/
  repository/
  model/
```

### After scaffold — tell the user:
```
✅ Project scaffolded. To start developing:

  go mod tidy
  make run

Now generating your agent team...
```

---

## Scaffold Rules

1. **Never overwrite existing files** — check before writing; skip if file already exists
2. **No `pnpm install` / `go mod tidy`** — write files only, user runs install manually
3. **Use `latest` for all deps in package.json** — agents and skills will handle pinning if needed
4. **Set `name: "my-app"` as placeholder** — user renames in package.json / wrangler.toml / go.mod
5. **Announce what was created** — list every file written before moving to Phase 1
