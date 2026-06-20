# SPA Frontend Stack Spec
# Đặc tả stack SPA Frontend

## Stack Overview

| Layer | Technology |
|-------|-----------|
| Framework | Nuxt v4, SSR disabled (`ssr: false`) |
| UI Library | Nuxt UI (latest) |
| CSS | Tailwind CSS v4 |
| Font | Google Sans (default, loaded via Google Fonts) |
| Theme Primary | Blue |
| Theme Neutral | Slate |
| State | Pinia + Pinia Colada |
| Utilities | VueUse |
| Rendering | Client-side only (SPA) |

No server-side rendering. No Nitro presets. No Cloudflare services.  
*Không có server-side rendering. Không dùng Nitro preset. Không có Cloudflare services.*

---

## nuxt.config.ts (canonical baseline)

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

## package.json scripts

```json
{
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "preview": "nuxt preview"
  }
}
```

`nuxt build` with `ssr: false` produces a client-only SPA in `.output/public/`. Use `nuxt build` (not `nuxt generate`) in CI/CD scripts — both fullstack and SPA use the same build command. Deploy to any static host (Netlify, Vercel, Cloudflare Pages, GitHub Pages).

---

## Pinia Store Conventions

```typescript
// stores/useCounterStore.ts
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  const increment = () => count.value++
  return { count, increment }
})
```

- Stores live in `app/stores/`
- Use Composition API style (not Options API)
- One store per domain concept

---

## Pinia Colada (Data Fetching)

Use Pinia Colada for async data fetching instead of `useFetch` or `useAsyncData` in SPA mode:

```typescript
// composables/useUsers.ts
import { useQuery } from '@pinia/colada'

export const useUsers = () => useQuery({
  key: ['users'],
  query: () => $fetch('/api/users'),
})
```

---

## VueUse Conventions

- Import from `@vueuse/core`
- Prefer VueUse composables over manual DOM/event code
- Common uses: `useLocalStorage`, `useDark`, `useMediaQuery`, `useDebounce`, `useThrottleFn`

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
├── public/
├── nuxt.config.ts
├── app.config.ts
└── package.json
```

---

## API Integration

Since this is a pure SPA, all data comes from external APIs. Use `$fetch` or Pinia Colada queries:

```typescript
// Direct fetch
const data = await $fetch('https://api.example.com/resource')

// With Pinia Colada (preferred for reactive, cached data)
const { data, status } = useQuery({
  key: ['resource', id],
  query: () => $fetch(`https://api.example.com/resource/${id}`),
})
```

Set `NUXT_PUBLIC_API_URL` in `.env` for the API base URL:

```typescript
// nuxt.config.ts
runtimeConfig: {
  public: {
    apiBase: process.env.NUXT_PUBLIC_API_URL || 'http://localhost:3001',
  },
}
```
