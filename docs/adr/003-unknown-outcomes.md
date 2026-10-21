# ADR 003 — Durable acceptance and explicit uncertainty

**Status:** proposed. External financial effects cannot share the local DB transaction. Choose 202 durable operation acceptance, provider identity before dispatch, attempt/observation history, UNKNOWN state and capability-specific inquiry/idempotent replay. Never cross-provider fallback after ambiguous dispatch.

Synchronous-only waiting hides but does not remove ambiguity; retry-all risks double effects; workflow infrastructure does not invent missing provider truth. Clients must inquire, and some outcomes remain unresolved. Reversal/cancel/refund are separate operations and can also time out. PS-02/03/06/10 validate every crash boundary and stated provider assumptions.
