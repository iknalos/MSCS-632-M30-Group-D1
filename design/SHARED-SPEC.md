# ConcurrentChat — Shared Specification

This document is the contract both implementations must satisfy. It exists so
that any difference between the Rust and Go code is a consequence of the
language, not of two teams solving different problems.

## 1. Domain model

### Message

| Field | Type (conceptual) | Notes |
|---|---|---|
| `id` | unsigned 64-bit | Assigned by the hub, monotonically increasing from 1 |
| `timestamp` | local date-time | Assigned by the hub at append time |
| `user_id` | unsigned 32-bit | Caller-supplied |
| `username` | string | Caller-supplied |
| `kind` | MessageKind | See below |

### MessageKind

| Variant | Payload | Searchable body? |
|---|---|---|
| `Text` | `body: string` | yes |
| `Direct` | `to: string`, `body: string` | yes |
| `Join` | none | no |
| `Leave` | none | no |
| `System` | `body: string` | yes |

**Note for implementers.** Rust models this as a data-carrying `enum`. Go has no
equivalent, so the Go side must choose a representation and document the
consequence. Either choice is acceptable provided the external behaviour is
identical.

## 2. Hub contract

The hub owns the history. No other component may hold a reference to it.

| Command | Request | Response |
|---|---|---|
| Post | `user_id`, `username`, `kind` | none |
| Ask | `query` | list of matching messages, oldest first |
| Stats | — | total, counts per user, counts per kind |
| Shutdown | — | number of messages recorded |

### Queries

| Query | Semantics |
|---|---|
| `History { limit }` | The most recent `limit` messages, oldest first. `limit` larger than the history returns everything. |
| `ByUser(name)` | Every message whose `username` equals `name`, case-insensitive. |
| `Search(keyword)` | Every message whose searchable body contains `keyword`, case-insensitive. Kinds with no body never match. |

### Ordering guarantee

Commands must be processed in the order they were accepted. A query issued
after a post must observe that post. **Implementations must therefore use a
single ordered command channel**, not one channel per command type.

## 3. Behaviour

### `demo` mode

1. Post `System("room opened")`.
2. Start one concurrent worker per participant: alice, bob, carol. Each posts
   `Join`, then its scripted lines interleaved with a yield, then `Leave`.
3. Wait for all workers.
4. Post one `Direct` message from bob to carol.
5. Post `System("room closing")`.
6. Print, in order: full history, `ByUser("alice")`, `Search("notes")`, summary.
7. Shut down and report the recorded count.

Expected totals: **18 messages**; by user alice=5, bob=6, carol=5, server=2; by
kind text=9, join=3, leave=3, system=2, direct=1. Interleaving order will vary
between runs and between languages; the counts must not.

### `bench P M` mode

Start `P` concurrent producers, each posting `M` text messages. Measure
wall-clock time from before the producers start until a round-trip `Stats` query
returns, which guarantees the hub has drained its queue. Report elapsed
milliseconds and messages per second.

### Interactive mode

| Command | Effect |
|---|---|
| `/send <user> <message>` | Post a `Text` message |
| `/dm <from> <to> <message>` | Post a `Direct` message |
| `/history [n]` | Last `n` messages, default 20 |
| `/user <name>` | Filter by sender |
| `/search <keyword>` | Keyword search |
| `/stats` | Summary counts |
| `/help`, `/quit` | Usage, exit |

## 4. Non-goals

Out of scope for this project: real networking, authentication, persistence to
disk, and a graphical interface. The assignment permits a local simulation, and
keeping the surface small keeps the comparison focused on language design.

## 5. Acceptance checklist

- [ ] `demo` produces the expected counts in both languages
- [ ] `ByUser` and `Search` are case-insensitive in both
- [ ] `Join` and `Leave` never match a keyword search
- [ ] Unit tests pass in both
- [ ] Go passes `go vet` and `go test -race`
- [ ] Both build with no warnings
- [ ] `bench` reports throughput in both
