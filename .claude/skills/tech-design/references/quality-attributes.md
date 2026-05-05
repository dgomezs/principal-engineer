# Quality Attributes

Used by the principal-engineer agent as the interview agenda for design-committing requests.
Used by the tech-design skill to drive conditional sections in the output document.

Address only attributes relevant to the design. For each that applies, answer the listed questions — don't just name the attribute.

---

## C4 Diagram Levels

Use C4-native Mermaid syntax (`C4Context`, `C4Container`, `C4Component`) for Levels 1–3, `classDiagram` for Level 4.

| Level | Use when |
|---|---|
| 1 — System Context | How X fits in the broader landscape |
| 2 — Container | Main services, DBs, brokers, UIs |
| 3 — Component | Internal structure of one service |
| 4 — Abstraction | Key interfaces and their relationships |

---

## Domain Decomposition

Before designing components, establish:

- **Bounded contexts** — What are the domain boundaries? What is the ubiquitous language within each?
- **Context map** — How do contexts relate? (shared kernel, customer/supplier, anti-corruption layer, open host?)
- **Aggregates** — What are the consistency boundaries within each context?
- **Domain events** — What state changes need to cross context boundaries?

---

## Reliability

- What is the blast radius if this component fails?
- Are there unbounded waits? Every network call needs a timeout.
- Are retries safe? Idempotency required before retrying mutations.
- What is the circuit-breaker / bulkhead strategy to prevent cascade failures?
- What is the backpressure mechanism if a consumer is slower than a producer?
- How are poison messages handled (dead-letter, alert, skip with audit)?
- What is RPO (acceptable data loss) and RTO (acceptable downtime)?
- Is there a fallback if a dependency is unavailable?

## Performance

- What is the latency budget? (p50, p99 targets)
- What is the throughput requirement (requests/sec, events/sec)?
- Where is the hot path? What is the cost of each operation on it?
- What can be cached, and what is the invalidation strategy?
- Is there any O(n) work that becomes O(n²) under load?

## Scalability

- What is the current load? What does 10x look like?
- Where are the bottlenecks — CPU, I/O, memory, network, lock contention?
- Can the component scale horizontally? What shared state prevents it?
- Are there partitioning or sharding considerations?

## Security

- Where are the trust boundaries?
- Authentication (who are you?) and authorization (what can you do?) — handled separately and at which layer?
- What data is classified PII, secret, or sensitive? Where does it flow and where is it stored?
- What is encrypted in transit and at rest?
- What is the attack surface at each trust boundary crossing? (invoke `threat-modeling` if non-trivial)
- Are secrets injected via environment/vault — never hardcoded or logged?
- Is input validated at the system boundary, before it reaches domain logic?

## Storage & Data

- What is the consistency model required (strong, eventual, read-your-writes)?
- What are the read and write patterns? Are indexes designed for those patterns?
- What is the data retention policy and how is deletion enforced?
- How does schema evolution happen without downtime (two-phase migration, expand/contract)?
- Are there hot partitions or large-table concerns?
- What is the backup strategy and has restore been tested?

## Observability

- How do we know the system is healthy (SLI/SLO)?
- What is traced end-to-end (distributed trace)?
- What metrics are emitted and what alerts fire on them?
- Are errors logged with enough context to diagnose without a debugger?
- Is there a runbook for the top failure modes?

## Evolvability

- What is tightly coupled? What does "change X" force you to also change?
- Is there a stable interface consumers can depend on, separate from internals?
- What is the deprecation and versioning strategy for APIs?
- How hard is it to add a new consumer, a new field, a new state?
