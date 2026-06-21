# Skill Manifest
# Danh sách skill theo loại dự án

Maps each project type to the skills installed during Phase 5 via `find-skills`.

---

## Type 1: Fullstack MVP

```
nuxt
nuxt-ui
pinia
pinia-colada
vue
vue-best-practices
vue-testing-best-practices
vitest
vueuse-functions
zod
pnpm
cloudflare-pages
drizzle               (optional — only when D1 is enabled)
nuxt-auth-utils       (optional — only when auth is needed)
github-actions
```

**Total: 15 skills** (12 base + drizzle optional + nuxt-auth-utils optional + github-actions)

---

## Type 2: SPA Frontend

```
nuxt
nuxt-ui
pinia
pinia-colada
vue
vue-best-practices
vue-testing-best-practices
vitest
vueuse-functions
zod
pnpm
nuxt-auth-utils       (optional — only when auth is needed)
github-actions
```

**Total: 13 skills** (no cloudflare-pages/drizzle — SPA has no Cloudflare server binding needs; nuxt-auth-utils optional)

---

## Type 3: Backend (Go)

```
(none)
```

Go projects get no library skills. The harness generates a Go-specific `setup`, `api-development`, and `testing` skill inline during Phase 5-2 from the `backend-go.md` spec.

---

## Install Instructions

For each skill in the list above, invoke:

```
Skill('find-skills', '{skill-name} from affaan-m/everything-claude-code')
```

Always prefer `affaan-m/everything-claude-code` as the source registry. If a skill is not found there, fall back to other sources. `find-skills` handles discovery, checks if the skill is already installed, and performs the installation. If it cannot find a skill, note it in the harness setup summary and continue — do not abort.
