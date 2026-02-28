# 11. Stream Processing
english | [russian](11-stream-processing.ru.md)

**TL;DR:** Streams, brokers, and log-oriented queues; CDC and event sourcing; time and windows; joins and state; fault tolerance via offsets/checkpoints and idempotency.

---

## Transmitting Event Streams

- Streams are infinite sequences of events; the goal is to continuously turn events into state/metrics/views.
- Streaming requires buffering and flow control (backpressure); otherwise overload triggers retry storms and growing lag.

## Messaging Systems

- Classic messaging relies on acknowledgements (acks) and redelivery on failures.
- This almost always implies at-least-once delivery: to avoid double effects, handlers must be idempotent or deduplicate.

### Direct messaging from producers to consumers

- Direct connections reduce layers, but complicate discovery, multicast, backpressure, and managing many consumers.
- On failure you must implement buffering and retries yourself.

### Message brokers

- A broker decouples producers from consumers, buffers spikes, provides fan-out, and simplifies operations.
- Key parameters: ordering (per partition), retention, acks, DLQ, retry policies.

## Partitioned Logs

- Log-oriented brokers store messages as an append-only log in partitions; consumers read by offset.
- This enables replay, independent consumers, and a natural “source of truth” for events.

### Consumer offsets

- An offset is a read cursor; for “effectively once” you must coordinate committing the offset with writing the output/state.
- If you commit the offset early → risk of loss; if late → risk of duplicates (handled by idempotency).

### When consumers cannot keep up with producers

- Growing consumer lag signals overload or poor key partitioning.
- Mitigations: scale consumers, change key partitioning, optimize state access, throttle producers, add rate limits.

### Replaying old messages

- Replay is used to recompute derived state, fix bugs, and build new views.
- Requires schema compatibility, versioned logic, idempotency, and clear retention/compaction policies.

## Databases and Streams

- Streams and databases complement each other: DB changes can be streamed as events (CDC), and streams can be materialized into tables/indexes.
- The goal is to keep systems in sync without fragile “dual writes”.

## Change Data Capture

- CDC reads the DB log (WAL/binlog) and publishes changes as a stream.
- Challenges: initial snapshot, snapshot↔log alignment point, ordering, schema evolution.

## Event Sourcing

- Event sourcing stores fact events as the source of truth; current state is the result of folding the log.
- Pros: audit and replay; cons: growing history, migrations, and the need for materialized views for fast reads.

### Commands and events

- A command is an intention and may be rejected; an event is a fact of a change that happened.
- This separation helps with validation, idempotency, and a clear audit trail.

## State, Streams, and Immutability

- Immutable events are easier to replicate and verify: you append facts instead of rewriting the past.
- Constraints: corrections/deletions (privacy), volume growth, need for compaction and snapshots.

## Processing Streams

- A stream processor maintains state and updates it per event; state durability must be managed explicitly.
- Many streaming mistakes are mistakes in handling time, ordering, and state recovery.

### Uses of Stream Processing

- CEP (event pattern detection), stream analytics (windowed aggregates), materialized views, search index updates, service integration.
- In all cases: late events, per-key ordering, and idempotency are critical.

## Reasoning About Time

- Event time (when it happened) ≠ processing time (when it was processed). With delays and reordering this changes aggregate results.
- Windows and watermarks define when you have “enough data” and how to handle late events.

### Types of windows

- Tumbling (non-overlapping), sliding, session windows (periods of activity).
- Window choice affects state size and system load.

## Stream Joins

- Streaming joins require buffering events (state) until a window closes; that increases cost and complexity.
- Stream-table joins (enrichment) require a versioned reference (“as-of” semantics), otherwise results depend on update timing.

## Fault Tolerance

- Goal: after a failure, continue with consistent state and without an incorrect double effect.
- Techniques: checkpoints, microbatching, atomic commit of state+offset (or an equivalent), idempotency, deduplication.

### Microbatching and checkpointing

- Microbatching simplifies consistency: process a small batch of events as a transaction.
- Checkpoints persist state and position for fast recovery, but too frequent checkpoints are expensive (cost vs recovery trade-off).

### Idempotence

- Duplicates are inevitable with at-least-once; use upsert/merge by key, operation IDs, and deduplication.
- Idempotency is the main practical way to get close to exactly-once.

## Summary

- Stream processing = logs + state + time.
- Success depends on correct management of offsets, checkpoints, windows, and idempotent handling.

