# Orchestrator Templates
# Mẫu Orchestrator

Sub-agent orchestration templates for each project type.  
*Mẫu điều phối sub-agent cho từng loại dự án.*

---

## Template A: Fullstack MVP

```markdown
---
name: webapp-harness
description: "Orchestrates the full Nuxt v4 Fullstack MVP build: runs architect, frontend, API, database, and deployment agents in the correct order, collects results, and produces a final summary. Trigger when user says 'build the app', 'start development', 'run the harness', or 'bắt đầu build'."
---

# Webapp Harness Orchestrator (Fullstack MVP)

## Phase 0: Context Check / Kiểm tra ngữ cảnh

Check if project files and prior run artifacts exist:
- `package.json` missing → scaffold was skipped or not done; remind user to run `pnpm install` before proceeding
- `_workspace/` exists → this is a re-run; skip completed phases, continue from last checkpoint
- Neither → first run on a scaffolded project; proceed to Phase 1

## Phase 1: Architecture (Sequential)

Run architect agent first. All other agents depend on its output.

```
Agent(
  subagent_type: "general-purpose",
  description: "Architect — design system blueprint",
  prompt: "Read the project brief at _workspace/00_brief.md. Produce _workspace/01_architect_blueprint.md containing: pages list, component tree, D1 schema (if enabled), and API contracts.",
  model: "opus"
)
```

Wait for completion. Verify `_workspace/01_architect_blueprint.md` exists before proceeding.

## Phase 2: Parallel Build

Start all builder agents simultaneously after architect output is confirmed.

```
# Run in parallel (run_in_background: true)
Agent(frontend-dev, run_in_background: true,
  prompt: "Read _workspace/01_architect_blueprint.md. Build all Nuxt v4 pages, components, and layouts. Follow fullstack-mvp.md conventions. Log decisions to _workspace/02_frontend-dev_decisions.md."
)

Agent(api-dev, run_in_background: true,
  prompt: "Read _workspace/01_architect_blueprint.md. Implement all Nitro API routes with Cloudflare bindings. Output to server/api/. Log to _workspace/02_api-dev_decisions.md."
)

Agent(database-dev, run_in_background: true,  # only if D1 enabled
  prompt: "Read _workspace/01_architect_blueprint.md. Create D1 schema, migrations, and seed data. Output to server/database/."
)

Agent(deployment, run_in_background: true,
  prompt: "Read the enabled Cloudflare services list. Generate wrangler.toml and DEPLOYMENT.md."
)
```

Wait for all to complete.

## Phase 3: QA (Sequential)

```
Agent(qa, run_in_background: false,
  prompt: "Validate integration across all completed modules. Compare API route shapes against frontend composables. Compare D1 schema against TypeScript types. Compare wrangler.toml bindings against server code. Output _workspace/qa_final_report.md."
)
```

## Phase 4: Summary

Collect all `_workspace/` outputs. Produce a final summary:
- Files created (count by type)
- QA issues found (with file paths)
- Next steps (deploy commands from DEPLOYMENT.md)

---

## Error Handling

| Failure | Action |
|---------|--------|
| Architect fails | Retry once. If still failing, stop — cannot proceed without blueprint. |
| Builder agent fails | Retry once. If still failing, note the gap in summary, continue with remaining agents. |
| QA fails | Note issues in summary. Do not block delivery. |
| Conflicting outputs | Keep both files, add source labels. Never delete. |
```

---

## Template B: SPA Frontend

```markdown
---
name: webapp-harness
description: "Orchestrates the Nuxt v4 SPA Frontend build. Runs architect, frontend-dev, and state-dev agents, then QA. Trigger for: 'build the app', 'start development', 'run the harness', 'bắt đầu build'."
---

# Webapp Harness Orchestrator (SPA Frontend)

## Phase 1: Architecture (Sequential)

```
Agent(architect, run_in_background: false,
  prompt: "Design the SPA component tree, routing structure, and state shape. Output _workspace/01_architect_blueprint.md."
)
```

## Phase 2: Parallel Build

```
Agent(frontend-dev, run_in_background: true,
  prompt: "Build all Nuxt v4 SPA pages, components, layouts per _workspace/01_architect_blueprint.md."
)

Agent(state-dev, run_in_background: true,
  prompt: "Build Pinia stores and Pinia Colada queries per _workspace/01_architect_blueprint.md."
)
```

## Phase 3: QA

```
Agent(qa, run_in_background: false,
  prompt: "Validate store↔component contracts. Check all Pinia Colada query keys match usage. Output _workspace/qa_report.md."
)
```
```

---

## Template C: Backend (Go)

```markdown
---
name: webapp-harness
description: "Orchestrates the Go backend build. Runs architect first, then backend-dev, then QA. Trigger for: 'build the backend', 'start development', 'run the harness', 'bắt đầu build'."
---

# Webapp Harness Orchestrator (Backend Go)

## Phase 1: Architecture (Sequential)

```
Agent(architect, run_in_background: false,
  prompt: "Design the Go API: package structure, handler/service/repository boundaries, data models. Output _workspace/01_architect_blueprint.md with API contracts."
)
```

## Phase 2: Implementation (Sequential — Go layers build on each other)

```
Agent(backend-dev, run_in_background: false,
  prompt: "Implement all layers per _workspace/01_architect_blueprint.md. Order: models → repositories → services → handlers. Output to internal/."
)
```

## Phase 3: QA

```
Agent(qa, run_in_background: false,
  prompt: "Write unit tests for all handlers and services. Verify error paths. Verify interface implementations match. Output test files and _workspace/qa_report.md."
)
```
```

---

## Data Transfer Conventions

```
_workspace/
├── 00_brief.md                    ← project brief (write before running orchestrator)
├── 01_architect_blueprint.md      ← architect output
├── 02_frontend-dev_decisions.md   ← builder decision log
├── 02_api-dev_decisions.md
├── qa_{module}_report.md          ← QA reports per module
└── qa_final_report.md             ← final QA summary
```

File naming: `{phase}_{agent}_{artifact}.{ext}`  
Preserve all `_workspace/` files after completion — they form the audit trail.

---

## Re-run Support

If the orchestrator skill is triggered again on an existing project, check for `_workspace/` first:

```markdown
## Phase 0 (add to all orchestrators)

1. List `_workspace/` files
2. If `01_architect_blueprint.md` exists → this is a re-run or extension
3. Ask user: "I found an existing harness. Do you want to: (a) extend it, (b) rebuild from scratch?"
4. If extend: skip phases whose outputs exist; run only new/changed phases
5. If rebuild: clear `_workspace/`, start from Phase 1
```
