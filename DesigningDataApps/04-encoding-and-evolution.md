# 4. Encoding and Evolution
english | [russian](04-encoding-and-evolution.ru.md)

**TL;DR:** How to choose serialization formats and manage schema evolution (backward/forward compatibility) when data flows through databases, RPC/REST, and messages.

---

## Formats for Encoding Data

- The format affects size, speed, security, and compatibility; the contract (schema) matters.

## Language-Specific Formats

- Tightly coupled to a language/runtime version; often unsafe and poorly interoperable across languages.

## JSON, XML, and Binary Variants

- Text formats are convenient but “fat” and ambiguous about types; binary formats save space/CPU.

### Binary encoding

- Compactness and speed at the cost of readability; you must agree on types and optional fields.

## Thrift and Protocol Buffers

- Schema‑first, code generation, binary serialization; compatibility via field IDs.

### Field tags and schema evolution

- Don’t reuse a field ID for a different meaning; add new fields as optional/with defaults.

### Datatypes and schema evolution

- Type changes are risky; safer to add a new field and migrate gradually.

## Avro

- Writer/reader schemas + resolution rules; convenient for events and data pipelines.

### The writers schema and the readers schema

- The writer encodes using its schema, the reader decodes using its schema; mismatches are reconciled by rules.

### Schema evolution rules

- Backward compatibility is often achieved by adding a field with a default; removal is OK if the reader doesn’t require it.

### But what is the writers schema?

- You need a way to deliver the schema: a schema registry, a message header, or an out‑of‑band channel.

### Dynamically generated schemas

- Autogeneration simplifies ingestion, but requires disciplined versioning and naming.

### Code generation and dynamically typed languages

- In static languages, type generation matters; in dynamic ones, validation and contract tests matter.

## The Merits of Schemas

- Schemas provide validation, documentation, and controlled evolution; they require governance.

## Modes of Dataflow

- Data “flows” through databases, services, and messages; each needs its own compatibility strategy.

### Dataflow Through Databases

- Different app versions read/write the same DB → you must read both old and new, or migrate.

#### Different values written at different times

- Old records stay in the old format; the reader must be compatible over time.

Working strategies:
- Lazy migration — transform a record on read
- Eager migration — bulk migration via ALTER + rewriting all data
- Support multiple formats during the transition

#### Archival storage

- For archives you want self‑describing, stable formats plus metadata. Formats like Avro, Parquet, or JSON-with-schema are usually better than ad‑hoc binary formats.

### Dataflow Through Services: REST and RPC

- REST is simpler and “coarser”; RPC is faster and more typed, but networks mean timeouts/retries/partial failures.

#### Web services

- HTTP + a contract (OpenAPI/Swagger/JSON Schema) + API versioning + error codes + optional/required fields.

#### The problems with remote procedure calls (RPCs)

- The local‑call illusion hides network problems: retries, duplicates, unpredictable latency.

#### Current directions for RPC

- Modern RPC (e.g., gRPC‑class) = strict schemas + explicit timeout/retry policies.

#### Data encoding and evolution for RPC

- Evolve compatibly: optional fields, new endpoints, clear error handling, idempotency.

### Message-Passing Dataflow

- Asynchrony reduces coupling and buffers peaks; delivery semantics matter.

#### Message brokers

- A broker stores/delivers messages; pick at‑least‑once semantics and make handlers idempotent.

#### Distributed actor frameworks

In the actor model:
- each actor has its own state
- interaction happens only via messages
- there is no shared memory

Actors exchange messages; you must consider ordering, redelivery, and state.
Actors simplify local logic, but they do not remove the fundamental problems of distributed systems.

## Summary

- Schema management and compatibility are the foundation of safe change in distributed systems.
