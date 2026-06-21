---
name: bigin-webapp-harness
description: "Web app harness factory for Claude Code. Generates a specialized agent team and skills for your web project. MUST use this skill when user says: 'build a harness', 'set up harness', 'create agent team', 'tạo harness', 'cấu hình harness', 'xây dựng harness', or when starting a new web project that needs structured agent automation. Supports three project types: Nuxt v4 Fullstack MVP (Cloudflare), Nuxt v4 SPA Frontend, Go Backend."
---

# bigin-webapp-harness — Web App Harness Factory

Meta-skill that generates agent definitions (`.claude/agents/`) and skills (`.claude/skills/`) for web application projects.

*Meta-skill tạo agent definitions (`.claude/agents/`) và skills (`.claude/skills/`) cho các dự án web application.*

---

## Core Principles / Nguyên tắc cốt lõi

1. Generate `.claude/agents/` and `.claude/skills/` files for every harness
2. **Sub-agents is the default execution mode** — agents run in parallel, return results to orchestrator
3. User selects project type and agent roles before generation begins
4. `architect` uses `model: "opus"`; all other agents use `model: "sonnet"`
5. Stack conventions are non-negotiable defaults — do not deviate unless user explicitly overrides

---

## Session Handoff / Chuyển tiếp phiên

**When user triggers save session:**

At any point during harness execution, if the user says "save session", "/save-session", "nearing limit", or similar:

1. **Capture current harness state:**
   - Current phase number and name
   - Project type selected
   - Agent roles selected
   - Optional services enabled (D1, R2, KV, auth)
   - Completed phases (checkbox list)
   - Current phase progress

2. **Write to SESSION.md:**
   ```markdown
   ## Current Harness State
   Phase: <current-phase-number> (<phase-name>)
   Project Type: <Fullstack MVP / SPA Frontend / Go Backend>
   Selected Agents: <agent-list>
   Optional Services: <D1/KV/R2/auth status>

   ## Progress
   - [x] Phase 0: Repo detection
   - [x] Phase 1: Type selection
   - [x] Phase 2: Agent role selection
   - [x] Phase 3: Stack verification
   - [ ] Phase 4: Agent generation (NEXT)
   - [ ] Phase 5: Skill install
   - [ ] Phase 6: Orchestrator generation
   - [ ] Phase 7: Validation

   ## Context Notes
   <any decisions made, user preferences, issues encountered>
   ```

3. **Confirm save:**
   ```
   ✅ Harness state saved to SESSION.md
   Resume: Next session will continue from Phase <next-phase>
   ```

**On session resume (when SESSION.md exists with harness state):**

1. **Read SESSION.md** and display harness state
2. **Restore context:**
   - Display project type, selected agents, optional services
   - Display progress checklist
   - Display context notes
3. **Continue from next uncompleted phase**
4. **Ask user:** "Resume harness from Phase <X> or make changes?"

---

## Phase 0: Repo Detection & Scaffold / Phát hiện và khởi tạo repo

Before asking anything, check if the repo is empty.

**Empty = none of these exist:** `package.json`, `nuxt.config.ts`, `go.mod`

```
If empty repo detected:
  → Tell the user: "This looks like an empty repo. I'll scaffold the project
    structure for you after you choose a type. Packages will be installed automatically."
  → Set scaffold_needed = true
  → Continue to Phase 1 (type selection happens first, scaffold happens after)

If files already exist:
  → Set scaffold_needed = false
  → Skip scaffold entirely, proceed normally
```

Scaffold is deferred to after Phase 3 (stack confirmation), so we know which type to scaffold. See `references/scaffold.md` for the exact files and directory structure per type.

---

## Phase 1: Project Type Selection / Chọn loại dự án

Ask the user to choose a project type. Present exactly these three options:

```
Which project type are you building?

1. Fullstack MVP — Nuxt v4 + Cloudflare (D1, R2, KV, Pages)
2. SPA Frontend — Nuxt v4, SSR disabled
3. Backend — Go

Type 1, 2, or 3.
```

*Vietnamese: "Bạn đang xây dựng loại dự án nào? 1. Fullstack MVP, 2. SPA Frontend, 3. Backend Go"*

Once type is selected, load the corresponding spec from references/:
- Type 1 → `references/fullstack-mvp.md`
- Type 2 → `references/spa-frontend.md`
- Type 3 → `references/backend-go.md`

---

## Phase 2: Agent Role Selection / Chọn vai trò agent

