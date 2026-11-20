Professional Node.js, TypeScript & MySQL Boilerplate

A production-ready REST API starter built with Node.js, Express, TypeScript, TypeORM, and MySQL. It features a clean layered architecture, centralized error handling, Joi request validation, structured Winston logging, security middleware, database migrations, and a complete User CRUD vertical slice.

Features

- Layered Architecture: Cleanly separates routes, controllers, services, entities, dto, and middleware.

- TypeScript Strict Mode: Enforces strict type safety across models, queries, and handlers.

- Database Migrations & Seeders: Uses TypeORM migrations (synchronize: false) and reusable seed scripts for safe schema updates.

- Centralized Error Handling: Custom error classes (NotFoundError, ConflictError, ValidationError) paired with asyncHandler for clean controllers and uniform JSON errors.

- Input & Environment Validation: Validates .env variables at startup and incoming body, params, and query payloads using Joi.

- Structured Logging: Winston logs JSON records with file rotation (combined.log, error.log) and unique X-Request-Id tracing.

- Security & Reliability: Pre-configured with Helmet, CORS, Express Rate Limiting, bcrypt password hashing, /health checks, and graceful shutdowns.

Setup Instructions

1. Create folder named professional-boilerplate
2. Open the folder in VS Code 
3. Install all dependencies
4. Setup all folder structures with its respective files
5. Configure .env to connect to the database
6. Run migrations & seed initial data
   npm run migration:generate -- src/migrations/InitialSchema
   npm run migration:run 
   npm run seed 
7. npm run dev 


Environment Variables documentation

Variable                         Required                 Default                       Description
NODE_ENV                           No                  development                    Environment mode (development, production, test)PORT                               No                  3000                           Server port
DB_HOST                            Yes                 —                              MySQL host (e.g., localhost)DB_PORT                            No                  3306                           MySQL port
DB_USERNAME                        Yes                 —                              MySQL username
DB_PASSWORD                        Yes                 —                              MySQL password
DB_DATABASE                        Yes                 —                              Database name (boilerplate_db)
FRONTEND_URL                        No                http://localhost:4200           Allowed CORS origin

API endpoints documentation

API Endpoints
Health CheckGET /health — Returns server uptime and database connection status (200 OK or 503 Service Unavailable).

Users (/api/users)
Method             Endpoint                 Description               Body / Query
GET                /api/users               Get paginated users       Query: page, limit, sort, order
GET                /api/users/:id           Get a single user by ID   Param: id (positive integer)
POST               /api/users               Create a new user         { "name", "email", "password", "age"? }
PUT                /api/users/:id           Update an existing user   { "name"?, "email"?, "age"?, "isActive"? }
DELETE             /api/users/:id           Delete a user             Param: id (returns 204 No Content)


How to run migrations and seeders
npm run migration:generate -- src/migrations/InitialSchema
npm run migration:run 
npm run migration:revert
npm run seed 


Reflection Answers

1. What was the most challenging part of building this boilerplate? How did you overcome it?
Resolving strict TypeScript and runtime compatibility issues between TypeORM v0.3+, modern Express, and ts-node. I fixed TypeORM query errors by switching to object-map select syntax ({ id: true }) and casting sort keys (keyof User), and resolved Express's read-only req.query getter error by using Object.defineProperty inside the validation middleware.

2. How does this boilerplate differ from your Midterm project? What specific improvements did you make?
Unlike the Midterm project, which kept logic tightly coupled inside route handlers and relied on basic error responses, this boilerplate introduces a strict layered architecture (Routes → Controllers → Services), fail-fast .env validation, versioned database migrations, structured file logging, and production security middleware (Helmet, CORS, rate limiting).

3. Why is centralized error handling better than try-catch in every controller?
It eliminates repetitive try-catch blocks across controllers via asyncHandler, guarantees a uniform JSON error format for all clients, and ensures every error is logged in one place while hiding sensitive stack traces in production.

