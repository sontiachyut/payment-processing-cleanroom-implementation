# Proposed payment design rationale

The project is an independent clean-room implementation. This document explains its own requirements, decisions, alternatives and validation boundaries. Application development has not started; no external example or architecture document proves this project's correctness.

## Financial identity and state

A payment represents the business transaction; an operation represents one financial action; a processor attempt represents transport work. Keeping these identities separate allows retries without redefining the intended charge. Confirmed financial facts are separate from accepted, dispatched and unknown execution states. A lost response cannot establish a decline.

**Choice:** durable scoped command idempotency, semantic request hash, stable operation/provider key and explicit UNKNOWN state. **Alternative:** request-local IDs or cache-only replay checks are easier initially but cannot preserve intent through crash, eviction or multiple instances. **Validation:** PS-01/PS-02 count logical operations and simulator effects through concurrent replay and response loss.

## Concurrency and money

Capture and refund limits require atomic decisions about both confirmed effects and outstanding reservations. Checks outside a transaction can let two callers consume the same balance.

**Choice:** short PostgreSQL aggregate-row coordination, uniqueness/check constraints, one active financial operation initially, and version/fence checks on completion. External IO happens after local acceptance commits and without row locks. **Alternatives:** optimistic retries can work under low contention; Serializable still needs abort handling; neither makes remote payment effects transactional. **Validation:** PS-04/PS-05/PS-10 exercise races, unknown reservations and stale owners. See [domain model](domain-model.md) and [PostgreSQL isolation](https://www.postgresql.org/docs/current/transaction-iso.html).

## Processor integration and routing

Processors differ in status vocabulary, inquiry, retry identity retention, capture and reversal support. Normalization should clarify those differences instead of hiding them behind one success/failure flag.

**Choice:** explicit capabilities and typed normalized outcomes, recorded deterministic routing, original-processor follow-up actions, and safe fallback only for provably undispatched operations or documented non-execution. **Alternative:** dynamic routing and broad retry rules add complexity before safe failure classification exists. **Validation:** PS-02/PS-03/PS-06 use two independently stateful simulators and a shared adapter contract suite. See [processor design](processor-integrations.md) and [Stripe error handling](https://docs.stripe.com/error-low-level).

## Recovery and event delivery

Local commit, external financial effect, event publication and consumer acknowledgment cannot share one ordinary database transaction. Each gap requires a durable identity and a recovery rule.

**Choice:** persisted work intent, inquiry-first recovery, current-owner fencing, transactional outbox, and consumer inbox/effect transaction before offset commit. Unknown work remains visible when source truth is unavailable. **Alternatives:** publish-before/after-commit without an outbox can lose consistency; blanket exactly-once claims overstate broker guarantees. **Validation:** PS-07–PS-10 inject every commit/response/ack gap. See [AWS outbox guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) and [Kafka semantics](https://kafka.apache.org/43/design/design/).

## Settlement and reconciliation

Captured, settled and paid out are different observations. Missing evidence in an incomplete file cannot establish missing settlement, and net payout cannot be compared directly with gross capture.

**Choice:** versioned synthetic batch/row identities, exact reference/kind/currency matching, explicit gross/fee/net signs, complete-window/grace policy, immutable correction history and stable exception occurrences. **Alternative:** fuzzy matching can propose candidates later but must not silently decide financial truth. A general ledger or bank reconciliation service is separate future scope. **Validation:** PS-11 covers incomplete imports, duplicates, fee/amount discrepancies, ambiguity, late corrections and reruns.

## Architecture and measurement

**Choice:** modular monolith and PostgreSQL first; Kafka for later replayable consumers; infrastructure only where a demonstrated requirement exists. Correctness tests precede simulator-only performance experiments. **Alternative:** early service extraction or extra coordination systems introduce failure surfaces without proving the basic money/state rules.

**Validation:** all twelve [production scenarios](production-scenarios.md), independent operation with commerce absent, and [cross-project contract tests](commerce-integration.md). PS-12 reports actual offered/accepted/rejected/completed/unknown counts, financial-effect reconciliation, latency and resource limits. Exploratory workload targets are not results. No production use or scale is claimed.

The [ADRs](adr/001-modular-monolith.md), [testing strategy](testing-strategy.md) and [implementation plan](implementation-plan.md) define the detailed work and completion evidence.
