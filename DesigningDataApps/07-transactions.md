# 7. Transactions
english | [russian](07-transactions.ru.md)

**TL;DR:** ACID, isolation levels and concurrency anomalies; how to achieve serializability via serial execution, 2PL, and SSI.

---

## The Slippery Concept of a Transaction

- A transaction is a bundle of correctness guarantees; in different systems “transactions” can mean different things.

## The Meaning of ACID

- Atomicity (all-or-nothing), Consistency (invariants), Isolation (concurrency), Durability (survive commit).

### Atomicity

- No partial effects are visible; abort rolls back changes.

### Consistency

- Domain invariants; the database helps with constraints, but logic often lives in the application.

### Isolation

- Concurrent transactions should not produce anomalies; in practice isolation levels are trade‑offs.

### Durability

- After commit the data survives failures (WAL/fsync/replication/snapshots).

## Single-Object and Multi-Object Operations

- Many systems are atomic at the level of a single object; but invariants are often cross-object.

### Single-object writes

- Atomic operations on a key/document simplify design, but don’t solve cross-object invariants.

### The need for multi-object transactions

- Transfers/counters/relationships often require multi-record transactions.

### Handling errors and aborts

- Retries must account for idempotency and external side effects.

## Weak Isolation Levels

- Weaker levels allow anomalies for performance; you must know which anomalies are possible.

### Read Committed

- Prevents dirty reads/writes; non-repeatable reads and phantoms are still possible.

#### No dirty reads

- Don’t read uncommitted data from another transaction.

#### No dirty writes

- Don’t overwrite uncommitted data from another transaction.

#### Implementing read committed

- Write locks + visibility rules for versions.

### Snapshot Isolation and Repeatable Read

- Read from a snapshot + detect write conflicts; good for readers.

#### Implementing snapshot isolation

- MVCC stores multiple versions; each transaction sees a consistent snapshot.

#### Visibility rules for observing a consistent snapshot

- Only versions committed before the “snapshot time” are visible.

#### Indexes and snapshot isolation

- Indexes must respect visibility, otherwise you can get “phantom” results.

#### Repeatable read and naming confusion

- Terms differ across DBMSs: sometimes repeatable read = snapshot isolation.

### Preventing Lost Updates

- Lost updates are prevented with atomic operations, locks, or optimistic version checks.

#### Atomic write operations

- Increment/CAS on the database side instead of read‑modify‑write in the app.

#### Explicit locking

- SELECT … FOR UPDATE serializes access to rows.

#### Automatically detecting lost updates

- Optimistic concurrency: update only if the version/etag hasn’t changed.

#### Compare-and-set

- CAS is a conditional write if the expected state matches.

#### Conflict resolution and replication

- Under replication, you must decide where to detect the conflict and how to merge it.

### Write Skew and Phantoms

- Snapshot isolation does not always prevent write skew; phantoms can break invariants.

#### Characterizing write skew

- Two transactions read a condition and write different rows, together violating a rule.

#### More examples of write skew

- Reservations, limits, schedules — cases with global invariants.

#### Phantoms causing write skew

- New rows matching the predicate appear between read and write.

#### Materializing conflicts

- Make the “conflict object” explicit (e.g., lock a row) so competitors conflict on one record.

### Serializability

- The strongest guarantee: as if transactions ran in some sequential order.

### Actual Serial Execution

- Serializing by executing one at a time (or within a shard) is simple, but limits throughput.

#### Encapsulating transactions in stored procedures

- Fewer network round trips; logic closer to the data.

#### Pros and cons of stored procedures

- Pros: performance/control; cons: harder to evolve and test.

#### Partitioning

- Serial execution is easier within one shard; cross-shard transactions are harder.

#### Summary of serial execution

- Works when transactions are short and load is controlled.

### Two-Phase Locking (2PL)

- A pessimistic approach: hold locks until commit; removes anomalies, but causes contention and deadlocks.

#### Implementation of two-phase locking

- Shared/exclusive locks + strict hold rules.

#### Performance of two-phase locking

- Throughput drops, latency grows; you need deadlock detection/timeouts.

#### Predicate locks

- Locks on a predicate to prevent phantoms.

#### Index-range locks

- A practical form of predicate locks via index ranges.

### Serializable Snapshot Isolation (SSI)

- Optimistic serializability on top of snapshot isolation: track dangerous dependencies and abort.

#### Pessimistic versus optimistic concurrency control

- Pessimistic: lock up front; optimistic: run and check at commit.

#### Decisions based on an outdated premise

- Danger: a decision made on an out-of-date snapshot.

#### Detecting stale MVCC reads

- Track reads of versions that later conflict with writes.

#### Detecting writes that affect prior reads

- Record read→write dependencies that make a schedule non-serializable.

#### Performance of serializable snapshot isolation

- Often better than 2PL at low contention; at high contention you get many aborts/retries.

## Summary

- Choose an isolation level based on your invariants; serializability is achieved via locks or via checks+aborts.

---

## Terms

- **ACID:** Atomicity, Consistency, Isolation, Durability
- **Isolation level:** which concurrency anomalies are allowed
- **Dirty read/write:** reading/writing uncommitted data
- **Snapshot isolation:** reading from a snapshot + write conflict checks
- **MVCC:** multi-version concurrency control
- **Lost update:** an update lost due to a race
- **Write skew:** invariant violation due to concurrent writes to different rows
- **Phantom:** rows appearing/disappearing for a predicate
- **Serializability:** equivalence to sequential execution
- **2PL:** two‑phase locking: serialize via locks
- **Predicate lock:** lock on a predicate to eliminate phantoms
- **Index-range lock:** a lock on an index range
- **SSI:** Serializable Snapshot Isolation: serializability by detecting dangerous dependencies
- **Optimistic concurrency:** check for conflicts at commit (versions/CAS)
- **CAS:** compare‑and‑set: conditional update
- **Idempotency:** repeating an operation is safe
