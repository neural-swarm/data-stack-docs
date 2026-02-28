# 6. Partitioning
english | [russian](06-partitioning.ru.md)

**TL;DR:** How to split data across nodes (range/hash), deal with hot spots, build secondary indexes under sharding, rebalance, and route requests.

---

## Partitioning and Replication

- Partitioning distributes data; replication duplicates it for fault tolerance.

## Partitioning of Key-Value Data

- Goal: even load and a manageable amount of data per node.

## Partitioning by Key Range

- Great for range queries; risk of hot ranges (e.g., time-based ranges).

## Partitioning by Hash of Key

- Distributes more evenly, but range queries require scatter/gather.

## Skewed Workloads and Relieving Hot Spots

- Hot keys break balance; use caches, salting, splitting popular keys, more partitions.

## Partitioning and Secondary Indexes

- Secondary indexes: local (by document) are easier to write; global (by term) are faster to read.

### Partitioning Secondary Indexes by Document

- The index is local to each shard’s data; index queries often hit all shards. Cassandra, HBase.

### Partitioning Secondary Indexes by Term

- A global index by value; reads are faster, writes are harder and require distributed updates. MongoDB, DynamoDB (GSI), CockroachDB, Elasticsearch/OpenSearch.

## Rebalancing Partitions

- When the number of nodes changes, move partitions without large performance drops and without massive rebuilds.

### Strategies for Rebalancing

- Move “chunks” (partitions), not individual keys; limit migration speed.

#### How not to do it: hash mod N

- Changing N changes almost all assignments → huge migration.

#### Fixed number of partitions

- Create many partitions up front → easier balancing, but you must guess the number. Kafka.

#### Dynamic partitioning

- Partitions split as they grow; adaptive, but harder to manage, and split/merge are heavy operations. Bigtable, HBase.

#### Partitioning proportionally to nodes

- The number of partitions grows with nodes; a compromise between flexibility and manageability. Cassandra / Dynamo‑style.

## Operations: Automatic or Manual Rebalancing

- Automation is convenient, but you need safeguards against “migrating at the worst moment” (limits, maintenance windows).

## Request Routing

The client/coordinator must know where a key lives:
- routing layer (Elasticsearch coordinating node, MongoDB mongos)
- smart client (Cassandra driver, Kafka producer, Dynamo‑style)
- catalog/metadata (Bigtable)

## Parallel Query Execution

- Scatter/gather speeds up big queries, but tail latency depends on the slowest shard.

Ways to reduce the impact of tail latency:
- Replication + speculative execution: if a shard is slow, send the request to a replica.
- Avoid scattering to all shards (prefer key-based, range-based)
- Partial aggregation on the shard
- Timeouts (return partial results)

## Summary

- Choose range vs hash based on query types; think through routing and rebalancing up front.

---

## Terms

- **Partition/Sharding:** splitting data into parts across nodes
- **Range partitioning:** sharding by key ranges
- **Hash partitioning:** sharding by hash of the key
- **Hot spot:** a key/range with disproportionately high load
- **Secondary index:** an index on a non-primary field
- **Scatter/Gather:** fanning a query out to shards and collecting results
- **Rebalancing:** redistributing partitions as the cluster changes
- **Routing:** sending a request to the right shard
- **Salting:** adding something to a key to spread load
- **Coordinator:** a node/layer that routes and aggregates
