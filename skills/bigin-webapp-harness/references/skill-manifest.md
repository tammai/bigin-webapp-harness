# Skill Manifest
# Danh sách skill theo loại dự án

Maps each project type to the skills installed during Phase 5 via `find-skills`.

> ⚠️ **Nuxt rule** — install **`nuxt4-patterns`**, NEVER antfu's `nuxt` skill (full rule in CLAUDE.md). If the Phase 5-2 fallback authors a nuxt skill from scratch, pull current Nuxt v4 patterns from **https://nuxt.com/llms.txt** — never copy the antfu skill's content.

---

## Type 1: Fullstack MVP

```
nuxt4-patterns
nuxt-ui
pinia
pinia-colada
vitest
vueuse
zod
pnpm
wrangler
drizzle               (optional — only when D1 is enabled)
nuxt-auth-utils       (optional — only when auth is needed)
session-handoff
```

**Total: 12 skills** (10 base + drizzle optional + nuxt-auth-utils optional)

---

## Type 2: SPA Frontend

```
nuxt4-patterns
nuxt-ui
pinia
pinia-colada
vitest
vueuse
zod
pnpm
nuxt-auth-utils       (optional — only when auth is needed)
session-handoff
```

**Total: 10 skills** (9 base + nuxt-auth-utils optional)

---

## Type 3: Backend (Go)

Go projects install **no library skills via `find-skills`**. All skills — `setup`, `api-development`, `testing`, and `session-handoff` — are generated inline during Phase 5-3 from the `backend-go.md` spec. (Do not route `session-handoff` through `find-skills` for Go projects.)

**Total: 0 skills installed via find-skills** (4 generated inline)

---

## Install Instructions

For each skill in the list above, invoke:

```
Skill('find-skills', '{skill-name} from affaan-m/everything-claude-code')
```

Always prefer `affaan-m/everything-claude-code` as the source registry; fall back to other sources only if a skill is not found there. `find-skills` handles discovery, already-installed checks, and installation. **If a skill cannot be found, do not silently skip it** — resolve all not-found skills with the single batched create-on-not-found fallback in SKILL.md Phase 5-2 (create at `project/.claude/skills/{name}/SKILL.md` per the 5-4 rules, install under a different name, or skip).
