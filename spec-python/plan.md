# Technical Plan — Manager Library (Python Reference)

How to build what [`spec.md`](spec.md) and [`constitution.md`](constitution.md) define. The schema is **not** described here: it lives in
[`data-model.md`](data-model.md). The system's shape is in [`architecture.md`](architecture.md).
This is a Python reference plan; the mandatory implementation stack is PHP/Laravel, React, PostgreSQL 16, and Docker Compose.

## 1. Technical decisions

| # | Decision | Reference choice (Python) | Why |
|---|---|---|---|
| D-01 | Language and runtime | Python 3.12 | Reference specification stack |
| D-02 | HTTP framework | FastAPI | Contract in [`api-contract.md`](api-contract.md) |
| D-03 | Persistence | SQLAlchemy 2 + Alembic on PostgreSQL 16 | The schema belongs to migrations (ADR-001) |
| D-04 | Concurrency | Version witness per book + range `CHECK` ([ADR-002](adr/adr-002-optimistic-concurrency.md)) | Last copy (RNF-2) |
| D-05 | Dates | UTC instants (`timestamptz`); the clock is a port (`Clock`) | Test overdue items without waiting 14 days |
| D-06 | Reporting | A single aggregate query through a read port ([ADR-004](adr/adr-004-aggregated-and-snapshot-reporting.md)) | Stable and inexpensive |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: live within their owner's row, with no table | They have no identity |
| D-08 | Covers | Binary in external storage (disk locally, S3 in production); the database stores only `cover_key` | Opaque key, never a path |
| D-09 | Passwords | Argon2id behind the `PasswordHasher` port; the domain never sees the plaintext password | Article IX |
| D-10 | Startup | The application creates the initial admin from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot produce the hash |
| D-11 | Authentication | JWT HS256, 8 hours, key in `JWT_SECRET` | Simple and stateless |
| D-12 | Testing | pytest, with fakes for ports and a real database only in integration tests | Testing pyramid |

## 2. Environment configuration

| Variable | Required | Example |
|---|---|---|
| `DATABASE_URL` | yes | `postgresql://library:library@db:5432/library` |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | defined in `.env`, never in the repository |
| `COVERS_DIR` | no | `./var/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

## 3. Testing strategy

| Level | What it tests | Against what |
|---|---|---|
| Domain | Invariants: maximum 3, 14-day due date, available copies in range | Nothing: pure objects |
| Application | Use cases: lend, return, report | Port fakes |
| Integration | Repositories and report query | Real PostgreSQL |
| Contract | Uniform error body and status codes | Running API |
| Architecture | Articles I, II, and III | an `import-linter` contract and a test in `tests/architecture/` |
| Concurrency | Two loans of the last copy | Real PostgreSQL |

## 4. Build order

Skeleton and health → initial seeded migration → domain → use cases with fakes → adapters → API → contract and concurrency tests.
The details, including evidence, are in [`tasks.md`](tasks.md).
