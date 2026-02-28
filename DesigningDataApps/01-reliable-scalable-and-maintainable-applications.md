# 1. Reliable, Scalable, and Maintainable Applications
english | [russian](01-reliable-scalable-and-maintainable-applications.ru.md)

**TL;DR:** Three goals of data‑intensive systems: reliability, scalability, maintainability — and how to discuss them in measurable terms (metrics, SLOs, trade‑offs).

---

## Thinking About Data Systems

- Data‑intensive systems = a combination of databases, caches, queues, and streams; properties matter at the level of the whole chain.
- Design is choosing trade‑offs: latency ↔ cost, consistency ↔ availability, simplicity ↔ feature richness.

## Reliability

- Goal: when faults happen, prevent an externally visible error (failure) or recover quickly (recovery).
- Partial failures are the norm: plan degradation and recovery ahead of time.

### Hardware Faults

- Disks/network/power fail; in distributed systems the probability of failure is higher.
- Typical measures: replication, redundancy, health checks, automatic failover.

### Software Errors

- Bugs, leaks, deadlocks, and wrong assumptions cause cascading failures.
- Helps: limits, isolation (bulkheads), tests, observability, simplification.

### Human Errors

- Config/deploy mistakes are a common cause of incidents.
- Helps: automation, staged rollout, feature flags, least privilege, backups and “undo”.

### How Important Is Reliability?

- It’s an economic decision: the cost of safeguards vs the cost of an incident (data, trust, money).

## Scalability

- The ability to grow with load with predictable cost/complexity.
- It’s important to understand workload parameters and latency “tails”.

### Describing Load

- Describe the workload: RPS, data size, read/write mix, fan‑out, hot keys, burstiness.

### Describing Performance

- Look at latency and throughput; measure latency by percentiles (p95/p99), not averages.
- Queues/concurrency amplify tail latency.

### Approaches for Coping with Load

- Scale up is simpler but limited; scale out requires distributing data/state.
- Practices: caches, async processing, replication, partitioning, backpressure, precompute.

## Maintainability

- Maintainability = operability + simplicity + evolvability; total cost of ownership often dominates.

### Operability: Making Life Easy for Operations

- Observability, alerts, runbooks, fast deploy/rollback, automation of routine operations.

### Simplicity: Managing Complexity

- Reduce “accidental” complexity: consistent interfaces, explicit dependencies, less magic.

### Evolvability: Making Change Easy

- Design for change: compatibility, migrations, gradual releases, extensible contracts.

## Summary

- Start from system properties (reliability/scalability/maintainability), then choose technologies.
