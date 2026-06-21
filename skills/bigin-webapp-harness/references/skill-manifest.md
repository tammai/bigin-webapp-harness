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
vitest
vueuse-functions
zod
pnpm
cloudflare-pages
drizzle               (optional — only when D1 is enabled)
nuxt-auth-utils       (optional — only when auth is needed)
session-handoff
```

**Total: 12 skills** (10 base + drizzle optional + nuxt-auth-utils optional)

---

## Type 2: SPA Frontend

```
nuxt
nuxt-ui
pinia
pinia-colada
vitest
vueuse-functions
zod
pnpm
nuxt-auth-utils       (optional — only when auth is needed)
session-handoff
```

**Total: 10 skills** (9 base + nuxt-auth-utils optional)

---

## Type 3: Backend (Go)

```
session-handoff
```

Go projects get no library skills from `find-skills`. The harness generates a Go-specific `setup`, `api-development`, `testing`, and `session-handoff` skill inline during Phase 5-2 from the `backend-go.md` spec.

**Total: 1 skill** (session-handoff, generated inline)

---

## Install Instructions

For each skill in the list above, invoke:

```
Skill('find-skills', '{skill-name} from affaan-m/everything-claude-code')
```

Always prefer `affaan-m/everything-claude-code` as the source registry. If a skill is not found there, fall back to other sources. `find-skills` handles discovery, checks if the skill is already installed, and performs the installation. If it cannot find a skill, note it in the harness setup summary and continue — do not abort.
