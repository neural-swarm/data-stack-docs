# 3. Storage and Retrieval
english | [russian](03-storage-and-retrieval.ru.md)

**TL;DR:**
Indexes and the physical layout of storage determine the real characteristics of a database: latency, throughput, amplification effects, behavior under load, and scalability. Different structures (B‑tree, LSM, column store, inverted index) are optimized for different workload types.

------------------------------------------------------------------------

## Data Structures That Power Your Database

An index is an alternative physical representation of data, optimized for a particular access pattern:

-   Point lookup
-   Range scan
-   Sorting
-   Aggregation
-   Text search

Any index introduces:

1.  Write cost
2.  Storage cost
3.  Complexity cost

Core principle:
**An index is a trade‑off between write locality and read locality.**

------------------------------------------------------------------------

## Hash Indexes

### Physical model

    hash(key) → bucket → pointer → value

Used in:

-   In-memory hash table
-   Extendible hashing
-   Cuckoo hashing

### Limitations

-   No range scans
-   No ordering
-   Expensive rehash

Good for key‑value workloads.

------------------------------------------------------------------------

## SSTables and LSM-Trees

LSM minimizes random writes and pushes complexity into reads.

### Memtable

Typically:

-   Skip list
-   Red‑black tree

A skip list is convenient for lock‑free implementations.

------------------------------------------------------------------------

## Constructing and Maintaining SSTables

An SSTable contains:

-   Data blocks
-   Index blocks
-   Bloom filters
-   Footer

The index inside an SSTable is usually:

-   Sparse (every Nth key)
-   Does not store all keys in full

This reduces the memory footprint.
SSTables use **sequential I/O**, which is great for both SSDs and HDDs.

------------------------------------------------------------------------

## Making an LSM-tree out of SSTables

Reads must check:

-   Memtable
-   L0
-   L1
-   L2...

### Compaction trade-off

Leveling:

-   Lower read amplification
-   Higher write amplification

Tiering:

-   Lower write cost
-   Higher read cost

------------------------------------------------------------------------

## Performance Optimizations (LSM)

-   Bloom filters
-   Block cache
-   Write batching
-   Compression per level
-   Rate-limited compaction

LSM behavior is controlled by the compaction policy.

------------------------------------------------------------------------

## Amplification Effects

### Write Amplification (WA)

WA = physical writes / logical writes

High WA:

-   Wears out SSDs
-   Increases latency

### Read Amplification (RA)

RA = physical reads / logical reads

LSM increases RA when:

-   The hierarchy is deep
-   There is a lot of overlap

------------------------------------------------------------------------

## B-Trees

Pages are 4–16 KB.

Tree depth is small (3–4).

Page splits cause random writes.

------------------------------------------------------------------------

## Making B-Trees Reliable

### WAL

1.  Write to the log
2.  Update the page

After a crash → WAL replay.

------------------------------------------------------------------------

## B-Tree Optimizations

-   Prefix compression
-   Fill factor
-   Pointer swizzling
-   Buffer pool tuning

Performance depends on page size and cache policy.

------------------------------------------------------------------------

## Comparing B-Trees and LSM-Trees

  Characteristic         B-tree          LSM-tree
  ---------------------  --------------- ---------------------
  Write throughput       Medium          Very high
  Read latency           Predictable     Can vary
  Write amplification    Lower           Higher
  Read amplification     Lower           Higher

------------------------------------------------------------------------

## Advantages of LSM-Trees

-   High ingest rate
-   Sequential writes
-   Good compression
-   Works well on SSDs

------------------------------------------------------------------------

## Downsides of LSM-Trees

-   Compaction spikes
-   Write stalls
-   Tuning complexity
-   Merge cost on reads

------------------------------------------------------------------------

## Other Indexing Structures

### Storing Values Within the Index

-   Heap table + secondary index
-   Clustered index (data stored inside the B‑tree)

Clustered indexes read faster, but make updates more expensive.

------------------------------------------------------------------------

### Multi-column Indexes

Left-prefix rule:

(A, B, C) supports A and A+B, but not B alone.

------------------------------------------------------------------------

### Full-text search and fuzzy indexes

Inverted index:

    term → posting list

Fuzzy search:

-   n-grams
-   Trigram index
-   Levenshtein distance
-   BK-tree

------------------------------------------------------------------------

### Keeping Everything in Memory

In-memory indexes:

-   Minimal latency
-   Limited by RAM

Durability is achieved via:

-   Append-only log
-   Snapshotting
-   Replication

------------------------------------------------------------------------

## Transaction Processing or Analytics?

OLTP:

-   Point lookups
-   Secondary indexes
-   Row storage

OLAP:

-   Full scans
-   Aggregates
-   Column storage

------------------------------------------------------------------------

## Data Warehousing

-   Separate storage
-   ETL / ELT
-   Column storage
-   Batch ingestion

------------------------------------------------------------------------

## The divergence between OLTP databases and data warehouses

OLTP:

-   ACID
-   Random access

OLAP:

-   Scan speed
-   Compression
-   Vectorized execution

------------------------------------------------------------------------

## Stars and Snowflakes: Schemas for Analytics

Star schema:

-   Facts + dimensions
-   Fast joins

Snowflake:

-   Normalized dimensions
-   More join cost

------------------------------------------------------------------------

## Column-Oriented Storage

-   Stored by column
-   Reads only the needed columns
-   Excellent compression

------------------------------------------------------------------------

## Column Compression

-   Dictionary encoding
-   Run-length encoding
-   Bit-packing
-   Delta encoding

------------------------------------------------------------------------

## Memory Bandwidth and Vectorized Processing

Vectorized execution:

-   Batch processing
-   SIMD
-   Less branch misprediction

In analytics, the bottleneck is often memory bandwidth.

------------------------------------------------------------------------

## Sort Order in Column Storage

Sorting:

-   Improves compression
-   Speeds up filtering
-   Makes zone maps more effective

------------------------------------------------------------------------

## Several Different Sort Orders

Some systems store multiple projections.

Cost:

-   More storage
-   More complex updates

------------------------------------------------------------------------

## Writing to Column-Oriented Storage

-   Delta store (row-oriented)
-   Merge into column segments

Similar to an LSM approach.

------------------------------------------------------------------------

## Aggregation: Data Cubes and Materialized Views

Data cube:

-   Pre-aggregated dimensions

Materialized view:

-   A precomputed query result

Used to speed up OLAP.
