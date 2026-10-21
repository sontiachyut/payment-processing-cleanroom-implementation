# Proposed payment validation strategy

No automated test implementation, load run or measurement exists. [PS-01–PS-12](production-scenarios.md) define mandatory behavioral assertions and faults; this document defines the common verification method.

## Layers

Pure policy tests cover money bounds, lifecycle transitions, reservation arithmetic, retry classification and routing eligibility. API tests cover validation/auth/tenant-safe errors and stable resource semantics. PostgreSQL/Testcontainers integration covers actual constraints, aborted transactions, locks, versions and restart persistence; in-memory substitutes cannot prove these semantics. Adapter contract tests use independent stateful simulators and source amount/status integrity. Kafka/Testcontainers checks publication/consumption/rebalance with real DB effects. Webhook tests verify raw-body signatures, durable inbox and observation merge. Reconciliation fixtures cover exact IDs, signs, completeness and corrections.

Use Spring Boot-managed JUnit 6/Jupiter, Mockito only at genuine boundaries and Testcontainers for real dependency semantics. Verify and pin compatible Spring Boot 4.x and test-library patches in milestone 1; avoid independently overriding managed versions without a demonstrated reason.

## Race and crash protocol

Barriers/latches synchronize competing readers/writers; arbitrary sleep is not proof of a race. Separate DB connections and two processes are required for multi-instance claims. Record seed and command/operation identities. Inspect durable counts, captured/refunded/reserved totals, provider effect count, attempt/observation history, audit/event identity and unknown age—not just HTTP response.

Fault points: before/after acceptance commit; before/after dispatch marker; before/after provider effect; response lost; before/after result commit; outbox publish/marker gap; consumer effect/offset gap; webhook receipt/application gap; import chunk/window close; claim expiry with old owner paused; DB/ownership connection lost. Each accepted operation must be final or visibly unresolved, never silently lost. Provider simulator state survives application restart; tests reset it only deliberately after evidence collection.

## Load and performance

Start low, then exploratory 100/500/1,000 offered RPS only if correct and safe on documented hardware. k6 open arrival-rate and closed concurrency scenarios reveal different effects; include unique purchases, replay storms, hot payment/merchant, competing refunds, provider tails, queue/broker outage and pool saturation. Stop at explicit CPU/memory/disk/queue/error bounds rather than crashing a working laptop.

Publish commit/runtime/dependency/container versions, hardware, DB settings/dataset/indexes, simulator latency assumptions, offered load/duration/warmup, seed and worker/pool limits. Report accepted/rejected/success/known-failure/unknown separately, p50/p95/p99 acceptance and completion latency, queue/lock/pool waits, resource use and event lag. After drain, reconcile every accepted action to source effects and invariant totals. Change one bottleneck variable and rerun comparable load; do not infer production capacity from one local run.

High volume uses only simulators, never Stripe sandbox or LLM APIs. Stripe test behavior is a separate low-volume integration lane. No production latency distribution or real user volume is assumed.

## Cross-project and completion

Shared future fixtures in [commerce contract](commerce-integration.md) map every PS to commerce assertions. Contract-negative/security tests verify amount/hash/scope mismatch, replay and unknown schema. Java must work with commerce absent. Cloud/deployment/CI tests are later approved work, not configured now.

Milestone evidence includes actual command/result, seed/config, reviewed failure and remaining limitation. Reading Hyperswitch tests does not make ours pass. Generated tests are unvalidated until executed against their intended dependencies; no result is fabricated.
