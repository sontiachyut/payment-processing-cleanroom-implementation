# Hyperswitch reference analysis

**Inspection baseline:** 2026-10-21. **Pinned repository:** [juspay/hyperswitch at b0bc2ca6a909a58adefa861285f07f985af10f66](https://github.com/juspay/hyperswitch/tree/b0bc2ca6a909a58adefa861285f07f985af10f66).

Method: resolve public repository commit, inspect its complete file tree, fetch selected source/docs/tests at that SHA, and read relevant functions/assertions. No Hyperswitch services, tests, containers or payment calls were run. Static inspection demonstrates code paths and assertions, not passing results or universal deployed guarantees. Feature-gated v1/v2 paths coexist; findings apply only to inspected paths/configurations. No Rust is copied or translated.

## Overview and source inventory

Hyperswitch is a much broader payment infrastructure project than our intended learning system. Its architecture document describes Router and Scheduler roles, PostgreSQL, Redis, locker and monitoring. We borrow the questions its boundaries answer, not its entire topology. The authoritative [architecture document at the inspected SHA](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/docs/architecture.md) is descriptive evidence; source inspection below narrows what was actually verified.

| ID | Inspected pinned source | Verified reading / relevance |
| --- | --- | --- |
| H1 | [Connector guide](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/add_connector.md) | Describes connector common/integration traits, flow-specific transformations, secret wrappers and test integration |
| H2 | [Payment confirm operation](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/operations/payment_confirm.rs) | Operation lifecycle around validation, tracker retrieval/update and flow-specific domain work; not one generic status assignment |
| H3 | [Payment capture operation](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/operations/payment_capture.rs) | Reads intent/active attempt, validates capture state/method and capturable amount; contains multiple-capture handling |
| H4 | [Payment cancel operation](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/operations/payment_cancel.rs) | Separate cancellation operation path rather than treating every reversal as refund |
| H5 | [Refund orchestration](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/refunds.rs) | Refund validation, gateway trigger, integrity checking and separate refund reversal paths; amount/state policy matters |
| H6 | [Stripe connector](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/hyperswitch_connectors/src/connectors/stripe.rs) | Flow-specific integration implementations and provider HTTP request/response boundary |
| H7 | [Stripe transformations](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/hyperswitch_connectors/src/connectors/stripe/transformers.rs) | Provider payload/status transformations; normalization has source-specific semantics |
| H8 | [Routing](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/routing.rs) | Single, priority, volume-split and advanced selection forms and substantial feature-specific routing machinery |
| H9 | [Retry orchestration](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/retry.rs) | GSM decision-driven retries, retry count/connector exhaustion and feature-specific retry paths; not retry every error |
| H10 | [Payment synchronization job](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/workflows/payment_sync.rs) | v1 path invokes PSync/PaymentStatus, finishes known terminal statuses or schedules retry; v2 support differs |
| H11 | [PSync flow](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/flows/psync_flow.rs) | Flow-specific synchronization boundary rather than a new authorization |
| H12 | [Refund recovery job](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/workflows/refund_router.rs) | Separate background refund execution/synchronization responsibilities |
| H13 | [Scheduler producer](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/scheduler/src/producer.rs) and [consumer](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/scheduler/src/consumer.rs) | Scheduling and execution are separate roles; process records/queue state need recovery semantics |
| H14 | [Incoming webhooks](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/webhooks/incoming.rs) | Source verification, flow dispatch and locking integration exist; this is not evidence every provider shares one replay policy |
| H15 | [API locking](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/api_locking.rs) | Explicit Redis-backed locking actions/key scoping; this project instead starts with PostgreSQL aggregate coordination |
| H16 | [Outgoing webhook retry](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/workflows/outgoing_webhook_retry.rs) | Notification retry is distinct from repeating a financial operation |
| H17 | [Payment attempt model](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/hyperswitch_domain_models/src/payments/payment_attempt.rs) | Intent/attempt distinction and attempt-specific provider/state metadata |

## Tests inspected

[Stripe connector tests](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/tests/connectors/stripe.rs) include `should_only_authorize_payment`, `should_capture_already_authorized_payment`, `should_partially_capture_already_authorized_payment`, `should_sync_authorized_payment`, `should_sync_payment`, `should_void_already_authorized_payment` and negative payment-input cases. The inspected authorization assertion expects Authorized; the make-payment path expects Charged. Connector tests may require provider credentials/setup; reading them is not executing them.

[Routing tests](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/routing/tests.rs) and [preferred-connector tests](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/crates/router/src/core/payments/routing/tests/preferred_connectors.rs) exercise routing preference updates/lookups and missing-preference behavior. They do not establish our PS concurrency, outbox or crash guarantees.

No direct proof of our intended same-key PostgreSQL arbitration, aggregate refund reservation, transactional outbox atomicity or stale-worker fencing was established from these inspected tests. Those are original design requirements with new future tests. Avoid attributing uninspected guarantees to Hyperswitch.

## Capability-by-capability design transfer

| Capability | Actual evidence and engineering issue | Proposed Java equivalent, alternatives and simplification | Failure/test lesson |
| --- | --- | --- | --- |
| Authorization/lifecycle | H2/H17 and connector tests separate logical state/attempt flow | Pure policy plus application orchestration; simpler operation state machine instead of translating Rust generic framework | PS-01/02; assert persisted identities, not only HTTP success |
| Adapters/normalization | H1/H6/H7 separate provider representations and flow contracts | Typed Java processor port + per-flow DTO mappings; composition rather than source translation | Mapping wrong status/currency can corrupt truth; shared adapter contract tests |
| Routing | H8 supports several algorithms and feature-specific machinery | Capability/currency eligibility and deterministic priority first; reject dynamic DSL/ML routing until measured need | PS-03; record choice/rule version and no uncertain fallback |
| Capture | H3 checks method/state/capturable amount, including optional complex cases | Full capture first; row lock, reservation, active-operation guard; defer overcapture/multiple capture | PS-04; original real-DB race tests required |
| Refund | H5 validates/refines refund operations and integrity | Refund reservation and linked capture; full then partial refunds | PS-05; include pending/unknown amounts, not merely confirmed totals |
| Cancel/reversal | H4/H5 distinguish operation families; connector semantics vary | Explicit cancel/refund/reversal capabilities with original linkage | AC-07; never infer post-capture refund semantics from a generic void |
| Payment synchronization | H10/H11 re-read provider status through PSync, complete/reschedule jobs | Durable inquiry-first jobs, normalized unknown/final observations | PS-02/10; response loss and crash injection, not repeat authorization |
| Timeout uncertainty | Sync/attempt paths show distinct observation/recovery concerns; no universal all-provider timeout guarantee established | UNKNOWN and reservation retention; inquiry or documented stable-key replay | Simulated effect-then-response-loss verifies no duplicate external effect |
| Retry eligibility | H9 uses configuration/GSM decisions, counters and connector exhaustion | Small explicit error matrix and one retry owner; do not reproduce GSM or token/PAN retry paths | PS-06; transport failure class alone cannot justify cross-processor retry |
| Outage/fallback | H8/H9 separate selection/retry policy, with many conditional paths | Health-aware selection for provably undispatched new operations only | PS-03; deliberately keep unknown original attempt pinned |
| Scheduling/recovery | H12/H13 separate producer/executor jobs and state | PostgreSQL job table, short claims/leases/fences; no initial Redis requirement | PS-10; pause old worker, reclaim, reject stale completion |
| Events/notifications | H14/H16 show inbound verification and outgoing delivery retry roles | Inbox/outbox, immutable event IDs, atomic local effects; Kafka at milestone 7 | PS-07/08/09; our outbox design is not claimed verified Hyperswitch implementation |
| Observability | Architecture document and inspected retry/sync paths contain logging/metrics | Structured IDs/error classes, Micrometer/OTel; no raw payment payloads | PS-12; correlate accepted, completed, unknown, rejected separately |

## Settlement/reconciliation evidence limit

The public [e-commerce reconciliation example](https://docs.hyperswitch.io/other-features/payments-modules/reconciliation-product/example-e-commerce) describes order/payment/bank evidence, transformations, expectations, matching and exceptions. It is product documentation, not proof that this inspected open-source Router commit contains that complete reconciliation engine. No corresponding complete reconciler implementation/test suite was verified here.

Our original MVP therefore uses versioned synthetic settlement batches, exact operation/reference/currency matching, explicit gross/fee/net signs, complete windows and grace periods. Three-way bank/payout reconciliation is a future extension, not a claim copied from a product page. PS-11 will have its own fixtures and fault tests.

## Adopt, reject and licensing

Adopt intent/attempt separation, normalized connectors, independent sync jobs, explicit retry policy, recorded routing and inspectable metrics. Reject immediate Redis/Kafka/service topology replication, connector-specific data models as our domain, direct Rust→Java translation, universal safety claims inferred from one connector, and implementing every payment method at once.

The pinned root [license](https://github.com/juspay/hyperswitch/blob/b0bc2ca6a909a58adefa861285f07f985af10f66/LICENSE) is Apache-2.0. This repository contains original documentation and links, no copied implementation. Any future code/assets/dependency reuse requires checking the actual file's terms, notices and provenance at that time; a public repository is not evidence that all external services/data are unrestricted.

Primary limitation: this was selected static inspection, not a security audit, production validation or exhaustive review of all feature gates. [PS scenario definitions](production-scenarios.md) cite the relevant H identifiers while labeling our strategies and tests as proposed.
