# Proposed payment API and event contracts

Documentation only; no OpenAPI, schema implementation, endpoint or event producer exists. APIs use `/api/v1`; tenant/merchant comes from validated service identity. Financial POST commands require `Idempotency-Key`. Keys contain no PII and are scoped to merchant/action/payment where applicable.

| Method and resource | Input | Acceptance/read contract |
| --- | --- | --- |
| POST /payments | business_reference, amount_minor, currency, synthetic payment method reference, optional commerce binding | 202 payment_id, operation_id, execution_state, Location and inquiry links |
| POST /payments/{id}/captures | amount_minor (full capture initially), business action reference | 202 stable operation; state/active-operation conflict is explicit |
| POST /payments/{id}/cancellations | reason enum | 202 cancellation of confirmed uncaptured authorization; capability-specific |
| POST /payments/{id}/refunds | positive amount_minor, reason, source capture identity | 202 only after amount reservation; pending/unknown reservations included |
| POST /payments/{id}/reversals | original_operation_id, supported reason | Capability-gated reversal protocol; not generic substitute for capture refund |
| GET /payments/{id} | optional minimum_version | Confirmed totals/state, active/unknown operation, aggregate version, last event sequence, observation times |
| GET /operations/{id} | none | Accepted/dispatch/unknown/final result, normalized reason and recovery status |
| GET /payments/{id}/history | bounded cursor/limit | Attempts/transitions with source and sequence; no raw secret payloads |
| GET /payments | reference/state/bounded interval, cursor, limit ≤100 | Tenant-filtered summaries and stable keyset cursor |
| POST /webhooks/{provider} | bounded raw request and signature headers | Verify first, persist inbox before 2xx; async application of observations |
| POST /settlement-batches | versioned synthetic file and batch/window identity | 202 staged import, validation/completeness state; importer permission |
| GET /settlement-batches/{id} | none | Validation/matching counts and exception refs |
| GET /reconciliation-exceptions | filters/cursor | Deterministic exception occurrences with exact evidence |

202 means durable acceptance, never payment success. Replay returns original acceptance identity and current-resource links. Invalid input 400; unauthenticated/forbidden 401/403; tenant-safe not found 404; state/in-flight/key-payload conflict 409; oversized input 413; capacity 429; durable acceptance unavailable 503. Safe error envelope: code, message, request/correlation ID, retryability and related operation ref. An HTTP 500/timeout does not prove a provider declined.

Proposed retention is dataset lifetime initially. A future purge policy needs a documented replay horizon, tombstones and business uniqueness guards. Provider key expiration requires inquiry/escalation; do not generate a new financial identity merely because a cache expired.

## Event envelope

Event ID, envelope_version, event_type, payload_version, producer, tenant/merchant, payment_id, operation_id, aggregate_version, event_sequence, occurred_at, recorded_at, correlation_id, causation_id, optional validated trace context and minimal typed data. Each payment's public fact sequence is assigned atomically with the outbox; multiple events in one aggregate version have distinct sequences. Timestamps are not reliable ordering identities.

Facts: operation.accepted/unknown/failed/resolved; authorization.confirmed/declined; capture.confirmed; cancellation.confirmed; refund.confirmed; reconciliation.exception.detected/updated. Event types never imply financial success from local acceptance. Topics `payments.events.v1` and `payments.exceptions.v1` are proposed; partition key is merchant/payment identity.

Consumers deduplicate source/event identity with local effects in one transaction, then commit offsets. Distinct events may share an operation, so event-level replay and effect-level dedup are separate. Gaps/stale versions lead to authoritative inquiry; source facts may arrive through webhook, polling and synchronous response. Unknown major schema is quarantined with a controlled replay procedure.

## Commerce client

The shared [commerce contract](commerce-integration.md) defines user/session/order/authorization binding and the full end-to-end flow. Java verifies client merchant scope, amount/currency, immutable business reference and trusted attestation shape/digest; commerce verifies user spending policy and inventory. Do not include mutable product descriptions as trusted payment authorization. No cross-database join is needed to execute an independently authenticated payment.
