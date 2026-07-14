---
title: TaskFlow User Registration Flow
type: flow
source_repositories:
  - taskflow
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Trigger

User initiates account creation by submitting registration form with name, email, and password.

## Actors

- **User** — Client application or API consumer
- **Auth Handler** — HTTP request handler for `/auth/register`
- **Auth Service** — Business logic layer for user registration
- **Repository** — Database access layer
- **PostgreSQL** — User credentials storage

## Ordered Steps

1. **User Submits Registration**
   - POST request to `/auth/register`
   - Body: `{ name: "Jane Doe", email: "jane@example.com", password: "secret123" }`

2. **Handler Receives Request**
   - Auth handler parses JSON body
   - Validates request structure (name, email, password required)
   - Passes to AuthService

3. **Service Validation**
   - Check if email already exists in database (query via repository)
   - Return conflict error if duplicate email found
   - Validate email format (basic regex assumed)

4. **Password Hashing**
   - Hash password using bcrypt with salt
   - Password never stored in plaintext
   - Hashed value stored in database

5. **User Creation**
   - Call `UserRepository.Create()` with name, email, password_hash
   - Database generates UUID for user_id
   - Insert row into users table
   - Return created user struct with id, created_at

6. **JWT Token Generation**
   - Create JWT claims with user_id, email, exp (24h from now), iat
   - Sign with HS256 algorithm and JWT_SECRET
   - Encode token as JSON Web Token string

7. **Success Response**
   - Return 201 Created status
   - Body: `{ user_id: UUID, token: JWT, expires_at: timestamp }`
   - User receives token for authenticated requests

## Branch Cases

### Email Already Exists
- **Trigger:** Step 3 finds duplicate email
- **Action:** Return 409 Conflict error
- **Message:** "Email already registered"
- **Resolution:** User must use different email or login instead

### Invalid Email Format
- **Trigger:** Step 3 validation fails
- **Action:** Return 400 Bad Request
- **Message:** "Invalid email format"
- **Resolution:** User corrects email and retries

### Database Connection Failure
- **Trigger:** Step 5 cannot connect to PostgreSQL
- **Action:** Return 500 Internal Server Error
- **Message:** "Database error"
- **Resolution:** Server operator investigates; user retries later

### Weak Password (Gap)
- **Trigger:** Currently never checked (design gap)
- **Current Behavior:** Any password accepted, even empty or single character
- **Recommended:** Add minimum length (12 chars) and complexity checks
- **See ADR:** `adr/password-strength-policy.md` (proposed)

## External Calls

- **PostgreSQL:** `SELECT email FROM users WHERE email = ?` (check uniqueness)
- **PostgreSQL:** `INSERT INTO users (name, email, password_hash) VALUES (?, ?, ?)`

## Failure Modes

| Scenario | Impact | Recovery |
|----------|--------|----------|
| Email already in use | User cannot register | Use different email or login |
| Network timeout during insert | Partial state; may retry | User retries; check idempotency |
| Password hash library crash | Request fails with 500 | Server operator investigates |
| JWT secret missing | Token generation fails | Ensure JWT_SECRET in environment |

## Resulting State

### User Created Successfully
- New row in users table with id, name, email, password_hash
- User can login with email/password
- User can create projects and tasks
- High score and config stored in separate table (not implemented)

### User Not Created
- No database changes
- User must try again with valid input
- Error message indicates failure reason

## Success Criteria

- HTTP 201 Created returned
- JWT token is valid and can be decoded
- User can login with email/password
- User can access protected endpoints with token
