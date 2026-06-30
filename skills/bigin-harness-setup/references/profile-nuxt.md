# Nuxt Profile Templates

Stack: Nuxt 4 SPA, Nuxt ESLint, Pinia + Pinia Colada, VueUse, Nuxt UI, nuxt-auth-utils, Zod, Vitest, Cloudflare Pages

---

## Commands

```
lint:       pnpm lint
format:     pnpm lint --fix
typecheck:  pnpm type-check
test:       pnpm test --run
build:      pnpm build
dev:        pnpm dev
```

Every file created or edited is auto-formatted by the Nuxt ESLint module: the `PostToolUse` hook in `.claude/settings.json` runs `pnpm lint --fix` after every Write/Edit. No custom script — it uses the project's own `lint` script (`@nuxt/eslint` flat config).

---

## CLAUDE.md Template

```markdown
# CLAUDE.md

Stack: Nuxt 4 SPA · Cloudflare Pages
Auth: nuxt-auth-utils (sealed session cookie)
Runtime: Node ≥22 · pnpm only

## Commands
| Purpose   | Command            |
|-----------|--------------------|
| dev       | `pnpm dev`         |
| test      | `pnpm test --run`  |
| lint      | `pnpm lint`        |
| format    | `pnpm lint --fix`  |
| typecheck | `pnpm type-check`  |
| build     | `pnpm build`       |

## Rules
See `.claude/rules/` — conventions, security, architecture.

## Hard Rules (non-negotiable)
- Files are auto-formatted with Nuxt ESLint on every create/edit (PostToolUse hook). Never disable the hook.
- No `--no-verify`. No `eslint-disable` without a justifying comment. No weakening eslint config to pass checks.
- No `@ts-ignore` or `as any` without a justifying comment.
- No unauthenticated endpoints.
- Auth/session via `nuxt-auth-utils` only — never roll your own session or token store.
- `openapi.yaml` is the API contract. Types generated from it — never hardcoded.
- All HTTP requests via `plugins/api.ts` only (Bearer token attached there).

## Spec Gate
Non-trivial features require an approved spec before implementation.
See `AI_TASK_GUIDE.md` for the workflow.
```

---

## conventions.md Template

```markdown
# Conventions

## Naming
- Components: PascalCase (`UserCard.vue`)
- Composables: camelCase with `use` prefix (`useUserList.ts`)
- API routes: kebab-case (`/api/users/[id].ts`)
- Pinia stores: camelCase with `Store` suffix (`useUserStore.ts`)
- Types/interfaces: PascalCase

## API Client — single entry point

All HTTP calls go through `plugins/api.ts`. This is the only place Bearer tokens are attached.

```ts
// plugins/api.ts
export default defineNuxtPlugin(() => {
  const api = $fetch.create({
    baseURL: useRuntimeConfig().public.apiBase,
    onRequest({ options }) {
      const token = useCookie('auth_token').value
      if (token) options.headers.set('Authorization', `Bearer ${token}`)
    },
  })
  return { provide: { api } }
})
```

Usage: `const { $api } = useNuxtApp()`
Never call `$fetch` with auth headers outside this plugin.

## OpenAPI Types

Generate types before consuming any new API surface:
```sh
pnpm openapi-typescript openapi.yaml -o types/api.d.ts
```

Import: `import type { paths, components } from '~/types/api'`
Never define API response shapes inline — always use generated types.

## Auth — nuxt-auth-utils

Session management uses `nuxt-auth-utils`. Never hand-roll session or token storage.

- Read session in components/pages: `const { loggedIn, user, session } = useUserSession()`
- Set session in a server route: `await setUserSession(event, { user })`
- Clear: `await clearUserSession(event)`
- Protect server routes: `const { user } = await requireUserSession(event)`
- Hash/verify passwords: `hashPassword` / `verifyPassword` from the module.
- Session secret comes from `NUXT_SESSION_PASSWORD` (env only — never committed).

Requires server routes (Nitro), which run on Cloudflare Pages even with `ssr: false`. The Bearer token for the separate backend (`plugins/api.ts`) is sourced from the session — not stored separately.

## State
- Global state: Pinia stores (`stores/`)
- Async data fetching: Pinia Colada queries (`useQuery`, `useMutation`)
- Local UI state: composables or `ref` in the component

## Component rules
- No business logic in components. Move it to a composable or store.
- Props typed with `defineProps<{}>()`. Events typed with `defineEmits<{}>()`.

## Formatting — ESLint only (no Prettier)

Formatting is ESLint via `@nuxt/eslint`. **Prettier is disabled** — never add it.

- `eslint.config.mjs`: `import withNuxt from './.nuxt/eslint.config.mjs'` → `export default withNuxt()`.
- Stylistic config in `nuxt.config.ts`:
  ```ts
  eslint: { config: { stylistic: { commaDangle: 'never', braceStyle: '1tbs' } } }
  ```
- Lint command: `eslint .` (the `lint` script). Fix: `eslint . --fix`.
- Commit-time fix via `lint-staged` in `package.json`:
  ```json
  "lint-staged": { "*.{ts,vue,js,mjs}": "eslint --fix" }
  ```
- Editor format-on-save uses the ESLint extension (`.vscode/settings.json`), not Prettier.
- Claude auto-formats every Write/Edit via the `pnpm lint --fix` `PostToolUse` hook in `.claude/settings.json`.
```

---

## architecture addendum

```markdown
## [Nuxt] Layers & Boundaries
- Use Nuxt Layers (`layers/`) for hard domain separation when the app grows beyond 3 domains.
- No business logic in components — composables or Pinia stores only.
- Composables in `composables/`. Shared utilities in `utils/`.
- Pages in `pages/` — routing only, delegate to composables for data/logic.
```

---

## settings.json Template

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm dev:*)",
      "Bash(pnpm build:*)",
      "Bash(pnpm lint:*)",
      "Bash(pnpm test:*)",
      "Bash(pnpm type-check:*)",
      "Bash(pnpm add:*)",
      "Bash(pnpm remove:*)",
      "Bash(pnpm install:*)",
      "Bash(pnpm openapi-typescript:*)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git pull:*)",
      "Bash(git stash:*)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/guards/bash-guard.py"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "pnpm lint --fix"
          }
        ]
      }
    ]
  }
}
```

---

## .vscode/settings.json Template

Editor format-on-save through the ESLint extension (matches `nuxt-fullstack-template`). Merge into an existing `.vscode/settings.json` rather than overwriting.

```json
{
  "prettier.enable": false,
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "dbaeumer.vscode-eslint",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```
