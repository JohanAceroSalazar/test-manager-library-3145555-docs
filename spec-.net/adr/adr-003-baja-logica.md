# ADR-003 · Books are soft-deleted, not removed

- **Status:** accepted
- **Context:** a loan item points to a book. If the book is physically deleted, the history and report are broken.
- **Decision:** the book has `deleted_at`. Soft deletion populates it; the catalog filters by `deleted_at IS NULL` in one place (global filter). A book with open loans is **not** soft-deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a **partial** index on active books, so a soft-deleted ISBN can be registered again.
- **Rejected alternatives:** physical deletion with cascade (destroys history); setting `active = false` (does not say when).
- **Consequences:** `loan_item.book_id` has a `RESTRICT` FK as a **final barrier**: a manual `DELETE` or a future code change fails loudly instead of corrupting history. The cover binary is the only data that is physically deleted.
- **How to verify:** querying a soft-deleted book returns 404; a `DELETE` through `psql` on a book with items fails because of the FK.
