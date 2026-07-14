---
title: TaskFlow Projects Service
type: service
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Purpose

The Projects service manages the lifecycle of project containers and provides statistics. All projects belong to an owner; only the owner can modify or delete them.

## Public API

### HTTP Endpoints
- **`POST /projects`** — Create new project
  - Auth: required (Bearer token)
  - Body: `{ name: string, description?: string }`
  - Returns: `{ id, name, description, owner_id, created_at }`
- **`GET /projects`** — List all projects for authenticated user
  - Auth: required
  - Returns: `[{ id, name, owner_id, created_at, ... }]`
- **`GET /projects/:id`** — Fetch single project
  - Auth: required
  - Returns: Project with full details
- **`PATCH /projects/:id`** — Update project description
  - Auth: required (owner only)
  - Body: `{ description: string }`
- **`DELETE /projects/:id`** — Delete project and all tasks
  - Auth: required (owner only)
  - Cascades: removes all tasks in project
- **`GET /projects/:id/stats`** — Project statistics
  - Auth: required (owner only)
  - Returns: `{ total_tasks, completed_tasks, pending_tasks, total_duration, ... }`

## Dependencies

- PostgreSQL — Projects table, foreign key to users
- Tasks service — Cascade delete; stats aggregation
- Auth middleware — User ID extraction from JWT

## Data Ownership

Owns:
- Project metadata (name, description, creation timestamp)
- Owner-to-project relationship
- Project stats calculation (counts, aggregations)

Delegates to:
- Repository layer — Database persistence
- Tasks service — Task lifecycle within projects

## Operational Notes

- Projects are isolated by owner; user cannot access another's projects
- Stats endpoint is expensive (aggregates all tasks); consider caching
- Description update is PATCH only; other fields immutable
- Cascade delete removes all tasks; no soft delete or recovery

## Known Risks

- No pagination on `GET /projects`; large user accounts will fetch all at once
- Stats calculation queries all tasks every request; no caching or materialized view
- Concurrent project creation could create duplicates if validation is async
- No archive/soft-delete; deleted projects are unrecoverable
- Description field is unbounded; no length validation could cause storage bloat
