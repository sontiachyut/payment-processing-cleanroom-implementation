# Risks and unresolved decisions

All application work remains unstarted. No proposed mechanism is a demonstrated guarantee until tested.

| Risk / open decision | Planned response and gate |
| --- | --- |
| Provider offers neither idempotency nor reliable inquiry | Retain UNKNOWN/escalate; never invent a safe retry guarantee |
| Old worker continues after lease loss | Fence local effects; provider stable key protects remote replay only under declared semantics; PS-10 |
| Hot payment contention | Short DB lock and one active operation first; measure before parallel reservations |
| Dependency compatibility | Java 21, Spring Boot 4.x and managed JUnit 6/Jupiter are selected; verify and pin compatible patches and integration libraries at M1 |
| Reference scope exceeds project | Original small domain, no Rust translation or inherited universal guarantees; pinned static inspection only |
| Reconciliation product not fully verified in OSS | Synthetic deterministic matcher, honest source/documentation distinction |
| Outbox ordering under concurrent relay | Single relay first; per-payment sequence, gap handling and aggregate-aware ownership before scale-out |
| Webhook duplicates/contradictions | Verified inbox and effect identity, preserve source observations and inquire instead of timestamp overwrite |
| Reversal/capture semantics vary | Capability matrix and separate linked operations; unsupported route rejected |
| Commerce revocation after dispatch | Trusted accepted binding plus actual inquiry/compensation; no promise to undo a possibly executed effect |
| Dispensed but capture fails | Visible commerce financial recovery; no second dispense/new unapproved charge |
| Performance claims outpace evidence | Targets labeled exploratory, actual effect reconciliation and hardware/config with every result |
| Deployment/security/cost unknown | Local synthetic-first; no ECS/RDS/model charges or public runtime until separately approved |

No ledger, PAN/CVC, real network certification, FX/payouts or active-active writes in MVP. Exact dependency patches, API versions, retry retention, grace periods, measured capacity and cloud topology are future reviewed decisions. Existing legacy repositories are not silently renamed/deleted by this new project.
