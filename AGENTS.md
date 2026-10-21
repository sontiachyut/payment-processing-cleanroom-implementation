# Milestone-controlled engineering and learning

Current authorization: milestone 0, research/architecture/documentation only. Do not generate application scaffolding, Java/Python functionality, migrations, tests, broker configuration, CI/CD, or infrastructure during this planning task. Proposed designs are not completed features.

Future sessions must:
1. Read README, the relevant milestone, scenario, domain/API design and ADR.
2. Teach the concrete engineering problem and fundamentals.
3. Show the pinned Hyperswitch source/test references and distinguish verified behavior from our proposed design.
4. Discuss alternatives and obtain the owner's approval before beginning a milestone.
5. After approval, generate code/tests and supporting changes only within that milestone's authorized scope.
6. Run checks, debug failures and explain important code paths, transaction/concurrency boundaries and recovery behavior.
7. Ask practical questions to validate understanding.
8. Update progress, ADRs, README and actual results only when acceptance evidence warrants completion.

The twelve PS scenarios are mandatory. Do not replace them with easy happy paths. No former-employer materials, real payment/card data, fabricated measurements, or reconstructed confidential architecture. Public sources are references, not code to translate verbatim.

Payments own financial truth. Commerce authorization and order/stock state are separate. An LLM never sets payment state or financial invariants. Keep both repositories independently runnable through narrow simulated adapters.

Use exact repository-scoped Git operations. No real payments, billable cloud/model calls or public runtime deployment without explicit authorization. Public documentation publication is authorized for milestone 0 and does not authorize implementation. Do not remove similarly named or legacy repositories.
