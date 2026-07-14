---
title: TaskFlow Stateless JWT Authentication
type: adr
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Status
chutieyapoajsdlfjas;ldkfgjalskdfjlskdjf
**Accepted** — Currently implemented in auth service.

## Context

TaskFlow is a REST API that needs to authenticate requests and authorize operations. The decision was between:
1. Server-side session storage (e.g., Redis, in-memory)
2. Stateless JWT tokens
3. OAuth/OpenID Connect

Key constraints:
- Horizontal scalability (multiple API instances)
- Simplicity (minimal dependencies)
- Standard HTTP authentication

## Decision

Use **JSON Web Tokens (JWT)** with HS256 signing algorithm. Tokens are stateless, cryptographically signed, and issued for 24-hour validity.

### Implementation Details

- **Token Claims:** `{ user_id, email, exp, iat }`
- **Signing Algorithm:** HS256 (HMAC-SHA256)
- **Secret Key:** JWT_SECRET from environment (shared across API instances)
- **Expiration:** 24 hours from issue time
- **Validation:** Performed on every protected endpoint via auth middleware

## Consequences

### Positive

1. **Stateless:** No server-side session storage needed; API scales horizontally
2. **Self-contained:** Token includes user identity; no DB lookup needed for auth
3. **Standard:** Industry-standard format; compatible with most clients and tools
4. **Decentralized:** Each API instance can validate independently using shared secret

### Negative

1. **No Revocation:** Issued tokens cannot be revoked until expiration; logout is client-side only
2. **No Refresh:** 24-hour window is fixed; users must re-login after expiration
3. **Large Payload:** JWT tokens larger than session cookies; more bandwidth per request
4. **Secret Key Rotation:** Changing JWT_SECRET invalidates all existing tokens
5. **Clock Skew Risk:** Client/server time mismatch can invalidate tokens prematurely

## Alternatives Considered

### A. Server-Side Sessions (Redis/In-Memory)
**Pros:** Revocable, short-lived, standard HTTP cookie auth
**Cons:** Requires session store dependency; breaks horizontal scaling; adds latency
**Rejected:** Violates scalability goal

### B. API Keys
**Pros:** Simple, long-lived, no expiration management
**Cons:** No user context; poor for multi-user systems; harder to rotate
**Rejected:** Unsuitable for per-user access control

### C. OAuth 2.0 / OpenID Connect
**Pros:** Industry standard; supports delegation; token refresh built-in
**Cons:** Complex; requires additional setup; overkill for internal API
**Rejected:** Over-engineered for current use case

## Implementation Evidence

**File:** `backend/internal/auth/service.go`
- `AuthService.Login()` generates JWT with 24h expiration
- `AuthService.ValidateToken()` verifies signature

**File:** `backend/internal/middleware/auth.go`
- Auth middleware extracts and validates token on protected routes
- Rejection returns 401 Unauthorized

**File:** `backend/internal/utils/jwt.go`
- JWT utility functions for signing and parsing

## Open Questions / Gaps

1. **Token Refresh:** Should we implement refresh tokens for smoother UX? (See separate ADR)
2. **Token Revocation:** How should logout work if we can't revoke tokens?
3. **Secret Rotation:** What's the process for rotating JWT_SECRET without disrupting users?
4. **Asymmetric Signing:** Should we use RS256 with separate sign/verify keys?

## Related ADRs

- `adr/taskflow-refresh-tokens.md` (proposed)
- `adr/taskflow-password-strength-policy.md` (proposed)
