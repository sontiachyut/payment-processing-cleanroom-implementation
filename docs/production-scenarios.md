# Twelve mandatory payment production scenarios

All scenarios are future acceptance requirements, not implemented tests or observed results. Reference IDs H1–H17 resolve to pinned sources in [reference analysis](reference-analysis.md); those sources motivate concepts and do not certify our proposed guarantees. Every scenario remains mandatory.

## PS-01 — Concurrent duplicate authorization

- **Business/trigger:** mobile client and retry proxy submit identical authorization/key simultaneously; duplicated logical work can create multiple holds.
- **Expected behavior:** one durable payment/operation; same semantic payload replays one acceptance identity, changed payload conflicts. Distinct merchant scopes do not collide.
- **Concepts/alternatives:** unique insert arbitration versus cache locks, semantic hash versus raw JSON, business uniqueness versus transport replay. Choose PostgreSQL unique merchant/action/key plus business reference, not Redis as truth.
- **Transactions/strategy:** T1 inserts key/payment/operation/audit/outbox together; loser rolls back and reads committed winner in a fresh transaction. External call follows later.
- **Tests/fault injection:** barrier-release 50 connections, same/different payload, commit then lose HTTP response, roll back winner, two JVMs.
- **Acceptance:** one canonical accepted operation and one eventual simulator effect; no inconsistent replay body.
- **Interview:** what linearizes the command? Why can't a JVM map or an aborted-transaction lookup solve it?
- **References:** H2/H15/H17 give operation/locking boundaries; our same-key database strategy and race tests are original. Milestone 3.

## PS-02 — Processor succeeds but response is lost

- **Business/trigger:** processor records an authorization but network response disappears; marking failed or fallback can duplicate the effect.
- **Expected behavior:** UNKNOWN, retained operation identity/reservation, inquiry-first recovery; unresolved remains visible if source cannot prove outcome.
- **Concepts/alternatives:** external atomicity gap, durable attempts, inquiry versus stable-key resubmission versus escalation. Choose capability-specific recovery, never new provider identity on timeout.
- **Transactions/strategy:** T1 acceptance; claim/dispatch marker commit; provider call outside DB; T2 observation/transition if available. Crash after effect before T2 is recoverable by original key/ref.
- **Tests/fault injection:** simulator effect-then-drop-response, worker kill before finalization, inquiry unavailable then restored, key-retention expiry.
- **Acceptance:** one simulator effect; local state becomes source-confirmed or honestly unknown; client replay never starts another authorization.
- **Interview:** can rollback undo a charge? What if provider supports neither idempotency nor reliable inquiry?
- **References:** H10/H11/H17 and Stripe sync test; our explicit unknown reservation/fence policy is proposed. Milestone 5.

## PS-03 — Processor outage and fallback

- **Business/trigger:** primary processor A is down while B is healthy; unsafe fallback after ambiguous dispatch can charge twice.
- **Expected behavior:** new eligible, provably undispatched authorization may choose B; dispatched unknown A stays pinned; capture/refund follows original processor.
- **Concepts/alternatives:** priority versus weighted routing, health versus financial outcome, circuit breaker versus retry. Choose deterministic priority/capability selection and stored rule version.
- **Transactions/strategy:** persist selected provider/key and route decision before network. A pre-dispatch rejection can choose another eligible route under policy; uncertainty requires inquiry, not route mutation.
- **Tests/fault injection:** breaker open before send, connection loss after send, different capability sets, healthy B under A saturation.
- **Acceptance:** route history explains every choice; no cross-provider duplicate effect; B progresses for eligible new work.
- **Interview:** what evidence makes fallback safe? Does an open breaker prove an earlier request failed?
- **References:** H8/H9 and routing tests; no blanket Hyperswitch fallback guarantee inferred. Milestones 4–5.

## PS-04 — Concurrent capture

- **Business/trigger:** fulfillment callbacks capture one authorization twice or exceed remaining authorized balance.
- **Expected behavior:** one active financial operation initially; captured plus reserved amount bounded; stable replay versus conflicting distinct command clearly distinguished.
- **Concepts/alternatives:** pessimistic row lock, optimistic compare-and-set, serializable retries, parallel reservations. Choose short aggregate lock/guard first, with versions for stale completion.
- **Transactions/strategy:** lock/recheck/reserve/create operation in T1; provider outside transaction; apply once/release reservation/audit/outbox in T2. UNKNOWN retains reservation.
- **Tests/fault injection:** capture/capture and capture/cancel barriers, different keys, lock timeout, stale worker, duplicate success observations.
- **Acceptance:** no overcapture; losing command never reaches provider; totals remain valid after crash/replay.
- **Interview:** why is a pre-transaction amount check insufficient? Which isolation anomaly matters?
- **References:** H3 and capture/partial-capture tests establish inspected validation paths, not our concurrent balance proof. Milestone 6.

