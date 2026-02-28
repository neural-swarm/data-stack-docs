# 9. Consistency and Consensus
english | [russian](09-consistency-and-consensus.ru.md)

**TL;DR:** Consistency models, linearizability and causality; total order and consensus; why 2PC and coordination are expensive and where they are actually needed.

---

## Consistency Guarantees

- Consistency is a set of promises about visibility and ordering: which reads are allowed after which writes.
- Choose guarantees for the job: causal is often enough for UX; linearizability is needed for locks/uniqueness.

## Linearizability

- Linearizability means the system behaves like a single copy: operations look atomic and ordered by real time.
- If a write finishes before a read starts, the read must see the result (for that object/key).

### What Makes a System Linearizable?

- Each operation has an execution interval; there exists a “linearization point” inside it where the operation appears to happen instantaneously.
- This is stronger than eventual consistency: it constrains the observed order under concurrency and delays.

### Relying on Linearizability

- Needed for: distributed locks and leader election, uniqueness constraints, cross-channel dependencies (DB ↔ queue).
- Without it you often need extra protection (e.g., fencing tokens) or you must centralize the invariant.

### Implementing Linearizable Systems

- Typically requires coordination: a strict leader with majority acknowledgement, or full consensus.
- Quorums W+R>N by themselves do not guarantee linearizability under concurrent writes and network delays.

### The Cost of Linearizability

- Cost: higher latency (wait for acknowledgements) and reduced availability under network partitions (CAP).
- In multi‑DC setups the cost grows with RTT; many systems pick causal/eventual for availability.

## Ordering Guarantees

- Correctness depends on event ordering: causality (happens‑before), sequences, total order.
- Causal order is partial: many events are concurrent and incomparable.

### Ordering and Causality

- Causal consistency guarantees dependent events are seen in the right order, but does not impose an order on all concurrent events.
- Linearizability is stronger than causal: it adds a real‑time requirement.

### Capturing causal dependencies

- To preserve causality you need metadata: version vectors, causal context, logical clocks.
- A single scalar timestamp cannot distinguish concurrency from causality.

### Sequence Number Ordering

- Sequence numbers make a stream replayable, but generating a global sequence in a distributed system usually leads to consensus.
- Lamport timestamps preserve happens‑before (if A→B then ts(A)<ts(B)), but not the other way around.

### Total Order Broadcast

- TOB: all nodes deliver messages in the same order; a foundation for state machine replication.
- Usually implemented via Raft/Paxos‑class consensus, or reduced to strong coordination.

## Distributed Transactions and Consensus

- 2PC provides atomic commit across nodes, but can block participants if the coordinator fails.
- Consensus chooses leaders and command order under failures; it underpins coordination services.

### Atomic Commit and Two-Phase Commit (2PC)

- Prepare phase: participants promise they can commit (log + locks). Commit/abort phase: the coordinator announces the decision.
- Weak point: “in doubt” — if the coordinator crashes, participants hold locks until they learn the outcome.

### Three-phase commit

- 3PC tries to remove blocking, but requires synchronous assumptions (bounded delays), which rarely hold in real networks.

### Distributed Transactions in Practice

- Practical issues: long-held locks, complex recovery, high latency, fragile failure modes.
- Common alternatives: sagas, outbox/inbox, idempotency, derived state via streams (event-driven).

## Fault-Tolerant Consensus

- Consensus is tied to TOB: a single command log provides total order.
- Epoch/term numbers + quorums cut off old leaders and avoid split brain.

## Membership and Coordination Services

- Membership service: cluster membership, failure detection, join/leave events. Centralized vs decentralized membership. You can’t perfectly detect failures (FLP). heartbeats, gossip protocol, timeouts.
- Coordination service: leader election, roles, state sync, configs, distributed locks. Paxos, Raft.
- The hard part is membership changes and correct timeouts in a world of partial failures.

### Allocating work to nodes

- Central coordinator: MapReduce master, Kubernetes scheduler
- Distributed queue: Kafka consumer groups, RabbitMQ, SQS
- Consistent hashing: key → hash → node. When adding a node, only a fraction is reassigned: Cassandra, DynamoDB, Redis Cluster.

### Service Discovery

- Hardcoded, client-side discovery, server-side discovery

## Summary

- Strong guarantees (linearizability/consensus) provide correctness for critical invariants, but cost latency and availability under partitions.
- Weaker guarantees (causal) often scale better and are sufficient for many product scenarios.

---

## Terms

- **Consistency model:** rules for visibility and ordering of operations
- **Linearizability:** like a single copy of data ordered by real time
- **Causal consistency:** preserve causal dependencies without total order
- **Happens-before:** the causal relationship between events
- **Lamport timestamp:** logical time that preserves happens‑before but can’t distinguish concurrency
- **Total Order Broadcast:** everyone delivers messages in the same order
- **Consensus:** agreement on a value/leader under failures
- **Epoch/term:** leader generation number to cut off old leaders
- **CAP theorem:** under partition you can’t have both strong consistency and availability
- **2PC:** two-phase commit for a distributed transaction
- **Prepare:** the participant’s promise: “I can commit later”
- **Lease:** time-bounded ownership (often paired with fencing)
