# Just-in-time Java/payments learning

Use one small approved assignment at a time. Explain the problem, inspect the referenced source/test, predict failure, implement within scope, run checks, then defend the observed result. Do not require an enormous theory syllabus before coding.

| Before milestone | Concepts to learn | Exercise / source |
| --- | --- | --- |
| M0 | Intent/attempt/operation, original design rationale, financial uncertainty | Walk PS-02 with lost response; [design rationale](design-rationale.md) |
| M1 | Java values/interfaces/exceptions, DI/MVC, DTOs, Maven/BOM, JPA session/flush/transaction, constraints | Explain restart/rollback and entity versus response; [Java](https://dev.java/learn/), [Spring transactions](https://docs.spring.io/spring-framework/reference/data-access/transaction.html) |
| M2 | Minor units/overflow, state machine and 202 acceptance | Draw confirmed state versus operation state after timeout; [domain model](domain-model.md), [Stripe lifecycle](https://docs.stripe.com/payments/paymentintents/lifecycle) |
| M3 | HTTP replay, semantic hashes, unique arbitration, transaction abort, business identity | Same key/changed amount table and barrier race; [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) |
| M4 | Ports/adapters, flow contracts and source normalization, capability-driven routing | Compare A/B status mappings and capability declarations; [processor design](processor-integrations.md) |
| M5 | Deadlines, uncertain effects, stable keys, inquiry, leases/fences, retry/backoff/breaker/bulkhead | Enumerate provider/local crash windows; [Stripe error handling](https://docs.stripe.com/error-low-level) |
| M6 | Row locks, optimistic versioning, Read Committed versus Serializable, money reservations/deadlock | Competing 30.00 refunds against 40.00 available; [PostgreSQL isolation](https://www.postgresql.org/docs/current/transaction-iso.html), [locks](https://www.postgresql.org/docs/current/explicit-locking.html) |
| M7 | Dual writes/outbox, at-least-once, partition order, offsets/rebalance and dedup | Crash after consumer DB commit/before offset; [outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html), [Kafka](https://kafka.apache.org/43/design/design/) |
| M8 | Raw signature verification, event versus effect IDs, stale observation and worker ownership | Reverse webhook order; pause old owner before reclaim; PS-07/PS-10 |
| M9 | Gross/fee/net, capture/settlement/payout, complete window/timezone/grace | Explain absent row in incomplete file; [settlement model](domain-model.md#settlement-normalization) |
| M10 | Arrival/service rates, p95/p99, saturation, query plans/indexes, pools and telemetry | Predict overload and measure one bottleneck; [EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html), [k6 scenarios](https://grafana.com/docs/k6/latest/using-k6/scenarios/) |
| M11 | Sandbox limitations, versions/credentials, cloud operational/cost boundaries | Separate actual Stripe observations from local injected faults; [Stripe testing](https://docs.stripe.com/testing) |
| M12 | User authority versus financial authorization, cross-system saga, independent contracts | Explain AC-07 compensation from actual payment/dispense state; [shared design](commerce-integration.md) |

First future implementation assignment: approve M1, verify compatible patches for the selected Java 21/Spring Boot 4.x/JUnit 6 baseline, then build one persisted nonfinancial resource and explain transaction/restart behavior. Do not start the full payment lifecycle or cloud setup in that session unless specifically approved.

Defer reactive programming, Kubernetes, multiple services, JVM micro-optimization, Redis coordination and multi-region writes until a concrete requirement exists. Learning deep correctness is more valuable here than a large technology inventory.
