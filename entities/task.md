---
title: Task Entity
type: entity
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Definition

A Task represents a unit of work within a project. Tasks track status, priority, assignments, and due dates.

## Fields

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `id` | UUID | Yes | Primary key, auto-generated | Unique identifier |
| `project_id` | UUID | Yes | Foreign key to projects | Owning project; immutable |
| `title` | string | Yes | 1-255 chars | Task description |
| `status` | enum | Yes | `todo`, `in_progress`, `done` | Current workflow state |
| `priority` | enum | No | `low`, `medium`, `high` | Importance level; default `medium` |
| `assignee_id` | UUID | No | Foreign key to users (unvalidated) | Person responsible for task |
| `due_date` | timestamp | No | ISO-8601 format | Target completion date |
| `created_at` | timestamp | Yes | Immutable | UTC, set at creation |
| `updated_at` | timestamp | No | Auto-updated | Last modified timestamp |

## Lifecycle

1. **Creation** — Project owner creates task via `POST /projects/:id/tasks` with required title
2. **Draft** — Task starts in `todo` status by default
3. **In Progress** — Owner updates status via `PATCH /tasks/:id`
4. **Completed** — Owner marks as `done`; workflow is linear
5. **Deletion** — Owner deletes via `DELETE /tasks/:id`; hard delete, no recovery

## Relationships

- **Many-to-One: Project** — Task belongs to exactly one project
- **Zero-or-One: User (Assignee)** — Task can be assigned to a user (unvalidated)

## Constraints

- Task immutably belongs to a project; cannot move between projects
- Title is required and non-empty
- Status must be one of the three allowed enum values
- Priority is optional; defaults to `medium`
- Assignee ID is not validated; any UUID accepted (design gap)
- Due date is optional; no enforcement (overdue not alerted)
- Only project owner can perform CRUD operations

## Examples

### Create Task Request
```json
{
  "title": "Implement user authentication",
  "status": "todo",
  "priority": "high",
  "assignee_id": "7e058a39-931e-425f-b3a8-0a1d06a0f3d2",
  "due_date": "2026-08-31T23:59:59Z"
}
```

### Task Response
```json
{
  "id": "9b1897c5-559d-43dd-bbf7-268a983b6f02",
  "project_id": "8b1897c5-559d-43dd-bbf7-268a983b6f01",
  "title": "Implement user authentication",
  "status": "in_progress",
  "priority": "high",
  "assignee_id": "7e058a39-931e-425f-b3a8-0a1d06a0f3d2",
  "due_date": "2026-08-31T23:59:59Z",
  "created_at": "2026-07-14T12:30:00Z",
  "updated_at": "2026-07-14T13:15:00Z"
}
```

## Known Gaps

- No description/details field; only title
- No comments or activity log
- No task dependencies or blocking relationships
- No time tracking or estimation
- No status transition validation (e.g., can jump todo → done)
- Assignee is unvalidated; can assign to non-existent user
- No task templates or recurring tasks
- No subtasks or task breakdown
- Due date not enforced; no overdue alerts
