---
title: TaskFlow Clean Architecture Layers
type: adr
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Status

**Accepted** — Currently implemented in project structure.

## Context

TaskFlow required a code organization strategy to:
- Separate concerns between HTTP routing, business logic, and database access
- Enable testing at multiple levels (unit, integration)
- Reduce coupling between features
- Support feature-based scalability

## Decision

Implement **Clean Architecture** with explicit layers:

1. **Handler Layer** (HTTP concerns)
   - HTTP request/response handling
   - Input validation and encoding
   - Routes and bindings

2. **Service Layer** (Business logic)
   - Core application rules
   - Data transformations
   - Orchestration between repositories

3. **Repository Layer** (Data access)
   - Database queries and mutations
   - Data persistence
   - Query optimization

### Code Organization

**Feature-based structure:**
```
internal/
  ├── auth/
  │   ├── handler.go     (HTTP endpoints)
  │   ├── service.go     (Auth logic)
  │   └── schema.go      (Request/response DTOs)
  ├── projects/
  │   ├── handler.go
  │   ├── service.go
  │   ├── repository.go
  │   ├── model.go       (Domain entity)
  │   └── schema.go
  ├── tasks/
  │   └── (same structure)
  └── middleware/
      ├── auth.go
      └── logger.go
```

## Consequences

### Positive

1. **Testability:** Each layer can be tested independently; services testable without HTTP/DB
2. **Decoupling:** Business logic independent of frameworks; database swappable
3. **Readability:** Clear separation of concerns; developers know where to look
4. **Scalability:** New features follow same pattern; reduces cognitive load
5. **Reusability:** Services callable from multiple handlers (e.g., CLI, gRPC)

### Negative

1. **Boilerplate:** Requires more files and interfaces; more code to maintain
2. **Learning Curve:** New developers must understand layer separation
3. **Overhead:** Extra indirection; small operations need to traverse three layers
4. **File Proliferation:** Many small files instead of fewer large files
5. **Enforcement:** No automatic enforcement; requires code review discipline

## Alternatives Considered

### A. Flat/Monolithic Structure
**Pros:** Fewer files; less boilerplate; faster initial development
**Cons:** Business logic mixed with HTTP concerns; hard to test; scales poorly
**Rejected:** Violates separation of concerns

### B. Modular Monolith (DDD)
**Pros:** Explicit boundaries; supports distributed systems better; rich models
**Cons:** More complex; requires deeper domain understanding; higher learning curve
**Rejected:** Over-complex for current scope

### C. Microservices
**Pros:** Independent scaling; isolated databases; technology flexibility
**Cons:** Network complexity; distributed tracing; operational overhead
**Rejected:** Premature optimization; single-instance deployment sufficient

## Implementation Evidence

**File Structure:**
- `internal/auth/` — Auth service with handler, service, schema
- `internal/projects/` — Projects service with full layers
- `internal/tasks/` — Tasks service with full layers

**Handler Example:** `internal/projects/handler.go`
- HTTP route binding via Gin router
- Request parsing and validation
- Response encoding

**Service Example:** `internal/projects/service.go`
- Business logic (ownership validation, stats calculation)
- Repository invocation
- Error handling

**Repository Example:** `internal/projects/repository.go`
- Database CRUD operations
- SQL query building
- Query optimization

## Open Questions / Gaps

1. **DTOs vs. Entities:** Currently minimal distinction; should we separate explicitly?
2. **Error Handling:** How deep should errors propagate? Should each layer have custom errors?
3. **Middleware Injection:** How do we pass logger, tracer, etc. through layers?
4. **Testing Doubles:** Should we mock repositories or use test database?

## Related ADRs

- `adr/taskflow-database-transactions.md` (proposed)
- `adr/taskflow-error-handling-strategy.md` (proposed)
