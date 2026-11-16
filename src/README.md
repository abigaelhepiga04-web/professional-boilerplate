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