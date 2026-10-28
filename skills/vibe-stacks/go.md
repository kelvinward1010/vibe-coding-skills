# Go

**Applies when:** `go.mod` exists. Follow the project's router (net/http, chi, gin, echo) and data access (database/sql, sqlx, sqlc, GORM, pgx).

## Dev

- Errors are values: check every `err`, wrap with context (`fmt.Errorf("load user %d: %w", id, err)`), compare with `errors.Is` / `errors.As`. Handle or return — never both log and return the same error.
- `context.Context` is the first parameter of anything doing I/O; pass the request context down to DB and HTTP calls. Never store a context in a struct.
- Small interfaces defined where they're used (consumer side), not next to the implementation.
- Every goroutine has a clear owner and exit: `errgroup`, `WaitGroup`, or context cancel. No fire-and-forget goroutines on request paths.
- Shared mutable state guarded by a mutex or owned by one goroutine via channels. Maps are not safe for concurrent writes.
- `defer rows.Close()`, `defer resp.Body.Close()` right after the error check.
- HTTP clients and servers always have timeouts (no `http.Get` with the default client in production code).
- Config via a struct loaded once at startup, passed explicitly — no package-level mutable globals.

## Review checks

- **Critical:** SQL built with `fmt.Sprintf`/string concatenation; missing authz check on handlers taking an id; data race on shared state (map written from multiple goroutines).
- Ignored errors (`_ =` or unchecked return) on anything that can fail meaningfully, including `Close` on writers and `tx.Commit`.
- Goroutine leak: blocked send/receive with no cancel path; `time.After` in a loop.
- `rows.Err()` not checked after iterating; transaction not rolled back on error paths (`defer tx.Rollback()`).
- Loop variable captured by goroutine/closure in code built with Go < 1.22 (check `go` directive in go.mod).
- `panic` used for normal error flow; `log.Fatal` outside `main`.
- Nil map write, nil pointer deref on optional fields from JSON/DB.

## Testing

- `go test ./...` with table-driven tests and `t.Run` subtests; `t.Parallel()` where tests don't share state.
- Always run `go test -race ./...` for changes touching concurrency; `go vet` and the project's linter (`golangci-lint`) must pass.
- HTTP handlers: `httptest.NewRecorder` / `httptest.NewServer`. Fake external services with small interface fakes, not mocking frameworks, unless the project already uses one.
- DB: real database via Testcontainers or the project's docker-compose; each test in a rolled-back transaction or a fresh schema.
- Coverage: `go test -coverprofile=cover.out -covermode=atomic ./...`.
