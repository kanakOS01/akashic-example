---
title: User Entity
type: entity
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Definition

A User represents an authenticated account in the TaskFlow system. Users own projects and can be assigned tasks.

## Fields

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `id` | UUID | Yes | Primary key, auto-generated | Unique identifier |
| `name` | string | Yes | 1-255 chars | Display name |
| `email` | string | Yes | Valid email, unique | Login credential; case-sensitive |
| `password_hash` | string | Yes | Bcrypt hash | Never returned in responses |
| `created_at` | timestamp | Yes | Immutable | UTC, set at creation |
| `updated_at` | timestamp | No | Auto-updated | Last modified timestamp |

## Lifecycle

1. **Creation** — User registers via `POST /auth/register` with name, email, password
2. **Password Hashing** — Password bcrypt-hashed before storage; plaintext never persisted
3. **Login** — User authenticates via `POST /auth/login`; JWT token issued for 24h
4. **Token Validation** — Auth middleware validates JWT on each protected request
5. **Deletion** — User deletion cascades: removes projects and tasks

## Relationships

- **One-to-Many: Projects** — User owns multiple projects
- **Zero-to-Many: Task Assignments** — User can be assignee on multiple tasks (but no membership validation)

## Constraints

- Email must be unique across all users
- Email format must be valid (basic regex validation assumed)
- Name is required but no length/content validation
- Password must meet minimum requirements (strength not enforced; see ADR)
- No username field; email is the login identifier

## Examples

### Registration Request
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "secret123"
}
```

### Token Payload (JWT Claims)
```json
{
  "user_id": "7e058a39-931e-425f-b3a8-0a1d06a0f3d2",
  "email": "jane@example.com",
  "exp": 1776097370,
  "iat": 1776010970
}
```

## Known Gaps

- No email verification; disposable addresses allowed
- No password reset mechanism
- No account deactivation; only full deletion
- No password strength requirements
- No MFA or 2FA support
- No user roles or permissions system
- No profile picture or metadata beyond name/email
