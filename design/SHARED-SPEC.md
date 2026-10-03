# ExpenseTracker — Shared Specification

The contract both implementations must satisfy, so that any difference between
the Python and C++ code is a consequence of the language rather than of two
people solving different problems.

## 1. Record

| Field | Type (conceptual) | Notes |
|---|---|---|
| `id` | integer | Assigned by the store, from 1, never reused |
| `date` | calendar date | Supplied as `YYYY-MM-DD` |
| `amount` | non-negative decimal | Stored as a double; rendered to two decimals |
| `category` | string | Trimmed and lower-cased on entry |
| `description` | string | Free text, searched case-insensitively |

Validation: a malformed date, a negative amount or an empty category is an
error. Python raises `ValueError`; C++ throws `std::invalid_argument`.

## 2. Store operations

| Operation | Semantics |
|---|---|
| `add(date, amount, category, description)` | Append and return the new id |
| `remove(id)` | Delete; return whether it existed |
| `all()` | Every record, oldest first, ties broken by id |
| `by_category(name)` | Case-insensitive exact category match, via the index |
| `in_range(from, to)` | Inclusive on both ends; either bound may be absent |
| `search(keyword)` | Case-insensitive substring of the description |
| `summary()` | Per-category count, total and percentage share, ordered by total descending, plus the overall total |

## 3. Seeded dataset

Both implementations ship the same ten records so that output can be compared
line by line.

| date | amount | category | description |
|---|---|---|---|
| 2026-09-01 | 82.40 | groceries | weekly shop |
| 2026-09-03 | 15.00 | transport | metro card top up |
| 2026-09-05 | 120.00 | utilities | electricity bill |
| 2026-09-08 | 46.75 | dining | dinner with study group |
| 2026-09-12 | 91.10 | groceries | weekly shop |
| 2026-09-15 | 9.99 | entertainment | streaming subscription |
| 2026-09-19 | 63.20 | groceries | weekly shop and household items |
| 2026-09-22 | 28.50 | transport | rideshare to campus |
| 2026-09-27 | 55.00 | dining | lunch with classmates |
| 2026-10-02 | 34.25 | utilities | water bill |

Expected results: overall **$546.19** across 10 records; groceries 43.3%,
utilities 28.2%, dining 18.6%, transport 8.0%, entertainment 1.8%. After
deleting record 3, 9 records totalling **$426.19**.

## 4. Demo sequence

1. List all expenses.
2. Filter by category `groceries` (3 records).
3. Filter by date range 2026-09-05 to 2026-09-20 (4 records).
4. Search for the keyword `weekly` (3 records).
5. Print the summary.
6. Delete record 3 and print the summary again.

## 5. Benchmark

`bench N` inserts `N` generated expenses cycling through five categories and 28
days, then runs the category filter, the date-range filter, a keyword search and
the summary. It reports insert time, insert rate and total query time. Both
implementations must report the same result counts for the same `N`.

## 6. Non-goals

Out of scope: persistence to disk, multi-user access, currency conversion and a
graphical interface. The assignment permits a text-based application, and
keeping the surface small keeps the comparison on language design.

## 7. Acceptance checklist

- [ ] `demo` output matches between the two languages apart from the title line
- [ ] Category, range and keyword matching are all case-insensitive and inclusive
- [ ] Summary shares sum to 100%
- [ ] Deleting a record updates both the totals and the category index
- [ ] Unit tests pass in both languages
- [ ] The C++ build is clean under `-Wall -Wextra` and reports no leaks under AddressSanitizer
- [ ] `bench` reports identical result counts in both