4. How does request validation with Joi improve security and code quality?
Joi blocks malformed data and prevents mass-assignment attacks (stripUnknown: true) before requests ever reach the database. For code quality, it automatically converts types (e.g., query strings to numbers) and keeps controllers clean and focused on core logic.

5. What's the difference between synchronize: true and migrations? Why are migrations essential for production?
synchronize: true automatically alters database tables on startup, which can accidentally drop columns and destroy live data. Migrations are version-controlled SQL scripts (up and down) that let teams safely review, deploy, and roll back schema changes without data loss.

6. How does structured logging (Winston) help in production debugging compared to console.log?
Winston outputs searchable JSON logs with timestamps, severity levels, and unique X-Request-Id tags to trace specific requests across the app. It also separates error logs (error.log) from general traffic (combined.log) and rotates files automatically so logs don't fill up server storage.

7. If you had to add a new feature (e.g., Product CRUD), how would you use this boilerplate? What would you need to add?
I would follow the exact same vertical-slice pattern without modifying core infrastructure:

 1. Create src/entities/Product.ts and register it in data-source.ts.
 2. Generate and run a migration (npm run migration:generate & migration:run).
 3. Add Joi schemas in src/dto/product.dto.ts.
 4. Implement business logic in src/services/productService.ts.
 5. Build src/controllers/productController.ts and src/routes/productRoutes.ts, then register the route in src/index.ts.

8. What additional features would you add to make this boilerplate even more production-ready?
I would add JWT authentication & role-based access control (RBAC), automated unit/integration tests with Jest and Supertest, Swagger/OpenAPI documentation, and Docker configuration for consistent cloud deployments.



Professional Node.js, TypeScript & MySQL Boilerplate (Now with JWT Authentication!)

Project Description

A production-ready REST API boilerplate built with Node.js, Express, TypeScript, TypeORM, and MySQL—now upgraded with stateless JWT (JSON Web Token) authentication.

This backend combines a clean layered architecture (routes, controllers, services, entities, dto, and middleware), centralized error handling, Joi request and environment validation, structured Winston logging, security hardening (Helmet, CORS, and rate limiting), and safe TypeORM database migrations with full user authentication and route protection.

Authentication Flow Documentation

This API uses stateless JWT authentication to verify user identity and secure private resources without storing session data in server memory:

1. Login Request: A user sends their registered email and password to POST /api/auth/login.

2. Credential Verification: AuthService looks up the user, verifies the account is active (isActive: true), and compares the plain-text password against the stored bcrypt hash.

3. Token Issuance: Upon successful verification, the server generates a signed JWT containing { userId, email }, signed with JWT_SECRET and configured with an expiration window (JWT_EXPIRES_IN).

4. Authenticated Requests: For subsequent calls to protected routes, the client attaches the JWT in the Authorization HTTP header.

5. Middleware Verification: authMiddleware intercepts incoming requests, validates the Bearer format, verifies the token's signature and expiration, attaches the decoded user payload to req.user, and passes control to the controller (or throws a 401 Unauthorized error if invalid/missing).


How to Get a Token (Login Endpoint)

Send a POST request to /api/auth/login with a valid user's credentials:

Endpoint: POST /api/auth/login

Headers: Content-Type: application/json

Request Body:

{
  "email": "test@example.com",
  "password": "Password123"
}


Response (200 OK):

{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": 1,
      "email": "test@example.com",
      "name": "Test User"
    }
  }
}


How to Use the Token (Authorization Header)

Include the token returned from the login endpoint in the Authorization header using the Bearer scheme on all protected requests:

Authorization: Bearer <YOUR_JWT_TOKEN>


Requests sent without an Authorization header, with an improper prefix, or with an expired/tampered token will automatically be rejected with 401 Unauthorized.

