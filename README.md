# bigin-webapp-harness

**Web App Harness Factory for Claude Code**  
_Nhà máy tạo Harness Ứng dụng Web cho Claude Code_

A meta-skill that turns a project description into a specialized agent team and the skills they use — focused on three web development stacks.

---

## Project Types / Loại dự án

| Type              | Stack                | Description                                                                                             |
| ----------------- | -------------------- | ------------------------------------------------------------------------------------------------------- |
| **Fullstack MVP** | Nuxt v4 + Cloudflare | Nuxt UI, Tailwind, Pinia, Pinia Colada, VueUse, Nitro (cloudflare-pages), D1/R2/KV (optional), Wrangler |
| **SPA Frontend**  | Nuxt v4, SSR false   | Nuxt UI, Tailwind, Pinia, Pinia Colada, VueUse, client-side only                                        |
| **Backend**       | Go                   | Gin router, standard project layout, testify                                                            |

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

| Pattern            | When                               | Default?              |
| ------------------ | ---------------------------------- | --------------------- |
| **Pipeline**       | Architect → Build → QA sequential  | ✅ All types          |
| **Fan-out/Fan-in** | Parallel builder agents in Phase 2 | ✅ Fullstack MVP, SPA |
| **Sub-agents**     | All agent execution                | ✅ Default mode       |

---

## Plugin Structure

```
bigin-webapp-harness/
├── .claude-plugin/
│   ├── plugin.json                        ← plugin metadata (name, version, author)
│   └── marketplace.json                   ← marketplace registry entry
├── skills/
│   ├── bigin-webapp-harness/              ← Harness factory (main skill, globally registered)
│   │   ├── SKILL.md                       ← 8-phase workflow
│   │   └── references/
│   │       ├── fullstack-mvp.md           ← Nuxt v4 + Cloudflare spec
│   │       ├── spa-frontend.md            ← Nuxt v4 SPA spec
│   │       ├── backend-go.md              ← Go backend spec
│   │       ├── agent-roles.md             ← Role catalog + agent file templates
│   │       ├── orchestrator-template.md   ← Sub-agent orchestrator templates A/B/C
│   │       ├── scaffold.md                ← Project scaffold templates
│   │       └── skill-manifest.md          ← Type → skills mapping
│   └── session-handoff/                   ← Session state persistence skill
│       └── SKILL.md
└── README.md
```

**Library skills (nuxt4-patterns, pinia, etc.) are fetched from `affaan-m/everything-claude-code` and installed into target projects on demand during Phase 5 — they are not part of this plugin.**

## Skills Installed at Harness-Time

| Skill                        | Purpose                                              | Project types            |
| ---------------------------- | ---------------------------------------------------- | ------------------------ |
| `nuxt4-patterns`             | Nuxt v4 core config, routing, data fetching           | Fullstack MVP, SPA       |
| `nuxt-ui`                    | Component library, design system, forms, layouts     | Fullstack MVP, SPA       |
| `pinia`                      | Stores, composables, SSR patterns, testing           | Fullstack MVP, SPA       |
| `pinia-colada`               | Async queries, mutations, cache, patterns            | Fullstack MVP, SPA       |
| `vitest`                     | Unit/component test setup, patterns, coverage        | Fullstack MVP, SPA       |
| `vueuse`                     | VueUse composables reference (mouse, storage, etc.)   | Fullstack MVP, SPA       |
| `zod`                        | Schema validation, type inference, form + API guards | Fullstack MVP, SPA       |
| `pnpm`                       | Package manager conventions, lockfile, CI setup      | All                      |
| `wrangler`                   | Cloudflare CLI: Wrangler config, D1/R2/KV, deploy     | Fullstack MVP            |
| `drizzle`                    | Drizzle ORM schema, migrations, queries for D1       | Fullstack MVP (D1 opt)   |
| `nuxt-auth-utils`            | Sessions, OAuth, password hashing, WebAuthn          | Fullstack MVP, SPA (opt) |

---

## License

MIT
