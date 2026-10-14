# ADR-001 · The schema is owned by application migrations

- **Status:** accepted
- **Context:** the `infra` repository starts a database engine. Someone could place DDL in a container startup script and someone else in application migrations; with two owners, no one knows which schema runs in production.
- **Decision:** **all DDL lives in application migrations** (tables, indexes, constraints, and category seed data). `infra` only creates the empty engine, user, and volume.
- **Rejected alternatives:** SQL scripts in `infra/db/init/` (a second source of truth); ORM in “create tables on startup” mode (leaves neither history nor rollback).
- **Consequences:** a Postgres extension or new index is created in the **same** migration that uses it. A schema changed manually in the engine is a defect, not a variant.
- **How to verify:** `infra` does not contain `CREATE TABLE`; the queries in §10 of `data-model.md` match the migrations.
