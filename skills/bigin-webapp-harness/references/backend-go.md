# Backend (Go) Stack Spec
# Đặc tả stack Backend Go

## Stack Overview

| Layer | Technology |
|-------|-----------|
| Language | Go (latest stable) |
| HTTP Router | `net/http` (stdlib) or `chi` (preferred for middleware) |
| Structure | Standard Go project layout |
| Config | Environment variables via `os.Getenv` or `godotenv` |
| Testing | `testing` stdlib + `testify` |

No specific ORM or database is prescribed by default — the architect agent decides based on project needs.  
*Không có ORM hay database cụ thể mặc định — architect agent quyết định dựa trên yêu cầu dự án.*

---

## Project Layout (Standard Go)

```
project/
├── cmd/
│   └── server/
│       └── main.go        ← entry point
├── internal/
│   ├── handler/           ← HTTP handlers
│   ├── middleware/        ← HTTP middleware
│   ├── service/           ← business logic
│   ├── repository/        ← data access layer
│   └── model/             ← domain models / DTOs
├── pkg/                   ← reusable public packages
├── migrations/            ← SQL migrations (if DB used)
├── .env.example
├── go.mod
├── go.sum
└── Makefile
```

---

## main.go (baseline with chi)

```go
package main

import (
    "log"
    "net/http"
    "os"

    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
)

func main() {
    r := chi.NewRouter()

    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.RealIP)

    r.Get("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("ok"))
    })

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    log.Printf("Server listening on :%s", port)
    log.Fatal(http.ListenAndServe(":"+port, r))
}
```

---

## Handler Pattern

```go
// internal/handler/user.go
package handler

import (
    "encoding/json"
    "net/http"
    "github.com/go-chi/chi/v5"
)

type UserHandler struct {
    svc UserService
}

func NewUserHandler(svc UserService) *UserHandler {
    return &UserHandler{svc: svc}
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")
    user, err := h.svc.GetByID(r.Context(), id)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }
    json.NewEncoder(w).Encode(user)
}
```

---

## Service Layer Pattern

```go
// internal/service/user.go
package service

import "context"

type UserService interface {
    GetByID(ctx context.Context, id string) (*User, error)
}

type userService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) UserService {
    return &userService{repo: repo}
}
```

---

## Makefile (standard commands)

```makefile
.PHONY: dev build test lint

dev:
    go run ./cmd/server

build:
    go build -o bin/server ./cmd/server

test:
    go test ./...

lint:
    golangci-lint run
```

---

## Go Conventions

- All errors are returned, never swallowed silently
- Use `context.Context` as first parameter in all service/repository methods
- No global state — inject dependencies via constructors
- Package names: lowercase, single word (e.g. `handler`, `service`, `repository`)
- Exported types: PascalCase
- JSON field tags: snake_case (`json:"user_id"`)
- Always validate input in handlers before passing to service layer

---

## Testing

```go
// internal/handler/user_test.go
func TestGetUser(t *testing.T) {
    mockSvc := &MockUserService{}
    h := NewUserHandler(mockSvc)

    req := httptest.NewRequest(http.MethodGet, "/users/1", nil)
    w := httptest.NewRecorder()

    h.GetUser(w, req)

    assert.Equal(t, http.StatusOK, w.Code)
}
```

- Unit test each handler, service, and repository independently
- Use `net/http/httptest` for handler tests
- Use interfaces for all dependencies to enable mocking
- Integration tests go in `_test` packages

---

## Environment Variables

```bash
# .env.example
PORT=8080
DATABASE_URL=postgres://user:pass@localhost/dbname
JWT_SECRET=changeme
```

Load with `godotenv` in development:

```go
import "github.com/joho/godotenv"

func init() {
    godotenv.Load() // loads .env if present, silently skips in prod
}
```
