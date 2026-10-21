# ADR 004 — Outbox and at-least-once integration

**Status:** proposed. Commit state/audit/outbox together, poll stable event IDs, publish then mark; consumer inbox+effect commits before offset. Kafka arrives at M7 for replayable independent consumers, not as a financial lock. Per-payment sequence and source inquiry handle duplicates/gaps/reordering.

Direct publication creates a dual-write gap; XA adds coordination; CDC is useful later but unnecessary initial machinery. Consequences: delivery lag/duplicates, backlog limits, quarantine/replay and schema governance must be operated explicitly. Producer idempotence/Kafka transactions do not atomically include external DB/processor effects. PS-07/08/09 are required evidence.
