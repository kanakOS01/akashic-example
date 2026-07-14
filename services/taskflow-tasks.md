---
title: TaskFlow Tasks Service
type: service
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Purpose

The Tasks service manages individual task objects within projects, including status lifecycle (todo → in_progress → done), priority assignment, and due date tracking.

## Public API

### HTTP Endpoints
- **`POST /projects/:id/tasks`** — Create task in project
  - Auth: required (project owner only)
  - Body: `{ title: string, status?: string, priority?: string, assignee_id?: UUID, due_date?: ISO8601 }`
  - Returns: `{ id, project_id, title, status, priority, assignee_id, due_date, created_at }`
- **`GET /projects/:id/tasks`** — List tasks in project
  - Auth: required (project owner only)
  - Query: `?status=todo&assignee=UUID&page=1&limit=10`
  - Returns: `[{ id, title, status, priority, ... }]`
- **`PATCH /tasks/:id`** — Update task
  - Auth: required (project owner only)
  - Body: `{ due_date?: ISO8601, ... }` (currently only due_date supported)
- **`DELETE /tasks/:id`** — Remove task
  - Auth: required (project owner only)

### Service Layer
- **`TaskService.CreateTask(ctx, projectID, req) -> (task, error)`**
- **`TaskService.ListTasks(ctx, projectID, filters) -> (tasks, error)`**
- **`TaskService.UpdateTask(ctx, taskID, updates) -> (task, error)`**

## Dependencies

- PostgreSQL — Tasks table, foreign key to projects
- Projects service — Project ownership validation
- Auth middleware — User ID extraction

## Data Ownership

Owns:
- Task metadata (title, status, priority, due date)
- Task-to-project relationship
- Status lifecycle (todo → in_progress → done)

Delegates to:
- Repository layer — Database persistence
- Projects service — Ownership verification

## Operational Notes

- All task operations require authenticated user to be project owner
- Status enum: `todo`, `in_progress`, `done` (enforced in schema)
- Priority enum: `low`, `medium`, `high` (enforced in schema)
- Due date is optional; no auto-completion or reminder system
- Assignee field exists but no validation against project members (anyone can be assigned)
- Pagination on list: default page=1, limit=10

## Known Risks

- Assignee validation missing; any UUID can be assigned (should validate against project members)
- No status transition validation (could jump from todo to done, or invalid state)
- No soft delete; deleted tasks are unrecoverable
- Due date has no enforcement; overdue tasks not auto-escalated or alerted
- Concurrent task updates could result in lost updates (no optimistic locking)
- Search/filter only by status and assignee; no full-text search on title
- No task dependency tracking; can't model blocking relationships