List of Protected vs. Public Endpoints
Public Endpoints (No Token Required)
Method         Endpoint            Description                                      Request Body 
GET            /health             Check server uptime and database connectivity       —
POST           /api/auth/login     Authenticate user credentials and receive a JWT  { "email", "password" }
POST           /api/users          Register / create a new user account             { "name", "email", "password", "age"? }


Protected Endpoints (Authorization: Bearer <token> Required)
Method         Endpoint            Description                                            Params / Body / Query
GET            /api/auth/me        Retrieve the currently authenticated user's profile    —   
GET            /api/users          Retrieve a paginated list of all users                 Query: page, limit, sort, order
GET            /api/users/:id      Retrieve a single user by ID                           Param: idPUT/api/users/:idUpdate an existing user's detailsParam: id | Body: { "name"?, "email"?, "age"?, "isActive"? }
DELETE         /api/users/:id      Delete a user account (returns 204 No Content)         Param: id

Reflection Answers
1. What is the difference between authentication and authorization? Give a real-world example of each.
Authentication (AuthN) verifies who a user is, while Authorization (AuthZ) determines what that verified user is permitted to access or do.Real-world example: Scanning your employee ID badge at a building entrance is authentication (proving your identity). Whether that badge unlocks the executive floor or only the main lobby is authorization.

2. Why is JWT considered "stateless" and how does that benefit scalability?
A JWT is stateless because all user claims (such as userId and email) and verification metadata are embedded directly inside the signed token rather than stored in server memory or a session database. This makes horizontal scaling effortless: any server instance behind a load balancer can independently validate incoming tokens using JWT_SECRET without needing shared session storage.

3. What are the three parts of a JWT token and what does each part do?A JWT consists of three Base64Url-encoded sections separated by dots (header.payload.signature):Header: Identifies the token type (JWT) and the cryptographic signing algorithm used (e.g., HS256).Payload: Holds the claims or user data (e.g., userId, email, issued-at iat, and expiration exp).Signature: A cryptographic hash generated from the encoded header, encoded payload, and secret key (JWT_SECRET) that guarantees the token has not been altered.

4. Why should you never put sensitive data (like passwords) in the JWT payload?The header and payload of a standard JWT are only Base64Url-encoded, not encrypted. Anyone who intercepts or inspects the token (such as pasting it into jwt.io) can immediately decode and read the payload in plain text.
5. How does bcrypt protect user passwords? What is the purpose of the "salt" and "rounds"?bcrypt is a one-way password-hashing function that prevents plain-text passwords from ever being stored in the database.Salt: A unique, random string added to each password before hashing so that two users with the exact same password produce completely different hashes, neutralizing rainbow table attacks.Rounds (Cost Factor): Determines how many iterations ($2^{\text{rounds}}$) the hashing algorithm executes, intentionally slowing down computation to make brute-force attacks impractical.

6. What happens if someone steals a JWT token? How can you mitigate this risk?Because JWTs are self-contained and stateless, anyone who obtains a valid token can impersonate that user until the token expires. Risks can be mitigated by:Keeping token lifetimes short (JWT_EXPIRES_IN).Storing tokens in HttpOnly, Secure, SameSite cookies rather than localStorage to defend against XSS attacks.Maintaining a token blacklist/revocation store (e.g., in Redis) to invalidate tokens immediately upon logout or suspicious activity.

7. Why is it important to use HTTPS when transmitting JWT tokens?Over unencrypted HTTP, network packets—including the Authorization: Bearer <token> header—travel in plain text, allowing attackers on the same network (Man-in-the-Middle attacks) to intercept and steal the token. HTTPS encrypts the entire connection in transit.

8. What is the difference between a short-lived access token and a long-lived refresh token? When would you use each?Access Token: Short-lived (e.g., 15 minutes) and attached to every API request to access protected endpoints, limiting the window of exposure if compromised.Refresh Token: Long-lived (e.g., 7–30 days), stored securely, and sent only to a dedicated token-refresh endpoint to issue new access tokens seamlessly without forcing the user to log in repeatedly.

