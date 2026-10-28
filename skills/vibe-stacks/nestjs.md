# NestJS (Node.js)

**Applies when:** `@nestjs/core` in package.json. For plain Node/Express services, the Review checks on validation, async errors and secrets still apply.

## Dev

- One feature = one module (`*.module.ts`, controller, service, DTOs). Controllers stay thin: parse input, call a service, return. Business logic lives in services.
- Every request body/query/param has a DTO with `class-validator` decorators (or the project's Zod pipe). Global `ValidationPipe` with `whitelist: true` and `forbidNonWhitelisted: true` — check it's on before relying on it.
- Inject dependencies through constructors; no `new SomeService()` and no module-level singletons holding state.
- Auth via Guards (`@UseGuards`), cross-cutting concerns via Interceptors/Pipes/Filters — not copy-pasted checks in each handler.
- Throw Nest HTTP exceptions (`NotFoundException`, …) or domain errors mapped by an exception filter. Never return `{ error }` with status 200.
- Config through `@nestjs/config` (validated schema), never `process.env` scattered through services.
- Multi-step writes in one transaction using the project's ORM (Prisma `$transaction`, TypeORM `QueryRunner`/`DataSource.transaction`).

## Review checks

- **Critical:** new endpoint without a guard where the rest of the controller/app requires auth; missing ownership check (user A can read/update user B's resource by id); raw SQL built with string concatenation; entity returned directly with password hash or internal fields.
- DTO missing or not validated → unchecked input reaches the service.
- Response shape: return a response DTO / serializer (`ClassSerializerInterceptor`, `@Exclude`) instead of the ORM entity.
- `async` call not awaited (floating promise) → errors swallowed, request ends early.
- Circular module imports papered over with `forwardRef` — usually a design smell.
- N+1: loop of repository calls; use relations/`include`/`IN` query.
- Blocking work (CPU-heavy, sync fs/crypto) on the request path → move to a queue (BullMQ) or worker.

## Testing

- Unit: Jest (or Vitest) with `Test.createTestingModule`, overriding providers with fakes. Test services, not decorators.
- Integration/e2e: boot the real app module with `supertest` against a real test DB (Docker/Testcontainers); reset data between tests. Cover: validation 400s, 401/403 for each guarded route, 404 for missing ids, the happy path.
- Don't mock the ORM in integration tests — that's where query bugs hide.
