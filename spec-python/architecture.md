# Architecture — Manager Library (Python Reference)

**Hexagonal** architecture (ports and adapters). The dependency rules are Articles I through III of [`constitution.md`](constitution.md).
This document describes the Python reference implementation; the mandatory implementation stack is PHP/Laravel, React, PostgreSQL 16, and Docker Compose.

## 1. Layers and dependencies

```mermaid
flowchart LR
    subgraph Inbound
      HTTP["adapters/inbound/http<br/>(FastAPI)"]
    end
    subgraph Core
      APP["application<br/>use cases + ports"]
      DOM["domain<br/>aggregates and value objects"]
    end
    subgraph Outbound
      DB["adapters/outbound/persistence<br/>(SQLAlchemy 2 + Alembic)"]
      STO["adapters/outbound/storage<br/>covers"]
      SEC["adapters/outbound/security<br/>hashing and token"]
    end
    HTTP --> APP --> DOM
    DB -. implements ports for .-> APP
    STO -. implements .-> APP
    SEC -. implements .-> APP
```

Dependency arrows point **inward**. `bootstrap` is the only part that sees both halves.

## 2. Directory structure

```text
src/library/
├── domain/              # Book, Loan, LoanItem, Member, User; Isbn, Quantity, DateRange
├── application/
│   ├── ports/outbound/  # BookRepository, LoanRepository, MemberRepository, UserRepository,
│   │                    # ReportReader, Clock, PasswordHasher, TokenIssuer, CoverStorage
│   └── services/        # LendBooksService, ReturnItemService, CreateBookService, SearchBooksService,
│                        # OverdueService, MostBorrowedReportService, LoginService, CreateLibrarianService
├── adapters/
│   ├── inbound/http/    # routers/controllers, request schemas, error handling
│   └── outbound/        # persistence/, storage/, security/
└── bootstrap/composition/   # src/library/bootstrap/composition/port_bindings.py — single composition point
```

## 3. Aggregates

| Root | Contains | Invariants it protects |
|---|---|---|
| `Book` | `Isbn`, total and available copies, soft deletion | available copies between 0 and total; total ≥ 1; non-empty title |
| `Loan` | collection of `LoanItem` | 1 to 3 items; no repeated book; 14-day due date; immutable except for return |
| `Member` | document number | required and unique document number |
| `User` | role, hash | normalized username; role in (`admin`, `librarian`) |

`Loan.AddItem` calls `Book.Withdraw` before adding the item: withdrawing the copy and creating the item are **a single operation**.
`Loan.ReturnItem` calls `Book.Restock` only once per item.

## 4. Flow: register a loan

```mermaid
sequenceDiagram
    participant C as Client
    participant H as HTTP
    participant S as LendBooksService
    participant M as MemberRepository
    participant B as BookRepository
    participant L as LoanRepository
    C->>H: POST /api/v1/loans
    H->>S: Lend(memberId, bookIds)
    S->>M: get member + unreturned items
    S->>S: overdue items? exceeds 3?
    S->>B: get books by batch (Q3)
    S->>S: Loan.AddItem → Book.Withdraw (for each book)
    S->>L: save loan + books (one transaction)
    L-->>S: ok or version conflict
    S-->>H: loan or BOOK_NOT_AVAILABLE
    H-->>C: 201 or 409
```

## 5. Errors

The domain and application raise their own errors (`BookNotAvailable`, `MemberHasOverdue`, `MemberLimitExceeded`, …). **Only** the HTTP adapter
translates them into the uniform body and status code in [`api-contract.md`](api-contract.md). A framework error never reaches the client.

## 6. What is not included

No events, queues, cache, microservices, or multi-tenancy. They are added only if a requirement calls for them.
