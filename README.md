# ConcurrentChat — Deliverable 1: Planning and Design

**MSCS-632-M30 Advanced Programming Language · Group Project (Option 2: Simple Chat Application)**
**Languages:** Rust and Go · **Instructor:** Jay Thom · **University of the Cumberlands**

| Team member | Primary role |
|---|---|
| Rahul Solanki | Rust implementation lead; benchmarking and profiling |
| Krinal Soni | Go implementation lead; testing and race detection |
| Bijay Raj KC | Shared specification, documentation, comparison report |

This repository holds the Friday planning deliverable only. Implementation
lives in the Day 2 and Day 3 repositories listed at the bottom.

---

## 1. What we are building

**ConcurrentChat** is a text-based chat application, implemented twice with
identical behaviour: once in Rust and once in Go. Multiple simulated users send
messages concurrently to a central hub that owns the message history and
answers queries against it.

### Core requirements (from the assignment)

| # | Requirement | How we satisfy it |
|---|---|---|
| R1 | Simulate multiple users sending messages to each other | Each user is an independent task (Rust) or goroutine (Go); a `demo` mode runs a scripted concurrent session |
| R2 | Store message history with timestamp and user ID | Every `Message` carries `id`, `timestamp`, `user_id`, `username`, and a kind-specific payload |
| R3 | Message filtering and search by user or keyword | Hub queries: `History(limit)`, `ByUser(name)`, `Search(keyword)`, plus a `Stats` summary |

### Language-specific requirements

| Language | Required focus | Our approach |
|---|---|---|
| Rust | Memory safety; enums and structs for message types; async concurrency | `enum MessageKind` with per-variant payloads; `struct Message`; Tokio tasks with `mpsc` + `oneshot` channels; history owned by one task so no lock is needed |
| Go | Goroutines and channels for message passing; performance | Hub goroutine owning the history; `select` loop over a single command channel; `sync.WaitGroup` for lifecycle; throughput benchmark mode |

---

## 2. Architecture

Both implementations use the same shape, so that differences in the code are
attributable to the languages rather than to differing designs.

```
   user task/goroutine  ─┐
   user task/goroutine  ─┼─► [ command channel ] ─► HUB ─► owns Vec<Message> / []Message
   user task/goroutine  ─┘                           │
                                                     └─► reply channel ─► caller
```

**Single-owner principle.** The history is owned by the hub alone. Nothing else
holds a reference to it, so neither implementation needs a mutex. The
difference we expect to document is *how that guarantee is obtained*: in Rust
the compiler enforces it through ownership, while in Go it is a convention the
compiler does not check.

### Components

| Component | Responsibility | Rust | Go |
|---|---|---|---|
| Domain model | Message representation, rendering, predicates | `src/model.rs` | `internal/chat/model.go` |
| Hub | Owns history; applies posts; answers queries | `src/hub.rs` | `internal/chat/hub.go` |
| Driver | CLI modes: demo, bench, REPL | `src/main.rs` | `main.go` |
| Tests | Model predicates and hub behaviour | `#[cfg(test)]` in `model.rs` | `internal/chat/model_test.go` |

### Command set (identical in both)

- `Post { user_id, username, kind }`
- `Ask { query, reply }` where query is `History{limit}` | `ByUser(name)` | `Search(keyword)`
- `Stats { reply }`
- `Shutdown`

### CLI modes (identical in both)

| Mode | Purpose |
|---|---|
| `demo` | Scripted concurrent session, then filter, search and summary |
| `bench P M` | `P` concurrent producers × `M` messages each; reports throughput |
| *(no arguments)* | Interactive REPL: `/send`, `/dm`, `/history`, `/user`, `/search`, `/stats`, `/quit` |

---

## 3. Anticipated language differences

These are the differences we expect to find. Day 3 reports what we actually
found, including where we were wrong.

| Area | Expectation |
|---|---|
| **Sum types** | Rust's `enum` carries a payload per variant and `match` is exhaustive. Go has no sum type, so we expect either a tagged struct with unused fields or an interface plus a type switch, neither of which is exhaustiveness-checked. |
| **Concurrency model** | Rust uses `async`/`await` over a task scheduler; Go uses goroutines that look synchronous. We expect Go's code to read more simply and Rust's to express more in its types. |
| **Error handling** | Rust's `Result` must be consumed; Go's `error` return can be silently discarded with `_`. |
| **Memory management** | Neither needs manual freeing. Rust frees deterministically at scope exit; Go uses a garbage collector, which we expect to show up as higher memory use under load. |
| **Modularity** | Rust modules with `pub` versus Go packages with capitalisation-based export. |
| **Performance** | Both compile to native code. We expect them within the same order of magnitude, with Rust ahead because it has no GC and no interface dispatch in the hot path. |

### Risks identified up front

1. **Ordering.** If commands travel on separate channels, Go's `select` chooses
   randomly among ready cases, so a query could overtake queued posts. Mitigation:
   one ordered command channel in both implementations.
2. **Demo determinism.** Concurrent interleaving differs run to run. Mitigation:
   assert on counts and filters, not on exact ordering.
3. **Toolchain parity.** Both must build and run on the same machine so timings
   are comparable. Mitigation: a single WSL2 Ubuntu 24.04 environment.

---

## 4. Timeline

| Day | Milestone | Owner |
|---|---|---|
| **Fri** | Shared specification, architecture, command set, repository setup | Bijay Raj KC (lead), all |
| **Fri** | Agreement that both implementations expose identical CLI modes | All |
| **Sat AM** | Domain model and hub in both languages | Rahul (Rust), Krinal (Go) |
| **Sat PM** | Concurrent demo mode working end to end; unit tests | Rahul, Krinal |
| **Sat PM** | Core-functionality report with screenshots | Bijay |
| **Sun AM** | Filtering, search, direct messages, statistics, REPL | Rahul, Krinal |
| **Sun AM** | Race detector and test pass in both | Krinal |
| **Sun PM** | Benchmark runs and comparison report | Rahul, Bijay |
| **Sun PM** | Presentation slides and rehearsal | All |

---

## 5. Build and run

Both implementations are built and run in **WSL2 Ubuntu 24.04** so that one
machine and one set of timings apply to both.

```bash
# Rust (rustc/cargo 1.95.0)
cd rust && cargo build --release
./target/release/concurrent_chat demo
./target/release/concurrent_chat bench 8 25000

# Go (go 1.27.1)
cd go && go build -o chat .
./chat demo
./chat bench 8 25000
```

Day 1 contains design documentation only; the commands above apply to the Day 2
and Day 3 repositories.

---

## 6. Repositories

| Deliverable | Repository | Visibility |
|---|---|---|
| Day 1 — Planning and design | `MSCS-632-M30-Group-D1` | Public |
| Day 2 — Core functionality | `MSCS-632-M30-Group-D2` | Private |
| Day 3 — Final, comparison, presentation | `MSCS-632-M30-Group-D3` | Private |
