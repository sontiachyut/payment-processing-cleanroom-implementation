# ADR 005 — Separate payment, commerce and reconciliation truth

**Status:** proposed. Java owns financial facts; commerce owns user approval, grant budget, stock/order/dispense; AI only interprets/selects/proposes under deterministic policy. Payment service validates trusted client scope/binding and its own invariants without depending on commerce DB availability.

One global success flag or shared DB would obscure independent recovery. Captured is not settled; missing settlement is not a failed capture; user purchase permission is not a processor hold. Consequences: explicit versioned contracts, compensation and honest partial outcomes. Full ledger/bank payout reconciliation and protocol interoperability remain separate future scope. Validate independent demos and AC/PS cross-matrix.