Present the available agent roles for the chosen project type (see `references/agent-roles.md` for the full role catalog per type).

Show the role list and ask:

```
Available agents for [project type]:
[list roles with one-line descriptions]

Which agents do you want? You can select all or a subset.
Recommended: [recommend the minimal set for an MVP]
```

Rules:
- Always include at least one **builder** agent and one **QA** agent
- User can add or remove roles freely
- If user says "all" or "recommended", use the full set for that type
- Confirm the final selection before proceeding

---

## Phase 3: Stack Verification / Xác nhận stack

Read the spec file for the chosen type. Confirm with the user:

```
Stack for your [type] project:
[summarize key tech from spec]

Any optional services to enable?
[list optional items, e.g. D1, R2, KV for fullstack MVP]

Proceed? (yes / or tell me what to change)
```

Capture all choices. These become the ground truth for all agent and skill generation.

---

## Phase 3.5: Scaffold (if empty repo) / Khởi tạo dự án

If `scaffold_needed = true`, execute scaffold now using `references/scaffold.md` for the confirmed project type.

**Rules:**
- Follow `references/scaffold.md` exactly for the confirmed project type
- **Nuxt types (1 & 2):** run `pnpm install` automatically at the end — project must be ready to develop immediately after scaffold
- **Go type:** do NOT run any install command — write files only, user runs `go mod tidy` manually
- Nuxi-generated files that scaffold.md Step 3 explicitly says to replace **must be overwritten** — the raw nuxi template config is incomplete (missing nitro preset, eslint stylistic, etc.)
- After scaffold, print the "Announce" block from scaffold.md for the chosen type

If `scaffold_needed = false`, skip this phase entirely.

---

## Phase 4: Agent Definition Generation / Tạo agent definitions

Create one file per selected role at `project/.claude/agents/{role-name}.md`.

**Rules:**
- Every agent MUST have a definition file — no inline roles in Agent tool prompts
- Built-in types (`general-purpose`, `Explore`, `Plan`) still require definition files
- QA agent must use `general-purpose` type (not `Explore` — read-only, cannot run scripts)
- `architect` uses `model: "opus"`; all others use `model: "sonnet"`

**Required sections per agent file:**
```markdown
---
name: {role-name}
description: {one-line role description — what triggers this agent}
model: opus   # architect only — all others use sonnet
---

# {Role Name}

## Role / Vai trò
[What this agent is responsible for]

## Stack Knowledge / Kiến thức stack
[Specific technologies, frameworks, conventions this agent must know — pulled from the type spec]

## Task Principles / Nguyên tắc làm việc
[How this agent approaches its work — conventions, quality bar, constraints]

## Input / Output Protocol
[What it receives, what it produces, file naming conventions]

## Error Handling
[What to do when blocked, tools fail, or output is ambiguous]
```

For QA agents, add:
```markdown
## Validation Approach
[Boundary cross-comparison: read both sides simultaneously — e.g., API route + frontend composable]
[Run incrementally after each module, not once at the end]
```

---

## Phase 5: Skill Install / Cài đặt skills

Skills are **not bundled** in this plugin — they are discovered and installed at harness-time via the `find-skills` skill. This keeps the plugin lean and skills current from their authoritative sources.

### 5-1. Install library skills via find-skills

Read `references/skill-manifest.md` for the skill list per project type. For each skill, invoke:

```
Skill('find-skills', '{skill-name} from affaan-m/everything-claude-code')
```

Always prefer `affaan-m/everything-claude-code` as the source registry. If a skill is not found there, fall back to other sources. Call `find-skills` once per skill, sequentially. If `find-skills` reports the skill is already installed, skip it. If it cannot find a skill, note it in the setup summary and continue — do not abort.

**Skills per type (summary):**

| Type | Skills to install |
|------|-----------------|
| Fullstack MVP | nuxt, nuxt-ui, pinia, pinia-colada, vitest, vueuse-functions, zod, pnpm, cloudflare-pages, session-handoff |
| SPA Frontend | nuxt, nuxt-ui, pinia, pinia-colada, vitest, vueuse-functions, zod, pnpm, session-handoff |
| Backend (Go) | session-handoff *(all others generated inline; see 5-2)* |

### 5-2. Generate project-specific orchestrator skills

After installing the library skills, generate these thin project-specific skills at `project/.claude/skills/`:

