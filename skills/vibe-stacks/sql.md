# PostgreSQL / MySQL

**Applies when:** the project uses PostgreSQL or MySQL, through an ORM/query builder (Prisma, TypeORM, Drizzle, SQLAlchemy, GORM, sqlc…) or a raw driver. Check which database and version — some features differ (noted below).

## Dev

- Every schema change is a migration in the project's tool (Prisma Migrate, TypeORM/Drizzle migrations, Alembic, golang-migrate, goose…). Never edit an already-applied migration.
- Migrations must be safe on a live table:
  - Add a column as nullable or with a default; backfill in batches; add `NOT NULL` afterwards.
  - Rename/drop in steps: add new → write both → migrate reads → drop old, across deploys.
  - Postgres: `CREATE INDEX CONCURRENTLY` on large tables (outside a transaction). MySQL: check the operation is `ALGORITHM=INPLACE`/`INSTANT` or plan a tool like gh-ost.
- Index every foreign key column and every column used in frequent `WHERE`/`ORDER BY`/`JOIN`. Composite index column order follows the query (equality columns first, then range/sort).
- Constraints in the database, not only in code: `NOT NULL`, `UNIQUE`, `FOREIGN KEY`, `CHECK`. Uniqueness enforced only in app code races.
- Multi-step writes in one transaction. Concurrent updates to the same row: `SELECT … FOR UPDATE`, optimistic version column, or an atomic `UPDATE … SET x = x + 1`.
- Upsert with the database's native form: Postgres `ON CONFLICT`, MySQL `ON DUPLICATE KEY UPDATE`.
- Money as `NUMERIC`/`DECIMAL`, never float. Timestamps as `timestamptz` (Postgres) / `DATETIME` stored in UTC (MySQL).
- Paginate list queries; prefer keyset (`WHERE id > ? ORDER BY id LIMIT ?`) over large `OFFSET` on big tables.

## Review checks

- **Critical:** query built by string concatenation with user input (including `ORDER BY`/column names — whitelist them); migration that drops/renames a column still read by running code; destructive migration with no backup/rollback plan.
- Migration locks a large table (non-concurrent index, column type change, `NOT NULL` without default on Postgres < 11 / MySQL older versions).
- N+1 queries: loop of single-row queries — use a join, `IN (...)`, or the ORM's eager loading.
- `SELECT *` on wide tables in hot paths; unbounded queries without `LIMIT`.
- New `WHERE`/`JOIN` column with no index — ask for `EXPLAIN` on large tables.
- Read-modify-write without a transaction or lock → lost updates.
- Unique rule enforced only by "check then insert" in code.
- Long transaction holding locks while calling an external service.

## Testing

- Integration tests run against the **same database engine and major version** as production (Testcontainers / docker-compose) — not SQLite as a stand-in.
- Apply the real migrations in test setup; test the migration itself on a copy with realistic data when it's risky.
- Isolation: transaction rollback per test, or truncate between tests. Never share mutable fixtures across parallel tests.
- Cover constraint violations (duplicate unique, missing FK) and concurrent updates for code that relies on them.
