---
title: TaskFlow API Architecture
type: system
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Boundaries

The TaskFlow system is a RESTful API backend for project and task management. It does not include:
- Web/mobile frontend clients
- Real-time features (WebSockets, subscriptions)
- File storage or attachments
- Email notifications or messaging

Boundary APIs:
- **Inbound:** HTTP REST (port 8000)
- **Outbound:** PostgreSQL (database), environment-based config

## Invariants

1. **Ownership Enforcement** — Only user who owns a project can CRUD it or its tasks
2. **Cascading Deletes** — Deleting a project automatically deletes all its tasks
3. **Stateless Auth** — All auth state lives in JWT token; no server-side sessions
4. **Immutable Foreign Keys** — Project owner and task project cannot change after creation
5. **Single Ownership** — Each project/task has exactly one owner
6. **Data Isolation** — Users cannot see projects or tasks owned by other users

## Runtime Topology

```
Client Requests
    ↓
HTTP Router (Gin)
    ├→ Auth Middleware (JWT validation)
    ├→ Logger Middleware (request logging)
    ├→ Handlers (auth, projects, tasks)
    │   ├→ Services (business logic)
    │   │   ├→ Repositories (DB queries)
    │   │   ├→ JWT utilities
    │   │   └→ Password hashing
    │   └→ Error handling
    └→ Response encoding (JSON)
        ↓
    PostgreSQL
```

### Request Flow (Example: Create Project)

1. Client: `POST /projects` with `Bearer token` header
2. Router: Route to Projects handler
3. Auth Middleware: Extract user_id from JWT token
4. ProjectsHandler: Call `ProjectService.CreateProject(user_id, req)`
5. ProjectService: Validate request, call `ProjectRepository.Create()`
6. ProjectRepository: Execute `INSERT INTO projects ...` with pgxpool
7. Database: Persist row, return row with id
8. Service: Return project struct
9. Handler: Encode as JSON, send 201 Created response

## Integration Points

### Database (PostgreSQL 15)
- Used for: User credentials, project metadata, tasks
- Connection pool: pgxpool (configured via env vars)
- Migrations: golang-migrate (versioned schema)
- Transaction support: Currently not used (single-statement ops)

### JWT Authentication
- Token signing: HS256 with shared secret
- Token format: Standard JWT with `user_id`, `email`, `exp`, `iat` claims
- Validation: On every protected endpoint; request rejected if invalid
- Refresh: Not supported (user must re-login after 24h)

### Docker Entrypoint
- Runs migrations automatically on startup
- Sets DATABASE_URL from environment
- Starts API server on port 8000

## Configuration

Environment variables (required):
- `DATABASE_URL` — PostgreSQL connection string
- `JWT_SECRET` — Shared key for signing tokens (hardcoded fallback)
- `PORT` — Server port (default 8000)

Config file locations (not observed in code):
- None; all config is env-based

## Observability

### Logging
- Logger middleware logs all HTTP requests and responses
- Format: Not specified in README; check internal/middleware/logger.go

### Metrics
- None currently exposed; no Prometheus or StatsD integration

### Errors
- API returns JSON error responses with status codes
- Stack traces likely logged to stderr during development

### Debugging
- No debug mode or verbose logging observed
- SQL query logging likely available via pgxpool configuration

## Data Flow Diagram

```
User Registration/Login
    ↓
├─ Validate credentials
├─ Hash password (register) / Compare hash (login)
├─ Generate JWT token
└─ Return token to client

Authenticated Operations (Create/Read/Update/Delete)
    ↓
├─ Extract user_id from JWT
├─ Verify ownership (for PATCH/DELETE)
├─ Query/mutate database
├─ Return entity or error
└─ Encode JSON response
```

## Known Risks & Constraints

- **No transactions:** Concurrent updates could lose data; cascade delete not atomic
- **No caching:** Stats endpoint recalculates every request
- **Hardcoded defaults:** JWT secret, port, timeouts all hardcoded or env-derived
- **No rate limiting:** Login/register endpoints open to brute force
- **No pagination defaults:** Could fetch thousands of rows; needs indexing
- **Password strength:** No validation; weak passwords allowed
- **Email verification:** Not implemented; disposable emails accepted
- **CORS:** Likely not configured; cross-origin requests will fail