## PS-05 — Concurrent refunds

- **Business/trigger:** capture 100.00, confirmed refund 60.00; two 30.00 requests race, one uncertain.
- **Expected behavior:** confirmed+reserved refunds ≤ capture; only eligible request accepted, unknown reservation retained, known failure releases exactly its own reservation.
- **Concepts/alternatives:** reservation arithmetic versus confirmed-only sum; serial execution versus parallel reservations. Choose one active financial operation and DB checks first.
- **Transactions/strategy:** payment lock and reserve at acceptance; external refund outside DB; effect/total/reservation/audit/event atomic at confirmation.
- **Tests/fault injection:** two distinct-key refunds, same-key replay, effect-then-timeout, late success/failure, duplicate webhook/inquiry, currency/overflow boundaries.
- **Acceptance:** no over-refund and one local effect per operation; captured payment remains captured when refund fails.
- **Interview:** when is budget available again? Why does a timeout not release the reservation?
- **References:** H5/H12 provide refund/recovery separation; our reservation/race tests are original. Milestone 6.

## PS-06 — Rate limiting and transient failures

- **Business/trigger:** processor returns 429 or intermittent errors; layered retries saturate pools and grow queue.
- **Expected behavior:** bounded classified retries, jitter, total deadline, provider isolation and controlled admission. No retry of known decline or unsafe ambiguous financial effect.
- **Concepts/alternatives:** retry-all versus never-retry versus explicit safety matrix; shared pool versus per-provider bulkheads. Choose one retry owner per boundary, durable due work and capacity limits.
- **Transactions/strategy:** store next due/attempt count in short job transaction; no sleeping under DB locks; preserve key across retries.
- **Tests/fault injection:** fake 429/Retry-After, 500 after effect, slow A/healthy B, exhausted budget, full executor/DB pool, retry amplification probe.
- **Acceptance:** bounded calls per operation, no memory queue explosion, predictable 429/503 admission and recoverable accepted work.
- **Interview:** breaker, bulkhead and retry solve what different problems? Why can more threads worsen throughput?
- **References:** H9/H13 and retry metrics; our explicit matrix is narrower. Milestones 5/10.

## PS-07 — Duplicate and out-of-order webhooks

- **Business/trigger:** provider repeats an event, delivers old pending after success, or response/inquiry/webhook confirm same operation.
- **Expected behavior:** verify source and size first; durable inbox, stable delivery dedup and effect dedup; no stale regression. Conflicting authoritative evidence is preserved/escalated.
- **Concepts/alternatives:** event ID versus operation identity, arrival time versus source sequence, trust versus payload parsing. Choose typed observation merge with canonical inquiry when order is unavailable.
- **Transactions/strategy:** inbox insertion commits before 2xx; processing applies observation/effect/outbox atomically; repeated ID/different digest quarantines. Cursor/version gaps remain explicit.
- **Tests/fault injection:** duplicate and permutation corpus, invalid signature, new event ID/same effect, reused ID/changed body, stale summary and crash after inbox commit.
- **Acceptance:** one effect, retained history, no final→pending rollback, no unverified event mutating state.
- **Interview:** why aren't timestamps sufficient? Why doesn't deduplicating webhook ID prevent all duplicate effects?
- **References:** H14/H15; provider-specific replay guarantees not generalized. Milestone 8.

## PS-08 — DB commit succeeds, publication fails

- **Business/trigger:** capture commits and relay/network crashes before Kafka receives the fact; downstream commerce misses it.
- **Expected behavior:** committed transition always has durable event intent; eventual relay recovers. A publish acknowledgment lost before marker causes duplicate, not lost identity.
- **Concepts/alternatives:** publish-before/after DB, XA, polling outbox, CDC. Choose transactional outbox and polling first.
- **Transactions/strategy:** payment/audit/outbox in one DB commit; external publish and marker separate; stable event ID/sequence across attempts.
- **Tests/fault injection:** fail before DB commit, after commit, after broker effect, before marker; rollback must publish no committed fact.
- **Acceptance:** state and intent both/neither; replay contains original identity; bounded backlog monitored and recoverable.
- **Interview:** what atomicity gap does outbox solve, and which gap remains? Does it deliver exactly once?
- **References:** H16 motivates notification retries; this outbox implementation is our design, not verified Hyperswitch behavior. Milestone 7.

