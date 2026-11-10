# Go Layered Backend: Caller-Owned Interfaces

A Go HTTP service stays testable when layers are permission boundaries, not folders. The Heptabase **go-backend** board (child of Learn Go, 37 cards by 2026-09-20) treats this as the production shape: handler never writes SQL, service never imports Gin, and the interface is owned by the caller.

## Dependency only points down

If HTTP, business rules, and SQL share one function, you cannot swap Postgres without rewriting tests, and you cannot prove "duplicate mail is read-only" without a live database. The lock is **caller-owned interface**: the layer that needs a capability defines the interface; the implementation lives further in.

Three layers plus one composition root:

```
cmd/server/main.go          composition root — the only place that sees everyone
        │
        ▼
internal/handler            HTTP (Gin). Parse, validate, status code.
        │  interface CaseService
        ▼
internal/service            rules. Duplicates, state hops, archive TTL.
        │  interface CaseRepository
        ▼
internal/db                 SQL. Rows ↔ model types.
```

`internal/model` is not a layer. It is the OpenAPI-generated contract both sides already agreed on. YAML is the source; generated Go is not hand-edited.

| Layer | May | Must not |
|---|---|---|
| handler | read `gin.Context`, bind JSON, call service with `c.Request.Context()`, `c.JSON` | run SQL, know table names, import `internal/db` |
| service | `context.Context` + `model.*`, call repo, return sentinel errors | import gin, `c.JSON`, open `*sql.DB` |
| db | `QueryRowContext` / `ExecContext` | decide HTTP status, invent duplicate policy |
| main | wire config, logger, pool, services, router | hold business rules |

If service "needs" Gin, the rule leaked into HTTP. If handler "needs" SELECT, I/O leaked upward.

## Compile-time interface check

Go implements interfaces implicitly. A struct that is never assigned to the interface type will not fail the build when OpenAPI adds a method. The board's lock is:

```
var _ handler.CaseService = (*CaseServiceImpl)(nil)
var _ service.CaseRepository = (*PostgresCaseRepository)(nil)
var _ ServerInterface = (*Handler)(nil)
```

`(*T)(nil)` costs no heap. Missing methods fail at compile time, not on the first request.

## Context, logs, tests

- **`gin.Context` is the HTTP envelope.** It does not travel into service or db. Pass `c.Request.Context()`. A new `context.Background()` starts an unrelated trace tree. See [[opentelemetry-lgtm]].
- Context values are pass-by-value headers whose pointers still share cancel state, which is why `WithCancel` / `WithTimeout` can broadcast through `Done()` closing a channel.
- Zap logs structured JSON fields, not string concatenation. Trace join depends on the field name matching Alloy/Grafana (`trace_id`, not `traceId`).
- Each layer uses the cheapest fake: stub the interface below it. Do not boot Postgres to test HTTP status mapping.
- Parameterized queries (`pgx`) are the SQL-injection boundary. Connection pooling is a resource, not a convenience wrapper.

## Related pages

- [[go-runtime-anatomy]]: scheduler, GC, and channel internals underneath the same Go oral.
- [[go-memory-model]]: arrays, slices, strings, maps.
- [[opentelemetry-lgtm]]: why `c.Request.Context()` is the only envelope that keeps spans one tree.

## References

