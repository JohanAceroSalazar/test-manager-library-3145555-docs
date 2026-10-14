# Tasks — Manager Library (Python Reference)

Each task is complete when its **evidence** exists and can be run (Article XII). Initial status for all tasks: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` returns `{"status":"ok"}`; the container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes, and the five seeded categories | T-01 | `data-model.md` §10 queries return what is documented; no column has a `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, soft deletion) | T-03 | Tests: never < 0, never > total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, calculated status, 14 days, max. 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (username normalization, valid role) | T-03 | `"Ana "` becomes `ana`; `admin` role through API rejected |
| T-07 | Authentication ports and use cases (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | Book ports and use cases (create, search, edit, soft deletion, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | **Lend** use case (`LendBooksService`) with fakes | T-05, T-08 | Complete HU-06: limits, overdue items, no copies |
| T-10 | **Return** and **overdue** use cases | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | **Most-borrowed report** use case and read port | T-09 | HU-10; previous title remains after rename (ADR-004) |
| T-12 | Persistence adapter: book, member, and user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository + version witness | T-12, T-09 | Two loans of the last copy: one 201, the other 409 (RNF-2) |
| T-14 | Security adapter: password hash and JWT token | T-07 | The password never appears in logs or responses (RNF-3) |
| T-15 | Cover adapter (disk) | T-08 | Uploading a 2 MB PNG succeeds; 3 MB → 422; no cover → null `cover_key` |
| T-16 | HTTP adapter: [`api-contract.md`](api-contract.md) routes, uniform error body, role authorization | T-10, T-11, T-14, T-15 | Contract test covers every route and every §2 code |
| T-17 | Composition and startup (initial admin from environment) | T-16 | The system starts with an empty database and can log in as admin |
| T-18 | Move **domain-only** invariants to the database: `ck_book_total_positive`, `ck_user_role` | T-12 | A violating `INSERT` through `psql` fails; `data-model.md` §4 is updated |
| T-19 | Architecture tests (Articles I, II, and III) | T-17 | NetArchTest passes and fails if a prohibited import is introduced |
| T-20 | `infra`: `docker compose up` starts empty engine, API, and frontend | T-17 | One command; the database starts without tables until migrations run |
| T-21 | `tool`: demonstration seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue items) | T-17 | After running it, `/loans/overdue` returns 3 items and the report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue items, report | T-16 | End-to-end manual walkthrough at the presentation event |
| T-23 | `page`: public static presentation site | — | It opens without the API; correct on mobile and desktop |

## Assessment definition of done

Anyone who reaches T-17 with evidence has completed the **core**. T-18 through T-23 distinguish a good submission from an outstanding one. The assessment evaluates
SDD performance, not task quantity: one task with evidence is worth more than three half-finished tasks.