**All types:**
- `webapp-harness/` — orchestrator (from `references/orchestrator-template.md`)
- `setup/` — project-specific setup guide (scaffold commands, local dev, first-run checklist)

**Fullstack MVP extras:**
- `api-development/` — Nitro + Cloudflare binding conventions (thin, references copied nuxt skill)
- `deployment/` — wrangler deploy commands, CI/CD steps
- `database/` — D1 conventions (only if D1 enabled)

**Go Backend (generate all inline, no library):**
- `setup/`, `api-development/`, `testing/`

### 5-3. Skill authoring rules (for generated skills only)

| Principle | Rule |
|-----------|------|
| **Aggressive descriptions** | Description is the only trigger — make it pushy and specific |
| **Stay lean** | ≤500 lines per SKILL.md; move details to `references/` |
| **Commands over prose** | Show exact CLI commands for setup/deploy skills |
| **Bilingual headers** | Use `## Section / Phần` format |

### 5-4. Progressive Disclosure

| Tier | When Loaded | Size |
|------|------------|------|
| Metadata (name + description) | Always | ~100 words |
| SKILL.md body | On trigger | ≤500 lines |
| references/ | On demand only | Unlimited |

---

## Phase 6: Orchestrator Skill / Skill điều phối

Create one orchestrator skill at `project/.claude/skills/webapp-harness/SKILL.md`.

The orchestrator wires all agents and skills into a single workflow. See `references/orchestrator-template.md` for templates.

**Sub-agent orchestration pattern (default):**

```
[Orchestrator]
    ├── Read project brief + stack config
    ├── Agent(setup-agent, run_in_background=false)      ← sequential: must finish first
    ├── Agent(builder-agent, run_in_background=true)     ← parallel start
    ├── Agent(api-agent, run_in_background=true)         ─┤
    ├── Agent(db-agent, run_in_background=true)          ─┘ wait for all
    ├── Agent(qa-agent, run_in_background=false)         ← sequential: runs after builders
    └── Synthesize results → final summary
```

**Data transfer between sub-agents (file-based):**
- Create `_workspace/` in project root for intermediate outputs
- File naming: `{phase}_{agent}_{artifact}.{ext}` (e.g. `02_builder_components.md`)
- Final outputs go to their target paths; preserve `_workspace/` for audit

**Error handling:**
- 1 retry on failure → if still failing, proceed without that agent's output (note in summary)
- Never delete conflicting outputs — keep both with source labels

---

## Phase 7: Validation / Kiểm tra

### 7-1. Structure Check
- [ ] All selected agent files exist at `.claude/agents/{name}.md`
- [ ] All skill files exist at `.claude/skills/{name}/SKILL.md`
- [ ] Orchestrator skill exists
- [ ] No files created in `.claude/commands/`
- [ ] `architect` has `model: opus`; all other agents have `model: sonnet`

### 7-2. Skill Trigger Test
For each skill, generate:
- 5 **should-trigger** queries (explicit + implicit phrasings, EN + VI)
- 5 **should-NOT-trigger** queries (near-miss: similar topic, wrong skill)

Good near-miss example: "Update the color theme" (ui-development vs state-management)
Bad near-miss: "Write a poem" — obviously irrelevant, no test value.

### 7-3. Dry-Run
- Orchestrator phase order is logical
- No dead links in data transfer paths (all `_workspace/` filenames referenced by downstream agents exist)
- QA agent runs after all builder agents

---

## Output Checklist / Danh sách kiểm tra

- [ ] `.claude/agents/` — all selected agent definition files
- [ ] `.claude/skills/` — library skills installed via find-skills (per skill-manifest.md) + generated orchestrator/setup skills
- [ ] `.claude/skills/webapp-harness/SKILL.md` — orchestrator
- [ ] `architect`: `model: opus` — all others: `model: sonnet`
- [ ] No `.claude/commands/` entries
- [ ] Skill descriptions are aggressive (pushy)
- [ ] All SKILL.md bodies ≤500 lines
- [ ] Trigger tests written (5 should + 5 should-not per skill)
- [ ] Stack conventions match the chosen type spec exactly

---

## References

- Stack specs: `references/fullstack-mvp.md`, `references/spa-frontend.md`, `references/backend-go.md`
- Agent role catalog: `references/agent-roles.md`
- Orchestrator templates: `references/orchestrator-template.md`
- Scaffold templates: `references/scaffold.md`
- Skill install manifest: `references/skill-manifest.md`
