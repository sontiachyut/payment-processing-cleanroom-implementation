# Proposed product requirements

**Baseline:** 2026-10-21. **Status:** milestone-0 documentation only; no application functionality exists.

## Users, problem and use cases

An API integrator needs stable command identity and inquiry when client responses disappear. Payment operators need to explain attempts and unknown outcomes. Reconciliation operators need complete-window evidence before declaring settlement missing. A commerce service needs authorization/capture/cancel/refund results it can trust independently of agent text.

The platform orchestrates synthetic card-style payments. It does not hold funds, connect to card networks directly, store PAN/CVC, or claim regulatory certification. Its educational purpose is correctness under concurrency and failure, with inspectable boundaries and reproducible tests.

## Functional requirements

| ID | Required capability | Scenario coverage |
| --- | --- | --- |
| F1 | Accept authenticated authorization commands with tenant/merchant scope, positive minor-unit amounts and allowlisted currency | PS-01, PS-02 |
| F2 | Model logical payment, operation, provider attempt and observation independently | PS-02, PS-10 |
| F3 | Capture no more than authorized; refund no more than captured including pending/unknown reservations | PS-04, PS-05 |
| F4 | Cancel uncaptured authorization; support reversal only where the adapter declares semantics | PS-02, commerce AC-07 |
| F5 | Durable scoped idempotency; same key/different payload conflicts; same command replays original acceptance identity | PS-01, AC-05 |
| F6 | Two simulated processors with normalized request/result contracts, sticky follow-up routing and recorded selection reasons | PS-03, PS-06 |
| F7 | Unknown-outcome state, inquiry-first synchronization, bounded classified retries and durable worker ownership | PS-02, PS-06, PS-10 |
| F8 | Verified webhook ingress, inbox deduplication, stale observation handling and conflict escalation | PS-07 |
| F9 | Atomic state/audit/outbox commit, stable event identity, replay-safe consumer effects | PS-08, PS-09 |
| F10 | Import versioned synthetic settlement records; exact identity/currency/amount/fee matching with completeness and grace policies | PS-11 |
| F11 | Tenant-scoped transaction/history/search, correlation, health and failure visibility | All |
| F12 | Standalone API/simulators, later Stripe sandbox, and independent commerce integration | PS-12, cross-project matrix |

## Scope and release gates

Milestones 1–3 establish the durable authorization slice and concurrent command behavior. Milestones 4–6 complete the initial financial lifecycle with two simulators. Milestones 7–9 cover asynchronous operations evidence, webhooks/recovery and reconciliation. Milestone 10 measures correctness under overload. Milestone 11 adds optional low-volume Stripe sandbox and separately approved deployment. Milestone 12 verifies commerce contracts; their design is already part of milestone 0.

MVP: one synthetic merchant/user integration, USD integer minor units, authorization, one full capture, uncaptured cancellation, full then partial refunds, inquiry, stable idempotency and visible uncertainty. All twelve scenarios remain required project acceptance cases even if not every feature belongs to the first vertical slice. Production-scale performance is not a prerequisite for a correct financial MVP.

Advanced: multiple partial captures, additional currencies, richer routing, authorization expiry, additional processor reversal protocols, partitioned relay/CDC and service extraction only with measured justification.

Excluded: real funds/cards, vaulting, underwriting, bank settlement connectivity, general ledger/account balances, FX, payouts, chargeback adjudication, multi-region active-active writes, AI financial decisions, and physical kiosk hardware control.

## Nonfunctional requirements

- **Correctness:** zero duplicate simulator effects for a replayed financial operation and zero invariant violations in the defined race/crash corpus. This is a proposed gate, not a result.
- **Consistency:** payment-aggregate financial decisions commit in PostgreSQL; event/read projections are eventually consistent and expose source version/lag.
- **Durability:** returned 202 means committed acceptance. Process restart recovers it; database hardware-loss durability requires separately validated backup/WAL procedures.
- **Availability:** reject writes if durable acceptance is unavailable. A provider, Kafka or commerce outage must not be confused with final payment failure.
- **Bounded work:** request/file limits, DB/HTTP pools, provider bulkheads, deadlines, queue/admission bounds and retry budgets.
- **Security:** authenticated merchant scope, distinct command/evidence roles, sanitized events/logs, no secrets in repositories or model inputs.
- **Visibility:** request→operation→attempt→event traces; queue age, unknown age, lock/pool waits, retry counts, reconciliation completeness.
- **Performance:** explore 100/500/1,000 offered requests per second against local simulators after a smaller baseline; publish actual rejection/completion/unknown counts and p95/p99. No target is an achieved metric or guaranteed capacity.
- **Reproducibility:** pinned dependency matrix, seeded scenarios and documented reset/recovery introduced during approved implementation, not scaffolding now.

## Success criteria

An engineer can explain every PS scenario, demonstrate accepted work surviving a restart, distinguish unknown from failed, prove balances under concurrent requests, replay events without duplicate local effects, and reconcile a complete synthetic batch. Commerce can exercise these interfaces without becoming a payment engine; Java works with commerce stopped.
