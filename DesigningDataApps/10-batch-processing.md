# 10. Batch Processing
english | [russian](10-batch-processing.ru.md)

**TL;DR:** Batch processing from Unix pipelines to MapReduce and DAG engines; why sorting/shuffle are fundamental for joins and GROUP BY, and how reliability and derived data work.

---

## Batch Processing

- Batch = processing a finite dataset in batches: high throughput and reproducibility, but high end-to-end latency.
- Main pros: easy to replay, convenient to recompute when logic changes, easier to reason about correctness.
- Main cons: not suitable for interactive SLAs, and often requires heavy shuffle/sort stages.

## Batch Processing with Unix Tools

- Unix pipelines are a minimalist batch form: streaming interfaces (stdin/stdout), composition of small utilities, transparent intermediate steps.
- This philosophy carries over to big data: staged processing, clear interfaces, fast experimentation.

### Simple Log Analysis

- Typical task: filter logs → extract fields → group → sort → aggregate.
- Even locally you often hit sorting (for grouping large volumes) and I/O.

### Chain of commands versus custom program

- A command chain is faster for prototyping and flexibility; a custom program can be more optimal once bottlenecks are known.
- Good practice: compose and validate first, then optimize the bottleneck stages.

### Sorting versus in-memory aggregation

- Sorting enables aggregation of huge datasets via external sort without keeping everything in RAM.
- In-memory aggregation is faster but RAM-limited and sensitive to skew (super-popular keys).

### The Unix Philosophy

- Key principles: a common interface, separation of logic from “plumbing”, transparency, experimentation.
- In data pipelines this means: separate transformations from orchestration and keep intermediate results observable.

## MapReduce and Distributed Filesystems

- MapReduce = map + shuffle/sort + reduce on top of a distributed filesystem (DFS), where data is replicated for reliability.
- The scaling idea: many independent map tasks, then group by key and reduce-aggregate.

### MapReduce Job Execution

- Map reads input blocks (often near the data), writes intermediate key/value pairs.
- Shuffle moves and sorts by key so all values for a key end up at the same reducer.
- Task failures are expected: the scheduler restarts tasks; determinism and no external side effects matter.

## Reduce-Side Joins and Grouping

- Reduce-side join is universal: everything with the same key meets in one reducer after shuffle.
- But it’s expensive due to full shuffle and sorting — often the main “tax” of MapReduce.

### Sort-merge joins

- Join is implemented as merging sorted streams by key: efficient at scale, but requires sort/shuffle.
- Ordering and partitioning by key become fundamental to performance.

### GROUP BY

- GROUP BY in MapReduce is a special case: map emits the key, shuffle groups, reduce aggregates.
- Commutative/associative aggregates can be partially computed before shuffle (combiner), reducing network traffic.

### Handling skew

- Skew destroys parallelism: one heavy key overloads a reducer.
- Techniques: salting keys, handling heavy hitters separately, two-phase aggregation, repartitioning (custom partitioner).

## Map-Side Joins

- Map-side joins avoid shuffle if data is co-partitioned/sorted, or if one side is small enough to broadcast.
- This is faster, but requires preparation (partitioning/sorting) and careful assumptions about size/format.

### Broadcast hash joins

- A small lookup table is loaded into each mapper’s memory as a hash table; fast but limited by RAM and table size.

### Partitioned hash joins

- Both sides are pre-partitioned the same way; each mapper processes its partition pair, avoiding a global shuffle.

### Map-side merge joins

- If both sides are already sorted and co-partitioned, the join can be done as a streaming merge on the map side.

## The Output of Batch Workflows

- Batch output is an artifact: a search index, an analytics “mart”, a file, or KV state for online serving.
- Think of output as a published product: versioning, schema, data quality, idempotent writes.

### Key-value stores as batch process output

- A common pattern: batch computes derived state and loads it into a KV store so online queries are fast.
- This separates heavy computation from the serving path (serving layer).

## Comparing Hadoop to Distributed Databases

- Hadoop/DFS ecosystems focus on batch and tolerate frequent task failures.
- Distributed databases focus on interactivity, indexes, and transactions; different latency and execution trade-offs.

## Beyond MapReduce

- A MapReduce limitation is mandatory materialization between stages (files on disk): reliable, but expensive for I/O and latency.
- DAG/dataflow engines reduce materialization, enable optimizations, and are faster for iterative workloads.

### Materialization of Intermediate State

- Materialization is useful as a checkpoint (debugging, replay, reliability), but costly at large scale.
- Alternative: lineage/recompute + checkpoints (a compromise between speed and recovery).

## Graphs and Iterative Processing

- Iterative algorithms fit poorly in MapReduce because they require shuffle on every iteration.
- Pregel-like models keep state near vertices/partitions and synchronize in supersteps.

## High-Level APIs and Languages

- Trend: move from low-level APIs to declarative queries (SQL/DataFrame) so the engine can optimize plans.
- Specialization also grows: domains (ML, graphs, streams) need different state/time models.

## Summary

- Batch is about reproducibility and scale, where sorting/shuffle are central for joins and aggregates.
- Modern systems trend toward DAG/SQL engines and better support for iteration and optimization.

---

## Terms

- **External sort:** sorting with disk when RAM is insufficient
- **MapReduce:** computation model: map + shuffle/sort + reduce
- **Shuffle:** repartitioning by key (network + sort), the expensive stage
- **Combiner:** partial aggregation before shuffle to reduce traffic
- **Skew:** uneven key/load distribution
- **Reduce-side join:** join via shuffle (universal but expensive)
- **Map-side join:** join without shuffle given prepared data / small table
- **Serving layer:** an online layer serving queries based on batch output
- **Materialization:** writing intermediate results as a durable checkpoint
- **DAG engine:** an engine that executes an operation graph with optimizations (lineage, pipelining)
