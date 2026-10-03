# ExpenseTracker — Deliverable 1: Planning and Design

**MSCS-632-M30 Advanced Programming Language · Group Project (Option 1: Expense Tracker Application)**
**Languages:** Python and C++ · **Instructor:** Jay Thom · **University of the Cumberlands**

| Team member | Primary role |
|---|---|
| Rahul Solanki | Python implementation; benchmarking and profiling |
| Krinal Soni | C++ implementation; testing and memory checking |
| Bijay Raj KC | Shared specification, documentation, comparison report |

This repository holds the Friday planning deliverable only. Implementation lives
in the Day 2 and Day 3 repositories listed at the bottom.

---

## 1. What we are building

**ExpenseTracker** is a text-based application for recording, filtering and
summarising personal expenses, implemented twice with identical behaviour: once
in Python and once in C++.

### Core requirements

| # | Requirement | How we satisfy it |
|---|---|---|
| R1 | Data storage for expenses with date, amount, category and description | An expense record carries `id`, `date`, `amount`, `category`, `description`; the store assigns the id |
| R2 | Filter and search by criteria (date range, category) | `by_category(name)`, `in_range(from, to)` and `search(keyword)`, all case-insensitive and inclusive |
| R3 | Summary showing total by category and overall | `summary()` returns per-category count, total and percentage share, plus the overall total |

### Language-specific requirements

| Language | Required focus | Our approach |
|---|---|---|
| Python | Dictionaries for storage, dynamic typing, `datetime` | Records are dicts in a dict keyed by id, plus a `defaultdict` category index; `amount` accepts int, float or numeric string; dates parsed and compared with `datetime` |
| C++ | Memory management, structs/classes, STL containers | `struct Expense` with declared fields; `std::vector` owns the records and `std::unordered_map` indexes them; `std::move`, `reserve`, RAII, and no `new`/`delete` anywhere |

---

## 2. Architecture

Both implementations use the same two-layer shape so that differences in the
code are attributable to the languages rather than to differing designs.

```
  CLI driver  ──►  ExpenseStore  ──►  records container
  (demo / bench / REPL)   │           + category index
                          └──►  queries: category, date range, keyword, summary
```

| Component | Responsibility | Python | C++ |
|---|---|---|---|
| Record | Build, validate, format one expense | `expense.py` | `expense.hpp` |
| Store | Hold records; run filters and the summary | `store.py` | `store.hpp` / `store.cpp` |
| Driver | CLI modes | `main.py` | `main.cpp` |
| Tests | Record helpers and store queries | `test_expense.py` | `tests.cpp` |

### CLI modes (identical in both)

| Mode | Purpose |
|---|---|
| `demo` | Seeded dataset, then category filter, date range, keyword search and the summary |
| `bench N` | Insert `N` expenses and run every query; reports insert and query timings |
| *(no arguments)* | Interactive prompt: `/add`, `/list`, `/cat`, `/range`, `/search`, `/summary`, `/del`, `/quit` |

---

## 3. Anticipated language differences

Recorded before implementation. Day 3 revisits each row and reports where we
were right and where we were wrong.

| Area | Expectation |
|---|---|
| **Type systems** | Python accepts an amount as int, float or string and fails only when the bad value is used; C++ fixes the parameter type at compile time, so the same mistake cannot be written |
| **Data structures** | Python `dict` and C++ `std::unordered_map` are both hash tables, so the algorithmic shape should match and the constant factors should not |
| **Memory management** | Python reference-counts and collects cycles; C++ containers own their buffers and free them in destructors. We expect C++ to use less memory and to need no explicit `delete` |
| **Dates** | Python's `datetime` parses, compares and formats out of the box; C++20 `<chrono>` gives a calendar type but we expect to write parsing and formatting ourselves |
| **Error handling** | Both raise exceptions, but Python surfaces type problems at run time while C++ surfaces most of them at compile time |
| **Performance** | C++ faster on both insert and query, probably by a single-digit multiple rather than orders of magnitude, since both use hash-indexed containers |

### Risks identified up front

1. **Index invalidation.** If the C++ store indexes records by vector position,
   deleting a record shifts every later element and invalidates the index.
   Mitigation: rebuild the index on delete, and document the cost against
   Python's id-keyed approach.
2. **Float money.** Binary floating point cannot represent every decimal amount
   exactly. Mitigation: both implementations use `double` and format to two
   decimals, so they agree with each other; noted as a limitation.
3. **Toolchain parity.** Timings are comparable only on one machine. Mitigation:
   a single WSL2 Ubuntu 24.04 environment for every build and measurement.

---

## 4. Timeline

| Day | Milestone | Owner |
|---|---|---|
| **Fri** | Shared specification, architecture, query semantics, repositories | Bijay Raj KC (lead), all |
| **Sat AM** | Record and store types compiling in both languages | Rahul (Python), Krinal (C++) |
| **Sat PM** | Seeded dataset, listing and category filter working end to end | Rahul, Krinal |
| **Sat PM** | Core-functionality report with screenshots | Bijay |
| **Sun AM** | Date-range filter, keyword search, summary with shares, delete, REPL | Rahul, Krinal |
| **Sun AM** | Unit tests in both; AddressSanitizer run on the C++ build | Krinal |
| **Sun PM** | Benchmark runs and comparison report | Rahul, Bijay |
| **Sun PM** | Presentation slides and rehearsal | All |

---

## 5. Repositories

| Deliverable | Repository | Visibility |
|---|---|---|
| Day 1 — Planning and design | `MSCS-632-M30-Group-D1` | Public |
| Day 2 — Core functionality | `MSCS-632-M30-Group-D2` | Private |
| Day 3 — Final, comparison, presentation | `MSCS-632-M30-Group-D3` | Private |
