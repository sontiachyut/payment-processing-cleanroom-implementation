# ADR 001 — Business modules in one initial deployable

**Status:** proposed. One engineer must understand financial decisions before introducing network boundaries. Choose one Spring Boot modular monolith/authoritative PostgreSQL with explicit policy, orchestration, adapter, routing, recovery, event, reconciliation and operations boundaries. Workers can share its artifact later.

Microservices offer independent releases but introduce distributed consistency/identity/operations costs prematurely; an unstructured monolith lets adapters bypass policy. Consequences: simpler local transactions, coupled deployment and required module reviews. Extract only when measured scaling/isolation/ownership requirements justify a new data/contract/recovery design.
