---
title: TaskFlow Authentication Service
type: service
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Purpose

The Authentication service handles user registration, login, and JWT token generation/validation. All API access requires a valid JWT token (except `/auth/register` and `/auth/login`).

## Public API

### HTTP Endpoints
- **`POST /auth/register`** — Create new user account
  - Body: `{ name: string, email: string, password: string }`
  - Returns: JWT token and user ID
- **`POST /auth/login`** — Authenticate existing user
  - Body: `{ email: string, password: string }`
  - Returns: JWT token with 24h expiration

### Service Layer
- **`AuthService.Register(ctx, req) -> (token, userID, error)`**
- **`AuthService.Login(ctx, req) -> (token, error)`**
- **`AuthService.ValidateToken(tokenString) -> (claims, error)`**

## Dependencies

- PostgreSQL — User credentials and hash storage
- JWT library (Go stdlib `crypto/hmac`) — Token signing
- Password hashing — bcrypt equivalent for secret validation

## Data Ownership

Owns:
- User credentials (email, hashed password)
- JWT token generation and validation logic
- User ID and email extraction from tokens

Delegates to:
- Repository layer — Database CRUD for users table

## Operational Notes

- JWT tokens expire in 24 hours (hardcoded; see ADR for refresh token discussion)
- Passwords must be bcrypt-hashed before storage; plaintext passwords rejected during login
- Email is unique per user; register fails if email already exists
- Token validation happens in auth middleware on every protected endpoint
- No rate limiting on login/register endpoints

## Known Risks

- No refresh token mechanism; users must re-authenticate after 24h
- No password reset flow; forgotten credentials are unrecoverable
- Email is not verified at registration; disposable email addresses allowed
- JWT secret likely hardcoded in config; rotation not implemented
- No CORS headers set; cross-origin requests will fail
- Login attempts not rate-limited; brute-force attacks possible
