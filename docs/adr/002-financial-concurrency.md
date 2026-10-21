# ADR 002 — Short DB coordination for financial decisions

**Status:** proposed. Capture/cancel/refund races require cross-process balance protection. Choose Read Committed with short payment-row lock, database checks/uniqueness, explicit reservations, one active financial operation and version/fence guarded completion. No network IO under DB lock.

Optimistic-only retries can be efficient at low contention but require whole-decision restart; Serializable needs abort handling and still cannot roll back remote effects. JVM locks are process-local; Redis leases do not atomically protect DB money. Consequence: bounded hot-payment throughput and honest conflicts/unknown holds. PS-01/04/05/10 must prove invariants before any parallel operation redesign.
