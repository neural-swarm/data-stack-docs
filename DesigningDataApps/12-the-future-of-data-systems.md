# 12. The Future of Data Systems
english | [russian](12-the-future-of-data-systems.ru.md)

**TL;DR:** How to connect tools via event streams and derived state; why batch and stream are converging; what database unbundling enables; and why end-to-end correctness and auditability become central.

---

## Data Integration

- Data integration is connecting specialized tools through change streams and derived representations (derived data).
- The key question: how to guarantee derived state is consistent with the source of truth with bounded lag and correctness.

## Combining Specialized Tools by Deriving Data

- Pattern: OLTP/source → log/CDC → stream → search index/cache/analytics warehouse.
- Instead of distributed transactions across systems, people often choose event-driven coordination: idempotent consumers + compensations.

### Derived data versus distributed transactions

- 2PC across different systems is expensive and operationally complex; derived data decouples systems but accepts eventual consistency.
- What’s critical: correct event contracts, schema versioning, and recovery/reconciliation procedures.

### The limits of total ordering

- A global total order is expensive (consensus) and can become a bottleneck.
- Often per-key ordering (by aggregate) or causal order is enough; this scales better.

## Batch and Stream Processing

- The boundary is blurring: many systems want both replay (batch) and live processing (stream) with one logic.
- You must be able to reprocess history when application logic changes: keep history and maintain schema compatibility.

### The lambda architecture

- Lambda = batch accuracy + speed layer + serving; the problem is two implementations of logic and the risk of divergence.
- Trend: unification — one engine/model for batch and stream (shared windows/time/state).

## Unbundling Databases

- Unbundling = split a “monolithic DB” into components: storage, compute, log, index, cache.
- Pros: independent scaling and evolution; cons: you must manage integration and correctness well.

### The meta-database of everything

- With unbundling, metadata becomes critical: schemas, lineage, quality, access, owners — otherwise the ecosystem is unmanageable.
- A meta-layer helps: find sources, understand pipeline dependencies, and audit.

## Designing Applications Around Dataflow

- Design applications as derivation functions over event streams: commands → events → state updates → reads from materialized views.
- This makes behavior replayable and auditable, but requires disciplined idempotency and deduplication.

## Observing Derived State

- Users read derived state (cache/index/view), so you must manage staleness and update delivery (push/pull).
- Offline clients add conflicts and require merge/CRDT-style approaches.

## Aiming for Correctness

- The end-to-end argument: guarantees must hold across the whole chain, not be “assumed” from individual components.
- Practice: operation IDs, duplicate suppression, reconciliation jobs, auditability (immutable logs).

## Enforcing Constraints

- Global constraints (uniqueness, limits) often require consensus/coordination.
- Coordination avoidance is possible if you restrict the operation model or use CRDTs/local invariants.

## Trust, but Verify

- Complex systems fail due to bugs and rare failure modes; you need a culture of verification: invariants, reconciliation jobs, audits.
- Auditability matters: answer “who/when/why” changed state.

## Doing the Right Thing

- Technical decisions have social consequences: privacy, surveillance, discrimination, feedback loops.
- You need: data minimization, transparency, access control, and accountability for how analytics is used.

## Summary

- The future is dataflow and derived data, batch/stream unification, and end-to-end correctness + auditability.
- In parallel, responsibility grows: privacy and fairness become engineering requirements.

---

## Terms

- **Derived data:** derived state (indexes/aggregates/caches) from a source of truth
- **Dataflow:** data moving through transformation stages
- **Staleness:** how out of date derived state is relative to the source
- **Replay/Reprocessing:** replaying history to recompute
- **Lambda architecture:** batch+speed+serving (two code paths)
- **Unbundling:** splitting DB functions into separate components
- **Metadata/Lineage:** metadata and data provenance in pipelines
- **End-to-end argument:** guarantees must be ensured at the ends of the chain
- **Operation ID:** a unique operation identifier for idempotency
- **Duplicate suppression:** suppress duplicates by operation/event ID
- **Reconciliation:** reconcile derived data against the source of truth
- **Coordination avoidance:** designs without global coordination (local invariants/CRDTs)
- **Auditability:** ability to reconstruct history and explain changes
- **Data minimization:** collect the minimum data needed for the goal
