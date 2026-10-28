# FastAPI (Python)

**Applies when:** `fastapi` in pyproject.toml / requirements. Check whether the project uses Pydantic v1 or v2 and sync or async DB access, and follow it.

## Dev

- Request and response bodies are Pydantic models. Separate input (`UserCreate`), output (`UserRead`) and DB models — set `response_model` so internal fields never leak.
- Shared concerns (DB session, current user, permissions, pagination) via `Depends`, not repeated in each route.
- Routes in `APIRouter`s per feature; business logic in a service layer, not in the route function.
- `async def` routes must only call async I/O. Sync DB drivers or blocking libs → use `def` routes (FastAPI runs them in a threadpool) or `run_in_threadpool`.
- Raise `HTTPException` (or domain exceptions with registered handlers) — never return error dicts with 200.
- Settings via `pydantic-settings` `BaseSettings`; no `os.environ[...]` scattered in code.
- DB: one session per request via a dependency with `yield`, commit/rollback in one place. Migrations with Alembic — every model change ships with a migration.
- Type hints everywhere; the project's checker (mypy/pyright) and linter (ruff) must pass.

## Review checks

- **Critical:** route missing the auth dependency where siblings have it; missing ownership check on `/{id}` routes; SQL built with f-strings; `response_model` missing on a route returning a DB object with secret fields.
- Blocking call (`requests`, sync DB driver, `time.sleep`, heavy CPU) inside `async def` → stalls the whole event loop.
- Mutable default arguments; Pydantic models with `dict`/`list` defaults not using `Field(default_factory=...)`.
- Session/connection not closed on error paths (dependency without `try/finally` or `yield` cleanup).
- N+1 with SQLAlchemy lazy loading — use `selectinload`/`joinedload`.
- Model changed without an Alembic migration, or migration not reversible.
- Background work started with `BackgroundTasks` that must survive a crash → needs a real queue (Celery, RQ, arq).

## Testing

- Pytest. API tests via `TestClient` (sync) or `httpx.AsyncClient` with `ASGITransport` + `pytest-asyncio`/`anyio` (async).
- Swap dependencies with `app.dependency_overrides` (current user, external clients) — clear them after each test.
- Real test DB (Docker/Testcontainers), transaction rolled back per test via a fixture. Run Alembic migrations in the test setup so tests catch migration bugs.
- Cover per route: 422 for invalid input, 401/403, 404, happy path, and the `response_model` not leaking fields.
- Coverage with `pytest-cov` (`--cov --cov-branch`).
