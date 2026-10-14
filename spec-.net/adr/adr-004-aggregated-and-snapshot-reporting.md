# ADR-004 · The report is calculated in the engine and groups by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a range. If it read the title or category from the live catalog, renaming a book or category would change a report for an already closed period.
- **Decision:** (1) the report is **not persisted**: it is calculated with a single aggregate query through a read port. (2) It groups by `book_id, book_title, category_name` **as frozen in the item**. (3) `loan_item.category_name` deliberately has **no FK**. (4) If a book changed category within the range, the report shows **two rows**: it does not choose a winner.
- **Rejected alternatives:** joining the live catalog (the report is no longer stable); taking “the most recent value in the range” (a new loan would change what was already read); materializing a report table (a second source of truth).
- **Consequences:** CA-10.1 says “one row per frozen book and category name,” not “one row per book.” The report does not include members or librarians (privacy).
- **How to verify:** lend a book, rename it, request the report for the same range: the report title is the old one.
