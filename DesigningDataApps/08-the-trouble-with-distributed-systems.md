# 8. The Trouble with Distributed Systems
english | [russian](08-the-trouble-with-distributed-systems.ru.md)

**TL;DR:** Why distribution breaks intuition: partial failures, unpredictable networks, unreliable clocks, and process pauses — and which architectural primitives help.

---

## Faults and Partial Failures

- In distributed systems failures are often partial: one node/link/zone breaks while the rest keeps working, making it hard to tell “failure” from “slowness”.
- A component fault does not have to become a service failure if you have redundancy, graceful degradation, and recovery.
- The core difficulty is uncertainty: “no response” doesn’t prove death, and “a response” doesn’t guarantee freshness.

## Cloud Computing and Supercomputing

- Cloud environments are usually noisier: shared hardware, virtualization, migrations, unpredictable pauses, and variable latency.
- HPC often has more control over the network and scheduling, but with different cost and assumptions.
- Conclusion: algorithms must tolerate jitter, process pauses, and degradation, rather than assuming perfect synchrony.

## Unreliable Networks

- Networks can drop/duplicate/reorder/delay packets; “the network is reliable” is a dangerous assumption.
- Retries and timeouts are unavoidable, but can amplify overload (retry storms) unless you manage backoff and limits.

### Network Faults in Practice

- Short degradations are common: micro-losses, transient partitions, routing issues.
- Failures are often correlated: one incident affects many nodes (shared network/rack/zone).
- Design protective circuits: rate limits, circuit breakers, bounded queues, backpressure.

### Detecting Faults

- Failure detection is usually based on timeouts and heartbeats — it’s statistics, not proof.
- Avoid flapping (frequent status changes): hysteresis, adaptive timeouts, noise suppression.
- Observability helps distinguish “dead” from “stuck/in GC/overloaded”.

### Timeouts and Unbounded Delays

- In the asynchronous model, delays have no upper bound, so a perfect crash detector does not exist.
- Choose timeouts from percentiles and SLOs; too short → false failovers, too long → slow recovery.
- Retries should use exponential backoff and jitter, otherwise they worsen overload.

### Network congestion and queueing

- Queues create tail latency: one overloaded component can slow the entire end‑to‑end request.
- Congestion can be on the network, CPU (encryption/processing), or disk; symptoms look similar but fixes differ.
- Practice: bounded queues, prioritization, load shedding, breaking retry chains.

### Synchronous Versus Asynchronous Networks

- A synchronous network assumes bounded delays; the Internet and most DC systems are closer to the asynchronous model.
- You can buy more predictable networks (QoS/dedicated lines), but process pauses and noise still remain.

### Can we not simply make network delays predictable?

- Making delays fully predictable is hard and expensive; architectures must tolerate variability.
- Even a perfect network doesn’t save you from process pauses, GC, VM migrations, contention, and overload.

## Unreliable Clocks

- Clocks drift (clock skew) and can “jump” during synchronization; you can’t build a strict order from wall clocks.
- Many distributed bugs are misuse of time (timestamps for uniqueness/ordering).

### Monotonic Versus Time-of-Day Clocks

- Time-of-day clocks show calendar time and can move backward/forward; monotonic clocks only move forward and are good for durations.
- Use monotonic clocks for timeouts/latency; time-of-day for user-visible timestamps.

### Clock Synchronization and Accuracy

- Synchronization (NTP/PTP) gives an approximation; accuracy is limited by networks and oscillator stability.
- The right model is time = value ± error (confidence interval), not an exact point.

### Relying on Synchronized Clocks

- Timestamp ordering breaks under skew; if you need order, use logical clocks or consensus/total logs.
- Global snapshots by time are possible only with strong sync guarantees and careful protocols.

## Process Pauses

- A process can “freeze” due to GC, page faults, CPU starvation, VM suspend — it looks like a network fault.
- Pauses break heartbeats and trigger false failovers; you need tuning and isolation for latency‑critical components.

## Knowledge, Truth, and Lies

- Nodes have incomplete knowledge; “truth” is often defined by a majority (quorum/majority). Examples: Raft, Paxos, ZAB.
- Leadership and locks must be protected from “zombie leaders” — fencing tokens matter.

### The Truth Is Defined by the Majority

- Quorums help avoid split brain and coordinate decisions under partial failures.
- But quorums assume correct membership and reasonable timeouts.

### The leader and the lock

- A leader often holds the right to write/coordinate (log/lock). An old leader can “come back” and write — dangerous.
- You need a way to cut off old leaders (epoch/term + fencing).

### Fencing tokens

- A fencing token is a monotonically increasing ownership number; the resource accepts commands only with the highest token.
- It protects against a stale leader even under pauses and partitions.

### Byzantine Faults

- Byzantine faults are arbitrary/hostile behavior. In typical DC systems we assume crash/omission faults.
- Still, “soft lies” happen: corrupted data, bugs, bad configs — integrity checks help.

## System Model and Reality

- Formal models help prove properties; reality violates assumptions (asynchrony, pauses, correlated failures).
- Correctness is split into safety (nothing bad happens) and liveness (something good eventually happens); you often sacrifice liveness for safety.

## Summary

- Distribution is hard due to partial failures, unpredictable delays, unreliable clocks, and process pauses.
- Mitigations: timeouts+backoff, quorums, fencing tokens, logical ordering, and clear safety/liveness criteria.
