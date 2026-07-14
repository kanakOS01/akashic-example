---
title: Akashic Glossary
type: reference
source_repositories:
  - gravitype
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Common Terms

### TaskFlow

- **JWT (JSON Web Token)** — Stateless authentication token signed with HS256. Contains user_id, email, exp, iat claims. 24-hour validity.

- **Ownership-Based Access** — Authorization model where only the user who created a project can modify or delete it. All operations restricted to owner.

- **Clean Architecture** — Code organization into Handler (HTTP), Service (business logic), and Repository (database) layers. Feature-based file structure.

- **Cascade Delete** — When a project is deleted, all tasks within it are automatically deleted from the database.

- **Stateless Authentication** — Authentication approach where server doesn't maintain session state. All user identity information is in the JWT token itself.

- **pgxpool** — PostgreSQL connection pooling library for Go. Manages database connections efficiently.

- **golang-migrate** — Database migration tool for Go. Versioned schema management; runs automatically on startup.

- **Repository Pattern** — Data access abstraction layer that encapsulates database queries. Enables swapping implementations without changing services.

### Gravitype

- **Textual Framework** — Python TUI (Text User Interface) framework. Provides widgets, event handling, CSS styling, reactive state management.

- **Reactive Properties** — Variables in Textual that trigger UI updates automatically when value changes. Used for score, level, lives, high_score.

- **Falling Words** — Word objects that move down the screen at speed determined by current level. Player must type before they reach the bottom.

- **Screen Stack** — Textual's navigation model. Screens are pushed/popped: main → game → game_over → back to main.

- **CSS Compilation** — Theme CSS combined with base styles into `theme_active.tcss` at app startup. Enables hot-reload during development.

- **Word Pool** — Category-specific list of words from which falling words are selected. Three pools: tech, general, mixed.

- **Level Progression** — Calculated as `1 + (total_score // 150)`. Higher level = faster falling words = harder difficulty.

- **Score Multiplier** — Points awarded for correct word match, scaled by current level. Encourages playing at higher difficulties.

## Cross-Project Terms

- **User** — Authenticated account in the system. TaskFlow users are project owners. Gravitype has no user concept (single-player).

- **UUID** — Universally Unique Identifier. Used for all entity IDs in TaskFlow (user, project, task).

- **Pagination** — Limiting and offsetting results from database queries. TaskFlow tasks endpoint supports `?page=1&limit=10`.

- **Status Enum** — Fixed set of allowed values. TaskFlow tasks have status: `todo`, `in_progress`, `done`. Gravitype games have status: game_over (implicit).

- **Timestamp** — UTC datetime in ISO-8601 format. Used for created_at, updated_at, due_date, token expiration.

- **Environment Variable** — Configuration passed to application at startup. TaskFlow uses DATABASE_URL, JWT_SECRET, PORT.

- **Config Persistence** — Saving user settings to filesystem. TaskFlow stores in environment; Gravitype stores in `.gravitype_config.json`.

## Acronyms

| Acronym | Expansion | Used In |
|---------|-----------|---------|
| JWT | JSON Web Token | TaskFlow auth |
| API | Application Programming Interface | TaskFlow REST endpoints |
| RBAC | Role-Based Access Control | TaskFlow gap (not implemented) |
| HS256 | HMAC SHA-256 | JWT signing algorithm |
| RS256 | RSA SHA-256 | Alternative signing (not used) |
| UUID | Universally Unique Identifier | TaskFlow entity IDs |
| CRUD | Create, Read, Update, Delete | Repository operations |
| HTTP | HyperText Transfer Protocol | TaskFlow transport |
| JSON | JavaScript Object Notation | Request/response format |
| SQL | Structured Query Language | Database queries |
| TUI | Text User Interface | Gravitype |
| CSS | Cascading Style Sheets | Gravitype styling |
| FPS | Frames Per Second | Gravitype rendering target |
| UX | User Experience | Design concern |

## Operators & Symbols

- `->` — Type signature (return type). Example: `get_words(category: str) -> List[str]`
- `?` — Optional field. Example: `description?: string`
- `==` — Equality comparison
- `//` — Integer division (Python). Example: `level = 1 + (score // 150)`
- `||` — Logical OR (Go/TypeScript). Example: `a || b`
- `&&` — Logical AND
- `:` — Type annotation. Example: `name: string`
- `;` — Statement terminator (Go)
- `@` — Decorator. Example: `@on(Button.Pressed)`

## File Extensions

| Extension | Meaning | Used In |
|-----------|---------|---------|
| `.py` | Python source | Gravitype |
| `.go` | Go source | TaskFlow |
| `.ts` | TypeScript source | (Unused in current projects) |
| `.tsx` | TypeScript + JSX | (Unused in current projects) |
| `.toml` | TOML config | Gravitype pyproject.toml |
| `.yaml` | YAML config | Pre-commit config |
| `.json` | JSON data | API spec, config |
| `.tcss` | Textual CSS | Gravitype styling |
| `.md` | Markdown | Documentation |
| `.sql` | SQL script | Database migrations |

## Standards & Conventions

- **ISO-8601:** Date/time format used in all APIs and configs. Example: `2026-07-14T12:30:00Z`
- **OpenAPI 3.0:** API specification format for TaskFlow (api_spec.json)
- **Bearer Token:** HTTP Authorization header format for JWT. Example: `Authorization: Bearer eyJ...`
- **REST:** HTTP verb semantics. GET (read), POST (create), PATCH (update), DELETE (delete)
- **Camel Case:** Variable naming in APIs and schemas. Example: `user_id`, `due_date`
- **Snake Case:** Python naming. Example: `user_id`, `is_active`
- **PascalCase:** Go type names and classes. Example: `ProjectService`, `AuthHandler`
