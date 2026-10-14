# Constitution — Manager Library (.NET Reference)

Non-negotiable project principles. They are neither style nor preference: every article includes **how it is verified**, because a principle
that nobody can verify is an intention. A change that violates an article is rejected without discussing the rest.

---

## Article I — The domain depends on nothing

`src/Library.Domain/` imports no other project module or third-party library: only the base library (`record`, `DateTimeOffset`); no EF Core, ASP.NET, or FluentValidation.

**Why:** this is the only way to test invariants without a database, server, or container, and to change ORM without touching rules.
**Verification:** NetArchTest prohibits the domain from depending on `application`, `adapters`, or `bootstrap`.

## Article II — The core declares its ports; the outside world implements them

The application layer knows only the domain. Everything it needs from outside (repositories, clock, password hashing, cover storage,
token generation) is expressed through a **port it defines itself**. No framework or ORM type appears in a port signature.

**Verification:** NetArchTest prevents `application` from importing `adapters`. Application tests run with fakes and without referencing `adapters`.

## Article III — A single composition point

A single module (`src/Library.Bootstrap/Composition/PortBindings.cs`) is where a port meets its adapter. No infrastructure is instantiated outside it.
**Verification:** no instantiation of an `adapters` class exists outside that module; this is checked with an architecture test.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (disk → S3) does not touch domain or application | The new adapter adds only one line in composition |
| **LSP** | Test fakes replace the real adapter without changing expectations | The same test passes against the fake and the real adapter |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementer leaves methods unused |
| **DIP** | The port is defined by its consumer | Ports live in the application package |

## Article V — The schema belongs to migrations

The `infra` repository starts an **empty** engine. All DDL (tables, indexes, constraints, extensions, and seed data) lives in
application migrations (EF Core migrations). See [ADR-001](adr/adr-001-schema-ownership.md).
**Verification:** `infra` contains no `CREATE TABLE`.

## Article VI — The database enforces invariants it can enforce

A rule that exists only in code protects the application, not the data. Every invariant expressible in the engine (`CHECK`, `UNIQUE`, FK)
is declared there. Anything not enforced is marked **pending** in [`data-model.md`](data-model.md); it is not promised.
**Verification:** the queries in `data-model.md` §10 match what the document declares.

## Article VII — Derived values are calculated, not stored

Loan status (`open`, `overdue`, `closed`) and days late are calculated. There is no column for them.
**Verification:** `data-model.md` lists neither a `status` nor a `days_late` column.

## Article VIII — Snapshotted values do not follow the catalog

A loan item keeps the title, ISBN, and category from the time of the loan. No report reads them from the live catalog.
**Verification:** a test renames a book after lending it and verifies that the report retains the previous title ([ADR-004](adr/adr-004-aggregated-and-snapshot-reporting.md)).

## Article IX — No secret enters the repository or a log

No passwords, precomputed hashes, or token keys. Every secret comes from the environment. The real `.env` is in `.gitignore`; only `.env.example` is versioned.
**Verification:** `gitleaks` or a pattern `grep` in CI; no HTTP response includes `password` or `password_hash`.

## Article X — The engine wins

If the data document and the database disagree, the document is wrong and must be corrected.

## Article XI — Language

Code, columns, and routes are in English. Documentation is in English; user-facing messages are in Spanish.

## Article XII — Nothing is assumed without evidence

A [`tasks.md`](tasks.md) task is complete when its **evidence** exists and can be run, not when it is merely “already programmed.”
