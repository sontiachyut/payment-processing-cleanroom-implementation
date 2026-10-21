# Proposed modular architecture

All modules and flows are planned. [README](../README.md) contains the context diagram. The design intentionally simplifies Hyperswitch instead of reproducing its distributed deployment or translating Rust traits.

| Module | Responsibilities and owned records | Boundary |
| --- | --- | --- |
| payment-api | Authenticated transport DTOs, validation, idempotency headers, inquiry/search | Never serialize JPA entities or let callers set status |
| payment-domain | Confirmed payment facts, operation policy, money invariants and transition decisions | No HTTP, Kafka or model dependency |
| command-orchestration | Durable acceptance, reservations, atomic state/audit/outbox, conflict handling | Short DB transactions only |
| processor-integration | Capability ports, normalized requests/results, provider keys/inquiry | Never directly mutate payment state |
| routing | Eligibility, priority, health gates, selection version/reason | No failover of possibly executed commands |
| recovery | Durable work, leases/fences, retry schedules and synchronization | A lease is not proof an old provider call stopped |
| event-processing | Outbox relay, webhook inbox, duplicate-safe consumer effects, schema policy | At-least-once delivery, no blanket exactly-once claim |
| reconciliation | Staged settlement import, exact matching, signed fee validation and exceptions | Never change captured state to failed based on absent settlement |
| operations | Scoped history, evidence APIs, metrics, traces and immutable logical audit | Read permission does not grant financial actions |

## Transaction boundaries

**T1 command acceptance:** authenticate/validate; resolve idempotency winner; lock payment; recheck state/remaining money; create operation and reserve amount; write acceptance audit/event intent; commit. Return 202 with stable resources.

**External phase:** worker claims operation in a short transaction, records stable provider identity and dispatch marker, commits, then calls processor with no DB locks held.

**T2 outcome application:** check ownership fence and legal transition; apply each confirmed effect once; release/retain reservation appropriately; persist attempt observation, audit, and outbox together. Timeout creates UNKNOWN and inquiry work, not a false decline.

**T3 publication/consumption:** publish outbox then mark; duplicates are possible. Consumer inbox plus local effect commit together before broker offset. Webhooks follow the same durable-ingress principle.

No DB transaction spans processor, Kafka, commerce, model or simulated dispensing. Payment commands remain usable during a bounded broker outage; backlog capacity/admission policy prevents disk exhaustion.

## Coordination and scaling

Use Read Committed, short payment-row pessimistic locks, unique guards and reservations for financial decisions; version checks/fences protect stale completion. Single active financial operation per payment initially. API processes can be stateless, but adding workers requires two-process correctness tests, aggregate contention measurement and shared connection/provider budgets. Redis is not canonical idempotency or financial locking.

Begin one deployable with business packages and one DB. A worker process from the same artifact is an execution boundary, not automatically a microservice. Kafka arrives at milestone 7 for external/replayable consumers. Outbox polling precedes CDC. No Kubernetes, service mesh, distributed cache or extra database without a demonstrated requirement.

## Stack adoption and alternatives

Java 21/Maven/Spring Web introduce conventional request, DI and transaction concepts. JPA handles routine mappings; narrow SQL is appropriate for row/work claiming. Flyway arrives with persistence. Testcontainers verifies actual PostgreSQL/Kafka semantics; Mockito isolates declared adapters rather than replacing transaction tests. Resilience4j adds per-provider capacity/breakers only after classified retry behavior is explicit. k6 arrives at milestone 10; Micrometer/structured logs begin earlier, OTel/Prometheus/Grafana consolidate operational visibility.

Docker Compose later provides local dependency reproducibility. GitHub Actions follows reproducible local tests. ECS/RDS deployment is milestone 11 work requiring approval and cost/security review, with no infrastructure files in milestone 0. The JUnit 5 versus supported Boot generation decision is documented in [ADR 006](adr/006-runtime-and-infrastructure.md).

## Operational boundaries

Confirmed payment snapshots come from primary authoritative storage initially. Replicas/projections need explicit freshness watermarks before use. Separate payment writer, webhook ingestor, settlement importer and evidence reader identities. Trust commerce as an authenticated client subject to merchant scope, amount/currency validation and its bound command identity; do not trust model-generated descriptions as user consent. [Integration contract](commerce-integration.md).

Health distinguishes DB acceptance capability from an individual provider outage. Alerts focus on old unknown operations, due-work age, pool/lock pressure, outbox age, consumer lag and reconciliation completeness. IDs belong in logs/traces, not high-cardinality metric labels. Logs omit tokens, raw provider payloads, headers and customer data.
