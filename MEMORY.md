# Akashic Knowledge Index

Generated at 2026-07-14 from gravitype and taskflow repositories.

## Services

- [Gravitype Game Core](services/gravitype-game-core.md) — Word generation, scoring, level progression
- [Gravitype UI Engine](services/gravitype-ui-engine.md) — Textual TUI, screen management, rendering
- [TaskFlow Authentication](services/taskflow-auth.md) — User registration, login, JWT validation
- [TaskFlow Projects](services/taskflow-projects.md) — Project CRUD, ownership, statistics
- [TaskFlow Tasks](services/taskflow-tasks.md) — Task CRUD, status lifecycle, filtering

## Entities

- [User](entities/user.md) — Authentication account; owns projects in TaskFlow
- [Project](entities/project.md) — Container for tasks; owned by single user
- [Task](entities/task.md) — Unit of work; belongs to project; has status, priority, assignee
- [Game Word](entities/game-word.md) — Falling word in Gravitype; has position, speed, category

## Flows

- [TaskFlow User Registration](flows/taskflow-user-registration.md) — Email signup, password hashing, JWT issuance
- [Gravitype Game Session](flows/gravitype-game-session.md) — Game start, word spawning, scoring, game over

## System Architecture

- [TaskFlow API Architecture](system/taskflow-architecture.md) — Clean layers, ownership invariants, database integration
- [Gravitype Game Architecture](system/gravitype-architecture.md) — Textual framework, game loop, state machine

## Architecture Decisions

- [TaskFlow JWT Authentication](adr/2026-07-14-taskflow-jwt-auth.md) — Stateless tokens, 24h expiration, HS256 signing
- [TaskFlow Clean Architecture](adr/2026-07-14-taskflow-clean-architecture.md) — Handler/Service/Repository separation
- [Gravitype Textual UI](adr/2026-07-14-gravitype-textual-ui.md) — Rich TUI framework over raw terminal I/O

## Reference

- [Glossary](glossary/glossary.md) — Terminology, acronyms, conventions
