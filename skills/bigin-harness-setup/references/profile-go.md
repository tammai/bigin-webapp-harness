# Go Profile Templates

Stack: Go REST API backend (Gin router)

---

## Commands

```
lint:       staticcheck ./...
typecheck:  go build ./...
test:       go test ./...
dev:        go run ./cmd/...
build:      go build -o bin/server ./cmd/...
```

---

## CLAUDE.md Template

```markdown
# CLAUDE.md

Stack: Go REST API · Gin
Go: ≥1.22

## Commands
| Purpose   | Command                          |
|-----------|----------------------------------|
| dev       | `go run ./cmd/...`               |
| test      | `go test ./...`                  |
| vet       | `go vet ./...`                   |
| lint      | `staticcheck ./...`              |
| build     | `go build -o bin/server ./cmd/...` |

## Rules
See `.claude/rules/` — conventions, security, architecture.

## Hard Rules (non-negotiable)
- No `//nolint` suppressions without a comment explaining the exception.
- No unauthenticated endpoints.
- Validate all inputs at handler boundaries — never in service or repo layer.
- `openapi.yaml` is written first; handlers implement it.
- Backend leads with additive changes. Breaking API change = version bump (`/v2/`).

## Spec Gate
Non-trivial features require an approved spec before implementation.
See `AI_TASK_GUIDE.md` for the workflow.
```

---

## conventions.md Template

```markdown
# Conventions

## Naming
- Packages: lowercase, single word (`handler`, `service`, `repository`)
- Exported types: PascalCase. Unexported: camelCase.
- HTTP handlers: `Handle{Resource}{Action}` (e.g. `HandleUserCreate`)
- Errors: package-level vars — `var ErrNotFound = errors.New("not found")`
- Files: snake_case (`user_handler.go`)

## HTTP Handler Pattern (Gin)

```go
func HandleUserCreate(svc *service.UserService) gin.HandlerFunc {
    return func(c *gin.Context) {
        // 1. Bind + validate at this boundary only
        var req CreateUserRequest
        if err := c.ShouldBindJSON(&req); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        // 2. Call service
        user, err := svc.Create(c.Request.Context(), req)
        if err != nil {
            respondError(c, err)
            return
        }
        // 3. Respond
        c.JSON(http.StatusCreated, user)
    }
}
```

Validation happens at the handler boundary. Services and repos receive clean, validated data.

## OpenAPI First

Write `openapi.yaml` before implementing any new route.
Any new route must be in `openapi.yaml` before its handler is written.

## Error Handling
- Return errors up the call stack — never panic in handlers.
- Use a shared `respondError(c *gin.Context, err error)` helper that maps domain errors to HTTP codes.
- Log errors at the handler boundary, not in services.

## Project Layout
```
cmd/server/main.go    ← entry point
internal/
  handler/            ← Gin handlers + route registration
  middleware/         ← Gin middleware
  service/            ← business logic
  repository/         ← data access
  model/              ← domain models / DTOs
pkg/                  ← reusable public packages
```
```

---

## architecture addendum

```markdown
## [Go] Package Structure
- All domain logic lives in `internal/`. Nothing in `pkg/` depends on `internal/`.
- Handler files: routing + input binding/validation only. No business logic.
- Business logic in `service/`. Data access in `repository/`. Never reverse these layers.
- Shared cross-cutting concerns (auth middleware, response helpers) in `internal/middleware/` or `pkg/`.
```

---

## settings.json Template

```json
{
  "permissions": {
    "allow": [
      "Bash(go build:*)",
      "Bash(go run:*)",
      "Bash(go test:*)",
      "Bash(go vet:*)",
      "Bash(go mod:*)",
      "Bash(go generate:*)",
      "Bash(staticcheck:*)",
      "Bash(golangci-lint:*)",
      "Bash(make:*)",
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
    ]
  }
}
```
