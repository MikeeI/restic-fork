# ISSUE-013 — internal/backend: failed RoundTrip retains its watchdog until later cancellation

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] The wrapper starts a watchdog and child context but does not cancel them when `RoundTrip` returns an error.
[S] They persist until timeout or parent cancellation instead of ending with the failed attempt.

## Reach-and-Impact

[S] The REST backend passes the existing parent context to `client.Do`, so an immediate failure can leave it alive.
[S] The default watchdog timeout is five minutes; this is delayed cleanup, not an unbounded permanent leak.
[A] Retained resources and failure-rate-dependent accumulation are unmeasured.

## Evidence

- [S] `internal/backend/watchdog_roundtriper.go:37-77`: watchdog start, error return, and successful body-close cancellation.
- [S] `internal/backend/http_transport.go:144-149`: five-minute default timeout.
- [S] `internal/backend/rest/rest.go:225-237`: existing-context request and client error return.
- [S] `internal/backend/watchdog_roundtriper_test.go:43-100`: successful streaming and already-canceled-parent coverage.

## Prior-Art

Coverage: the complete local ledger was screened for watchdog failure cleanup on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Cancel the attempt's watchdog in the error branch after the existing timeout-error translation.
Keep successful response-body cancellation tied to body cleanup rather than function return.

## Scope-and-Constraints

- Preserve timeout identity, upload/download monitoring, and successful body lifetime.
- Do not introduce unconditional deferred cancellation that aborts streaming responses.
- Preserve native transport errors and caller-owned parent cancellation.
- Review correspondence: P11; reliability ownership explains the performance/resource symptom.

## Verification

Planned, not run: immediate transport failure with a live parent, plus existing successful streaming checks.
Confirm prompt watchdog/timer cleanup, a readable successful body until close, and unchanged timeout identity.
The existing canceled-parent check alone does not reproduce this failure; use a disposable scenario, not a new test.

## Publication-Blockers

- No observed live-parent cleanup reproduction or correction verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce watchdog failure retention
Action: Observe watchdog lifetime after an immediate RoundTrip failure while its parent remains alive.
Done-When: request failure, parent state, cleanup delay, resource counts, command, and environment are recorded.
