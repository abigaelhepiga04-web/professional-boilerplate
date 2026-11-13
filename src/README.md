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
3. Install all indepedencies
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