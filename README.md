# bigin-webapp-harness

**Web App Harness Factory for Claude Code**  
*Nhà máy tạo Harness Ứng dụng Web cho Claude Code*

A meta-skill that turns a project description into a specialized agent team and the skills they use — focused on three web development stacks.

---

## Project Types / Loại dự án

| Type | Stack | Description |
|------|-------|-------------|
| **Fullstack MVP** | Nuxt v4 + Cloudflare | Nuxt UI, Tailwind, Pinia, VueUse, Nitro (cloudflare-pages), D1/R2/KV (optional), Wrangler |
| **SPA Frontend** | Nuxt v4, SSR false | Nuxt UI, Tailwind, Pinia, Pinia Colada, VueUse, client-side only |
| **Backend** | Go | chi router, standard project layout, testify |

**All Nuxt types share:** Google Sans font, primary blue / neutral slate theme, `ssr: false`.

---

## Usage / Cách dùng

Trigger in Claude Code with:

```
Build a harness for this project
Set up a harness
Tạo harness cho dự án này
Cấu hình harness
```

The skill will guide you through:
1. Choosing your project type (Fullstack MVP / SPA / Backend Go)
2. Selecting which agent roles you want
3. Confirming your stack and optional services
4. Generating all agent definitions and skills

---

## What Gets Generated / Những gì được tạo ra

```
your-project/
├── .claude/
│   ├── agents/
│   │   ├── architect.md
│   │   ├── frontend-dev.md
│   │   ├── api-dev.md          ← fullstack only
│   │   ├── database-dev.md     ← only if D1 enabled
│   │   ├── deployment.md       ← fullstack only
│   │   ├── state-dev.md        ← SPA only
│   │   ├── backend-dev.md      ← Go only
│   │   └── qa.md
│   └── skills/
│       ├── webapp-harness/     ← orchestrator
│       │   └── SKILL.md
│       ├── setup/
│       ├── ui-development/
│       ├── api-development/    ← fullstack only
│       ├── database/           ← if D1 enabled
│       ├── deployment/         ← fullstack only
│       └── state-management/   ← SPA only
└── _workspace/                 ← intermediate outputs (audit trail)
```

---

## Key Differences from revfactory/harness

| Feature | revfactory/harness | bigin-webapp-harness |
|---------|-------------------|---------------------|
| Default mode | Agent Teams | **Sub-agents** |
| Domain | General purpose | **Web apps only** |
| Stack | Any | **Nuxt v4, Go** |
| User input | Domain description | **Project type + role selection** |
| Language | Korean | **English + Vietnamese** |

---

## Installation / Cài đặt

### Via Marketplace

```
/plugin marketplace add tammai/bigin-webapp-harness
/plugin install bigin-webapp-harness@bigin
```

### Direct (Global Skill)

```bash
cp -r skills/bigin-webapp-harness ~/.claude/skills/bigin-webapp-harness
```

**Requirement:** Claude Code with Agent support enabled.

---

## Architecture Patterns Used

| Pattern | When | Default? |
|---------|------|---------|
| **Pipeline** | Architect → Build → QA sequential | ✅ All types |
| **Fan-out/Fan-in** | Parallel builder agents in Phase 2 | ✅ Fullstack MVP, SPA |
| **Sub-agents** | All agent execution | ✅ Default mode |

---

## Plugin Structure

```
bigin-webapp-harness/
├── .claude-plugin/
│   └── plugin.json                        ← 1 skill registered globally
├── skills/
│   ├── bigin-webapp-harness/              ← Harness factory (main skill, globally registered)
│   │   ├── SKILL.md                       ← 8-phase workflow
│   │   └── references/
│   │       ├── fullstack-mvp.md           ← Nuxt v4 + Cloudflare spec
│   │       ├── spa-frontend.md            ← Nuxt v4 SPA spec
│   │       ├── backend-go.md              ← Go backend spec
│   │       ├── agent-roles.md             ← Role catalog + agent file templates
│   │       ├── orchestrator-template.md   ← Sub-agent orchestrator templates A/B/C
│   │       ├── scaffold.md                ← Project file templates
│   │       └── skill-manifest.md          ← Type → skills mapping
│   ├── nuxt/                              ← Nuxt v4 deep reference
│   ├── nuxt-ui/                           ← Nuxt UI component library
│   ├── pinia/                             ← Pinia state management
│   ├── pinia-colada/                      ← Pinia Colada async data
│   ├── vue/                               ← Vue 3 core patterns
│   ├── vue-best-practices/                ← Vue best practices
│   ├── vue-testing-best-practices/        ← Vue testing
│   ├── vueuse-functions/                  ← VueUse function reference
│   ├── pnpm/                              ← pnpm package manager conventions
│   ├── cloudflare-pages/                  ← Cloudflare Pages deployment
│   ├── drizzle/                           ← Drizzle ORM + D1 (SQLite)
│   └── github-actions/                    ← GitHub Actions CI/CD workflows
└── README.md
```

**Library skills are installed into projects on demand during Phase 5 — not loaded globally.**

## Bundled Skills

| Skill | Purpose | Project types |
|-------|---------|---------------|
| `nuxt` | Nuxt v4 core config, routing, data fetching | Fullstack MVP, SPA |
| `nuxt-ui` | Component library, design system, forms, layouts | Fullstack MVP, SPA |
| `pinia` | Stores, composables, SSR patterns, testing | Fullstack MVP, SPA |
| `pinia-colada` | Async queries, mutations, cache, patterns | Fullstack MVP, SPA |
| `vue` | Script setup, new APIs, advanced patterns | Fullstack MVP, SPA |
| `vue-best-practices` | Component design, performance, accessibility | Fullstack MVP, SPA |
| `vue-testing-best-practices` | Unit tests, component tests, E2E | Fullstack MVP, SPA |
| `vueuse-functions` | Per-function reference for entire VueUse library | Fullstack MVP, SPA |
| `pnpm` | Package manager conventions, lockfile, CI setup | All |
| `cloudflare-pages` | Wrangler config, D1/R2/KV bindings, env vars, deploy | Fullstack MVP |
| `drizzle` | Drizzle ORM schema, migrations, queries for D1 | Fullstack MVP (D1 opt) |
| `github-actions` | CI (typecheck + build) and deploy workflows | Fullstack MVP, SPA |

---

## License

MIT