9. How does the auth middleware know which user is making the request?authMiddleware extracts the token from the Authorization header, verifies its signature with jwt.verify(), decodes the embedded user payload ({ userId, email }), and attaches it to the Express request object (req.user = decoded) so downstream controllers know exactly which user initiated the request.

10. If you had to implement role-based authorization (e.g., admin vs. regular user), how would you modify your current setup?Database & Entity: Add a role column (e.g., 'user' | 'admin', defaulting to 'user') to the User entity and generate a TypeORM migration.JWT Payload: Include role: user.role when signing the JWT in AuthService.login and expose it on req.user.Role Middleware: Create a requireRole(...allowedRoles) middleware to run after authMiddleware that checks whether allowedRoles.includes(req.user.role) and throws a 403 ForbiddenError if the user lacks permission.


Activity 10: Email Verification & Password Reset

# Authentication API with Email Verification & Password Reset

A Node.js, Express, TypeScript, TypeORM, and MySQL authentication service extended with **email verification**, **password reset**, **Nodemailer** email delivery, **Joi** request validation, structured logging, and centralized error handling.

## Authentication Flow Documentation

### 1. Registration & Email Verification Flow
1. A client sends `POST /api/auth/register` with `name`, `email`, `password`, and optional `age`.
2. The service checks whether `email` already exists in `users` (throwing a `409 Conflict` if found), hashes the password with `bcrypt`, generates a 64-character hex `verificationToken` using `crypto.randomBytes(32).toString('hex')`, sets a **24-hour** `verificationTokenExpiry`, and saves the user with `isVerified = false`.
3. `EmailService.sendVerificationEmail()` sends an HTML email containing a verification link (`${FRONTEND_URL}/verify-email?token=<token>`).
4. Attempting `POST /api/auth/login` before verifying returns `401 Unauthorized` asking the user to verify their email address first.
5. Clicking the link or calling `GET /api/auth/verify-email?token=<token>` validates that the token exists, the account is not already verified, and the token has not expired; it then sets `isVerified = true` and clears `verificationToken` and `verificationTokenExpiry` to `null`.
6. If a user never received or lost their verification email, `POST /api/auth/resend-verification` generates a fresh token and 24-hour expiry for unverified accounts while returning a generic response so account existence isn't leaked.

### 2. Login Flow
1. A client sends `POST /api/auth/login` with `email` and `password`.
2. The service looks up the user (including `password`, `isActive`, and `isVerified`), verifies `isVerified` is true and `isActive` is true, compares the password with `bcrypt.compare()`, and signs a JWT containing `userId` and `email`.

### 3. Forgot Password & Reset Password Flow
1. A client sends `POST /api/auth/forgot-password` with `email`.
2. Regardless of whether the email exists or is verified, the endpoint returns a generic `200 OK` message (`"If an account with that email exists, a reset link has been sent."`) to prevent email enumeration. For existing, verified accounts, it generates a 64-character hex `resetPasswordToken` with a **1-hour** `resetPasswordExpiry` and sends a reset email (`${FRONTEND_URL}/reset-password?token=<token>`).
3. A client sends `POST /api/auth/reset-password` with `token` and `newPassword`.
4. The service validates the token and checks that `resetPasswordExpiry` is in the future, hashes `newPassword` with `bcrypt`, updates `password`, and clears `resetPasswordToken` and `resetPasswordExpiry` to `null`.

## Testing with Ethereal (Viewing Emails)

During development and lab testing, **Ethereal** acts as a fake SMTP catch-all server so real emails are not sent to external inboxes:

