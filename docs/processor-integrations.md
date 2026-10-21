# Proposed processor adapters, routing and recovery

No adapter code or integration configuration exists. The proposed interface, simulator behavior and retry rules define explicit capability and normalization boundaries for this project.

## Common port

Capabilities declare authorize/capture/cancel/refund/reversal/inquiry support; manual/partial capture; stable idempotent resubmission guarantees and retention; reference/key inquiry support; validity windows and normalized error classes. Request includes stable operation/provider key, synthetic method/ref, integer money/currency, deadline and trace link. Response includes known success, known decline, provably undispatched rejection, or ambiguous outcome, plus sanitized provider reference and timestamps.

Never normalize away UNKNOWN. A socket error does not establish that the remote effect did not happen. Store selected provider and rule version before dispatch; capture/cancel/refund remain attached to the original financial chain.

## Two independent stateful simulators

| Simulator | Normal behavior | Failure controls and purpose |
| --- | --- | --- |
| Processor A | Manual authorize/capture, cancel, partial refund, stable-key replay and inquiry by key/reference | Apply effect then lose response; return late confirmation; rate limit; inject tail latency; test safe recovery |
| Processor B | Same core lifecycle through different request/status vocabulary; narrower reversal/inquiry capability in selected scenarios | Known pre-dispatch outage, explicit decline, inquiry unavailable and key-retention expiry; exercise honest unresolved state |

Each keeps its own durable effect state separately from Java app state. Resetting both during a restart would erase evidence of the atomicity gap. Simulator hidden oracle is available to tests, not API clients/AI. Seed and fault point make tests reproducible. Do not treat assumed timing distributions as real provider latency.

## Routing and fallback

Start deterministic capability/currency eligibility and configured priority. Add per-provider health/breakers without dynamically moving an already dispatched command. Record candidate set, selected provider, rule version, pre-dispatch health and rationale.

Safe fallback: a new authorization has not been dispatched, a selected provider's breaker rejects before IO, or explicit authoritative proof establishes non-execution under a reviewed policy. Ambiguous dispatch/timeout/500 is not permission to try another processor. Distinguish circuit-open admission from an earlier operation already in flight. Provider-specific retry policies must be checked against declared capabilities and cannot be assumed universal.

| Observation | Default policy | Evidence needed |
| --- | --- | --- |
| Input/policy rejection | Do not retry | Fix command/permission; new user decision if required |
| Known decline | Persist final failure; no generic retry | Specific provider policy and business authorization for another attempt |
| Breaker/capacity reject before dispatch | Bounded defer or eligible new-route selection | Durable marker proves request not sent |
| 429 / transient response | Backoff+jitter/deadline at one owning layer | Provider semantics establish retry safety; financial identity unchanged |
| Lost response / uncertain dispatch | UNKNOWN and inquiry-first | Stable provider key/reference; no new provider identity |
| Inquiry unavailable | Bounded reschedule then visible escalation | Preserve uncertainty/reservation; no invented failure |
| Confirmed result from any source | Apply local effect once | Operation identity, amount/currency/status consistency and current fence |

Per-provider bulkheads, bounded queues, total time/retry budgets and circuit breakers arrive after basic behavior works. Retry layers must not multiply API, worker and adapter attempts. Recovery jobs persist due time/generation and use leases/fences; old dispatched jobs inquire instead of blindly resending.

## Void, reversal, expiration and compensation

Cancellation of a confirmed uncaptured authorization may release a hold; actual semantics are adapter-specific. Timeout reversal needs a linked original reference and provider support, and reversal can itself be unknown. Captured funds require refund semantics. Local authorization expiry is not proof a provider released funds. Observe authoritative result before reporting compensation complete.

Commerce recovery is deterministic: no dispense after unknown authorization; confirmed failed dispense with uncaptured authorization requests cancel/void where supported; already captured failed fulfillment requests a governed idempotent refund; unknown dispense never triggers blind repeat dispense/refund. The model cannot call raw financial tools; commerce coordinator owns this policy.

## Stripe sandbox later

Introduce a third adapter at milestone 11 using a dedicated sandbox, test-only keys/method IDs, documented manual-capture/refund/webhook behavior and pinned API version. No real PAN/CVC or live-mode configuration. Sandbox integration is low volume, never high-throughput load. It does not model arbitrary network/settlement reality; local proxy response loss is our injected fault, not control over Stripe internals. [Official testing](https://docs.stripe.com/testing), [load-testing limitations](https://docs.stripe.com/rate-limits#load-testing).

## Contract verification

One adapter suite covers capability declarations, response mapping, stable replay keys, source amount/currency integrity, inquiry, known decline versus unknown, and retained simulator effect after application restart. Each guarantee requires tests against this project's implementation. [Design rationale](design-rationale.md) records the decisions and their validation boundaries.