## PS-09 — Duplicate Kafka consumption

- **Business/trigger:** consumer commits local projection then dies before offset; rebalance redelivers.
- **Expected behavior:** one canonical local effect for stable event/operation identities; malformed schema quarantined; old versions do not regress source observations.
- **Concepts/alternatives:** offset-first loss, effect-first duplicates, Kafka-only transactions versus external DB cooperation. Choose inbox+effect DB transaction then offset.
- **Transactions/strategy:** insert source/event unique guard and update projection/job intent atomically. Acknowledge only after successful commit; report divergent reused IDs.
- **Tests/fault injection:** kill after DB/before offset, reset isolated group offsets, duplicate outbox publication, reversed versions, unknown major schema.
- **Acceptance:** count of canonical effects/cases unchanged after replay; every committed valid fact accounted for or visibly quarantined.
- **Interview:** where does Kafka's exactly-once scope end? Why isn't producer idempotence sufficient?
- **References:** no inspected Hyperswitch test proves this project-specific guarantee; use official [Kafka semantics](https://kafka.apache.org/43/design/design/). Milestone 7.

## PS-10 — Recovery worker crashes mid-operation

- **Business/trigger:** worker dies or pauses after claim/provider effect; new worker reclaims while old may resume.
- **Expected behavior:** durable due work, fenced ownership, inquiry-first handling of dispatched ambiguity, no stale finalization or duplicate remote effect under declared provider capabilities.
- **Concepts/alternatives:** singleton worker versus leases/fences versus workflow engine. Choose PostgreSQL claims and monotonically increasing fence; clarify external limits.
- **Transactions/strategy:** short claim commit; network outside DB; current fence check with effect application. Expired lease does not mean provider stopped.
- **Tests/fault injection:** kill/pause before/after effect, expire lease, start second worker, release old worker, lose DB connection, unavailable inquiry.
- **Acceptance:** every accepted operation is final or visibly unresolved; stale owner cannot overwrite new state; reservations remain correct.
- **Interview:** what protects local state versus remote effects? Why test process pause as well as kill?
- **References:** H10/H12/H13 scheduling and synchronization; our fencing policy is original. Milestone 8, foundations in 5.

## PS-11 — Settlement mismatch

- **Business/trigger:** complete batch omits an eligible capture, changes gross/fee/currency or duplicates row identity.
- **Expected behavior:** deterministic matched/mismatched/ambiguous/overdue-missing exceptions; incomplete files do not prove absence; corrections preserve revisions.
- **Concepts/alternatives:** exact versus fuzzy matching, operation versus payout reconciliation, gross/fee/net signs, timezone/calendar/grace. Choose exact identity/kind/currency and explicit completeness first.
- **Transactions/strategy:** stage bounded batch/rows; validate; close input window; match/run/result/exception identities commit in bounded chunks with resumable checkpoints; do not rewrite payment facts.
- **Tests/fault injection:** duplicate batch/row, mid-import crash, late correction, amount/fee/net inconsistency, missing after grace, incomplete window, ambiguous references.
- **Acceptance:** repeat import/match is idempotent; exception includes source row/window/reason; no automatic refund or invented bank settlement.
- **Interview:** what proves settlement should exist already? Why is net payout not gross capture?
- **References:** public reconciliation example is documentation-only evidence, not inspected engine code. Milestone 9.

## PS-12 — High-volume concurrency and backpressure

- **Business/trigger:** offered load exceeds provider/DB capacity, hot merchant contends, queues grow.
- **Expected behavior:** preserve all financial invariants, bound accepted work/resources, controlled rejection, measurable queue drain and recoverability.
- **Concepts/alternatives:** open arrival-rate versus closed loop, tail latency/coordinated omission, connection pools, index plans, fairness, Little's Law assumptions. Choose k6/local stateful processors and correctness reconciliation.
- **Transactions/strategy:** same short boundaries as other scenarios; bounded workers/pools/admission; no in-memory queue as sole durable acceptance.
- **Tests/fault injection:** baseline then exploratory 100/500/1,000 RPS if machine/correctness permits; hot key, replay storm, slow provider, Kafka/DB pressure, stop-before-exhaustion bounds.
- **Acceptance:** report offered/accepted/rejected/completed/unknown counts, simulator effect reconciliation, p95/p99, queue/lock/pool metrics, hardware/config/seed; no external load traffic or fabricated targets-as-results.
- **Interview:** can higher HTTP throughput mean fewer financial completions? What is the bottleneck and evidence?
- **References:** Hyperswitch metrics/topology are contextual, not our benchmark. Milestone 10.