1. Generate test SMTP credentials:
   ```bash
   node -e "const nodemailer = require('nodemailer'); nodemailer.createTestAccount().then(account => { console.log('User:', account.user); console.log('Pass:', account.pass); console.log('Host:', account.smtp.host); console.log('Port:', account.smtp.port); })"

2. Add the credentials to your .env file (SMTP_HOST=smtp.ethereal.email, SMTP_PORT=587, SMTP_USER=..., SMTP_PASS=..., EMAIL_FROM=noreply@yourapp.com).

3. Start the server (npm run dev) and trigger registration or forgot-password.

4. Look for the 📧 Ethereal preview URL log entry in your terminal output and open that https://ethereal.email/message/... link in your browser to inspect the rendered HTML email and copy the token query parameter from the action link.

## List of all endpoints with examples

POST /api/auth/registerRegisters a new user and dispatches a verification email.   

JSON{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "Password123",
  "age": 30
}

GET /api/auth/verify-email?token=<token>
Verifies a user's email address using the single-use verification token.   
GET http://localhost:3000/api/auth/verify-email?token=17c7a3d87709245e81319717740b94295ed8a2e581b1bff954525c91b31757c1

POST /api/auth/resend-verification
Resends a verification email if the account exists and is not yet verified.   
JSON{
  "email": "john.doe@example.com"
}

POST /api/auth/login
Authenticates a verified, active user and returns a JWT.  
JSON{
  "email": "john.doe@example.com",
  "password": "Password123"
}

POST /api/auth/forgot-password
Initiates the password reset flow without revealing whether the account exists.   
JSON{
  "email": "john.doe@example.com"
}

POST /api/auth/reset-password
Resets the user's password using a valid, unexpired reset token.
JSON{
  "token": "xyz789...",
  "newPassword": "NewPassword456"
}

GET /api/auth/me
Returns the authenticated user's profile when provided a valid. 
Authorization: Bearer <jwt> header.
GET http://localhost:3000/api/auth/me
Authorization: Bearer <your_jwt_token>

## Security Features Implemented
1. Cryptographically strong random tokens: Tokens are generated with crypto.randomBytes(32).toString('hex') (256 bits of entropy).
Token expiration windows: Verification tokens expire after 24 hours and reset tokens expire after 1 hour.

2. One-time token consumption: Tokens and their expiry timestamps are cleared (null) immediately after successful verification or password reset.

3. Email enumeration protection: Forgot-password and resend-verification responses avoid disclosing whether an email address is registered.

4. Hidden sensitive entity fields: Columns such as password, verificationToken, and resetPasswordToken use select: false on the User entity and are indexed in the database.

5. Password hashing & input validation: Passwords are hashed with bcrypt, and request payloads are validated with Joi schemas.

Reflection Questions
1. Why is email verification important for security? What problems does it prevent?
Email verification proves that the person registering an account actually controls the email address they entered. Without it, attackers can register accounts using someone else's email address—causing unwanted notifications or inbox clutter for the victim, potentially claiming usernames/handles tied to that email, or creating accounts with typos that prevent legitimate users from registering or recovering access later. It also improves deliverability and account integrity because verified users are more likely to receive security alerts and password recovery links.

2. Explain the "always return success" pattern in forgot password. Why is it a security best practice?
The "always return success" pattern returns the same neutral confirmation message (such as "If an account with that email exists, a reset link has been sent.") whether the email exists in the database or not. This prevents attackers from using the forgot-password endpoint as an oracle to discover valid user accounts, which would otherwise give them a targeted list of emails for phishing, credential stuffing, or brute-force attacks.

3. What's the difference between a verification token and a reset token? Why do they have different expiration times?
A verification token confirms ownership of a newly registered (or changed) email address and flips isVerified to true, whereas a reset token authorizes changing an account's password when the user cannot supply their current password. Reset tokens carry higher immediate risk if intercepted because they can directly take over an account, so they use a shorter expiration window (1 hour) to limit the window of exposure. Verification tokens generally allow a longer window (24 hours) because users may not check their email immediately after signing up and the account cannot log in until verified anyway.

4. Why do we use crypto.randomBytes(32) instead of Math.random() or Date.now() for token generation?
Math.random() is a pseudo-random number generator not designed for cryptographic security, meaning its outputs can potentially be predicted or guessed if enough values or internal state clues are observed. Date.now() is deterministic/guessable based on timestamps—if an attacker knows roughly when a request was made, the search space is tiny. crypto.randomBytes(32) draws from the operating system's cryptographically secure pseudo-random number generator (CSPRNG), producing 256 bits of high-entropy randomness that cannot be practically guessed or brute-forced.

5. What happens if a user never verifies their email? How does your system handle this?
In this system, an unverified user has isVerified = false and is blocked from logging in (POST /api/auth/login throws a 401 Unauthorized error telling them to verify their email first). They can request a new verification email via POST /api/auth/resend-verification if their 24-hour token expires, and forgotPassword avoids sending reset links to unverified accounts. In a production system, you might also periodically clean up stale, unverified accounts after a set retention period so unverified emails don't sit around indefinitely.

6. Explain why tokens are cleared after use. What security risk does this prevent?
Clearing verificationToken / verificationTokenExpiry and resetPasswordToken / resetPasswordExpiry to null after use enforces single-use (one-time) tokens. If a token remained stored and valid after being used, anyone who later found it in browser history, logs, forwarded emails, or a compromised client could replay it—for example, reusing a password reset token to change the password again even after the legitimate user already reset it.

7. What is email enumeration and how does your implementation prevent it?
Email enumeration is an attack where an adversary probes endpoints (like registration, login, or password reset) to determine whether specific email addresses have accounts on the system. Our implementation prevents it in forgotPassword and resendVerification by returning the same generic success message whether the user exists or not (and logging internal warnings server-side rather than exposing account status in the HTTP response). (Note: Registration still returns 409 Conflict for an already-registered email, which is common in standard sign-up UX, though some high-security apps handle even that more cautiously.)

8. If you had to implement two-factor authentication (2FA) on top of this system, how would you approach it?
I would add 2FA fields on the User entity (such as isTwoFactorEnabled, a select: false encrypted twoFactorSecret for TOTP authenticator apps, and hashed backup/recovery codes) along with setup and verification endpoints (/2fa/setup, /2fa/verify-setup, /2fa/disable). During POST /api/auth/login, once email, password, isVerified, and isActive pass, I would check isTwoFactorEnabled: if enabled, instead of immediately issuing the final session JWT, I would issue a short-lived, limited-scope temporary challenge token and require POST /api/auth/login/2fa with a valid 6-digit TOTP code (or recovery code) before minting the full access/refresh token pair.

9. What are the trade-offs between using Ethereal (test service) vs. SendGrid (production service)?
Ethereal is free, requires no account setup, and captures messages in a sandbox preview URL rather than delivering them to real users—making it ideal for local development and automated/manual testing without spamming real inboxes or burning production quota. However, Ethereal is not meant for real user delivery or high reliability. SendGrid is built for production deliverability, analytics, reputation management, and scale, with dedicated infrastructure and webhooks—but it introduces paid tiers/quotas, API key/domain setup (SPF/DKIM/DMARC), and real external side effects during testing if not separated by environment.

10. How would you handle a situation where a user's email provider marks your emails as spam?
First, I would verify DNS authentication records for the sending domain—specifically SPF, DKIM, and DMARC—and ensure the EMAIL_FROM address matches a domain you control rather than a generic/unauthenticated sender. Next, I would audit the email content and sending practices: keep HTML clean and consistent with plain-text fallbacks, avoid spammy phrasing or excessive links, include clear branding/unsubscribe or support info where appropriate, and throttle/rate-limit verification and reset emails so users aren't flooded. Finally, I would monitor bounce/spam reports via the email provider's dashboard or webhooks and give users a clear "check your spam/junk folder" note alongside the resend-verification option.