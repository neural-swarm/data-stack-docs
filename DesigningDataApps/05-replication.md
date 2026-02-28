# 5. Replication
english | [russian](05-replication.ru.md)

**TL;DR:** Why replicas exist, how leader/follower, multi‑leader, and leaderless setups work, what replication lag breaks, and how to detect/resolve conflicts.

---

## Leaders and Followers

- The leader accepts writes; replicas apply the log. Reads from replicas can be faster, but may be stale.

## Synchronous Versus Asynchronous Replication

* **Synchronous replication**: a write is acknowledged only after it is committed on the leader and on a follower (or a quorum). Minimal risk of data loss, but higher latency because you wait for acknowledgements.
* **Asynchronous replication**: the leader acknowledges after local commit. Lower latency and higher throughput, but if the leader crashes you can lose the latest writes.
* There are intermediate variants (semi-sync, quorum) to balance speed and reliability.

## Setting Up New Followers

* A new follower receives a **snapshot** — a consistent copy of the data at some point in time.
* Then it performs **log catch‑up** — applying changes after the snapshot.
* Key coordinates: **LSN / offset / term-index**, to resume from the exact position.
* Snapshots and logs must match; otherwise you need a resync.

## Handling Node Outages

* Behavior depends on the node’s role (follower or leader).
* RPO/RTO and automatic failover mechanisms matter.

### Follower failure: Catch-up recovery

* After recovery the follower catches up using the log from the last LSN.
* If the needed logs are gone (retention), it needs a new snapshot.
* Large replication lag reduces the usefulness of the node.

### Leader failure: Failover

* A new leader is selected (often via a consensus algorithm).
* Clients switch to the new endpoint.
* Risks: **split brain** and loss of the latest writes with async replication.
* Protection: quorums, term/epoch control, fencing mechanisms.

## Implementation of Replication Logs

- Statement‑based (fragile), WAL shipping (low level), logical log (more robust), triggers (flexible/risky).

### Statement-based replication

- Nondeterminism (NOW/RAND), side effects, different execution plans.

### Write-ahead log (WAL) shipping

- Fast, but tied to the engine’s internal format/version.

### Logical (row-based) log replication

- Transfers row changes/events; convenient for integrations and across versions.

### Trigger-based replication

- Logic in triggers; hard to maintain and test.

## Problems with Replication Lag

* **Replication lag** is how far followers are behind the leader in applying writes.
* It breaks user expectations: a user may not see data they just wrote.
* It can break causal order if reads go to different replicas with different lag.
* It requires explicit read routing and read consistency strategies.

### Reading Your Own Writes

* Users expect to see their own changes immediately after writing.
* Solutions: read from the leader, use **sticky sessions** (pin a client to a replica), or pass a version/LSN and require reads at least that fresh.

### Monotonic Reads

* Within one session, data shouldn’t “go backwards”.
* Achieved by sticking to one replica or tracking the minimum observed version and reading only from replicas that have reached it.

### Consistent Prefix Reads

* Observations should preserve causal order: you shouldn’t see an effect without seeing its cause.
* Requires reading from replicas that apply operations in the right order without gaps.

### Solutions for Replication Lag

* Route reads by freshness (read from leader / nearest up‑to‑date replica).
* Wait until a required version is reached (read‑after‑write via LSN/offset).
* Monitor **replication lag / staleness** and exclude overly stale nodes from reads.

## Multi-Leader Replication

- Multiple leaders accept writes (multi‑DC, offline, collaborative editing), but conflicts are inevitable. The core challenge is convergence: all replicas must eventually reach the same state.

### Use Cases for Multi-Leader Replication

- multi‑datacenter, offline clients, collaborative editing.

#### Multi-datacenter operation

- Local writes and surviving a DC failure, at the cost of complicated consistency. Writes from different DCs may conflict. Replication is asynchronous → eventual consistency.

#### Clients with offline operation

- Local changes + later sync → you need a merge strategy.

#### Collaborative editing

- Concurrent edits require CRDT (Conflict‑free Replicated Data Types) / OT (Operational Transformation)‑class ideas or custom merge logic.

### Handling Write Conflicts

- Avoid (routing/ownership) or resolve (merge/LWW/custom rules); the goal is convergence.

#### Synchronous versus asynchronous conflict detection

- Sync: coordinate up front (distributed locks or quorum protocols). Async: detect and resolve later.

#### Conflict avoidance

- Route all writes for a key to one leader (routing by key) or assign ownership, e.g., split keys across DCs.

#### Converging toward a consistent state

Replicas must converge even if messages are delivered in different orders.
You generally need:
- idempotent operations,
- deterministic merge rules,
- tolerance to redelivery,
- no infinite update loops.

#### Custom conflict resolution logic

Domain‑specific merge rules can be complex. Options:
- Last Write Wins;
- field‑level merge;
- domain rules (e.g., summing counters);
- manual conflict resolution by the user.

#### What is a conflict?

- A conflict is a violated invariant, not just “two parallel writes”.

### Multi-Leader Replication Topologies

- Ring/star/mesh affect propagation delay and feedback loops.

## Leaderless Replication

Writes go to multiple nodes with no leader; reconciliation happens on reads and/or in the background. Reads also query multiple nodes. Cassandra, DynamoDB, Riak.

How it works:
- Each key has a replica set (often via consistent hashing).
- On write, a coordinator (any node) sends the write to N replicas.
- On read, the coordinator queries several replicas and compares versions.

Problems:
- Replicas can diverge.
- You need mechanisms to detect and resolve conflicts.

### Writing to the Database When a Node Is Down

- Write to the available nodes; later catch up/transfer when the node recovers.

### Read repair and anti-entropy

- Read repair fixes on reads; anti‑entropy synchronizes in the background.

### Quorums for reading and writing

- W+R>N increases the chance of a fresh read, but does not guarantee linearizability.

### Limitations of Quorum Consistency

- Networks/timeouts/concurrent writes/clock skew (LWW) break expectations; stale reads are still possible.

### Monitoring staleness

Measure:
- replication lag
- versions (vector clock size, offset)
- convergence lag

Tools: replication delays, conflicts, hinted handoff queue size.

### Sloppy Quorums and Hinted Handoff

- Write to “temporary” nodes during failures and later hand data to the owners; increases availability, complicates consistency.

#### Multi-datacenter operation

- Cross‑DC latency increases the conflict and staleness window. Local quorum vs global quorum.

### Detecting Concurrent Writes

- You must distinguish causal order from concurrency (happens‑before).

#### Last write wins (discarding concurrent writes)

- LWW is simple, but can lose updates (overwrite).

#### The happens-before relationship and concurrency

- Causality ≠ timestamp; concurrent means there is no happens‑before relationship.

#### Capturing the happens-before relationship

- Versions/metadata help detect concurrency.

#### Merging concurrently written values

- Merge strategies (e.g., CRDT‑style) reduce manual conflicts.

#### Version vectors

- Per‑replica version vectors capture causality and detect conflicts.

## Summary

- Replication improves availability/latency, but requires a consistency strategy and conflict handling.
