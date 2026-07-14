---
title: Project Entity
type: entity
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Definition

A Project is a container for related tasks, owned by a single user. Projects provide organizational structure and stats aggregation.

## Fields

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `id` | UUID | Yes | Primary key, auto-generated | Unique identifier |
| `owner_id` | UUID | Yes | Foreign key to users | Project owner; immutable |
| `name` | string | Yes | 1-255 chars | Display name |
| `description` | string | No | 0-2000 chars | Optional details; patchable |
| `created_at` | timestamp | Yes | Immutable | UTC, set at creation |
| `updated_at` | timestamp | No | Auto-updated | Last modified timestamp |

## Lifecycle

1. **Creation** — Owner creates project via `POST /projects` with name and optional description
2. **Active** — Project holds tasks; user can list and query via `GET /projects` and `GET /projects/:id`
3. **Modification** — Owner can update description via `PATCH /projects/:id`
4. **Stats Query** — Owner can fetch aggregated stats via `GET /projects/:id/stats`
5. **Deletion** — Owner deletes via `DELETE /projects/:id`; cascades delete all tasks

## Relationships

- **Many-to-One: User (Owner)** — Exactly one owner per project
- **One-to-Many: Tasks** — Project contains zero or more tasks

## Constraints

- Owner is immutable after creation
- Only the owner can view, update, or delete the project
- Name is required; no blank projects
- Description is optional and unbounded (should add max length)
- Deletion cascades to all tasks in project

## Examples

### Create Project Request
```json
{
  "name": "Q3 Product Roadmap",
  "description": "Features and bugs for Q3 2026"
}
```

### Project Response
```json
{
  "id": "8b1897c5-559d-43dd-bbf7-268a983b6f01",
  "owner_id": "0dfba3db-ffb9-49df-887c-2ebba8c24088",
  "name": "Q3 Product Roadmap",
  "description": "Features and bugs for Q3 2026",
  "created_at": "2026-07-14T12:30:00Z",
  "updated_at": "2026-07-14T12:30:00Z"
}
```

### Stats Response
```json
{
  "total_tasks": 24,
  "completed_tasks": 8,
  "pending_tasks": 16,
  "in_progress_tasks": 0,
  "high_priority_tasks": 5,
  "overdue_tasks": 2
}
```

## Known Gaps

- No project-level access control (e.g., shared projects, roles)
- No archive/soft-delete; deletion is permanent
- No project status or visibility flags (all private to owner)
- No project templates or copying
- Description length is unbounded
- No project tags or categories
