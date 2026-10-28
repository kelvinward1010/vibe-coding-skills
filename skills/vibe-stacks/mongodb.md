# MongoDB

**Applies when:** `mongodb`, `mongoose`, `@nestjs/mongoose`, `motor`, `beanie`, `pymongo`, or `go.mongodb.org/mongo-driver` is a dependency.

## Dev

- Model for the queries you run: embed data read together and bounded in size; reference (store ids) for data that grows without limit or is shared. Never let an array grow unbounded inside one document (16 MB limit, slow updates).
- Schema validation in code (Mongoose schema, Pydantic/Beanie model, Go struct tags) **and** a `$jsonSchema` validator on important collections.
- Indexes declared in code/migrations, matching each query: equality fields first, then sort, then range (ESR rule). `unique: true` for natural keys — not a "find then insert" check.
- Update with operators (`$set`, `$inc`, `$push`, `$addToSet`) instead of read-modify-write of the whole document. Use `findOneAndUpdate` with a filter for conditional updates.
- Multi-document writes that must succeed together → transaction (needs a replica set) or redesign so one document holds the invariant.
- Project only the fields you need. Paginate with a range on an indexed field (`_id > last`) instead of large `skip`.
- Mongoose: `.lean()` for read-only queries; avoid `populate` chains in hot paths.
- Schema changes on existing data need a migration script (migrate-mongo or the project's tool) — old documents don't change themselves.

## Review checks

- **Critical:** query object built from raw request body (`find(req.body)`, `{ $where: … }`) → NoSQL operator injection — validate types and strip `$`-prefixed keys; missing ownership filter (`{ _id, userId }`) on user-scoped documents.
- Query on a field with no supporting index on a large collection (ask for `explain("executionStats")`: `COLLSCAN` is a red flag).
- Read-modify-write without atomic operators or version check → lost updates.
- Unbounded arrays or documents that grow per event/log entry.
- `ObjectId` compared as string vs object inconsistently; invalid id strings not handled (cast error → 500 instead of 400/404).
- Field renamed/added without handling old documents.
- `$lookup` used as a join on hot paths — consider embedding or denormalizing.

## Testing

- Integration tests against a real MongoDB (Testcontainers, or `mongodb-memory-server` for Node) — use a replica set if the code uses transactions.
- Create the same indexes in tests as in production so unique constraints are exercised.
- Clean collections between tests; never point tests at a shared dev database.
- Cover: duplicate key error handling, invalid ObjectId input, concurrent updates on the same document, migration of old-shaped documents.
