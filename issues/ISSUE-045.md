# ISSUE-045 — internal/backend/b2: failed saves close the writer and may commit truncated objects

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: Medium
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `b2Backend.Save` calls `w.Close()` on copy and length errors before cancelling the writer context.
[A] blazer `Writer.Close` finalizes data already written; reported by review from blazer v0.7.2 `b2/writer.go:509-560`, not independently read.

## Reach-and-Impact

Review correspondence: E27; severity: Low.
[A] A read error mid-upload can leave a truncated object under its final name if retries do not overwrite it.
[S] `HasAtomicReplace` is true for b2, so the retry backend does not remove failed uploads.

## Evidence

- [S] `internal/backend/b2/b2.go:224-225` — cancel runs only when `Save` returns.
- [S] `internal/backend/b2/b2.go:234-243` — error paths call `w.Close()`.
- [S] `internal/backend/b2/b2.go:152` — `HasAtomicReplace` is set.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched; blazer semantics not verified directly.

Contribution fit: unresolved until blazer behavior and prior art are checked.

## Proposed-Change

Call `cancel()` before `w.Close()` on both error paths.

## Scope-and-Constraints

- Preserve: successful upload finalization.
- Exclude: other cloud backends.
- Cost: minimal once blazer cancellation semantics are confirmed.

## Verification

- With a mocked blazer transport and a reader failing after N bytes, no upload finalize request is sent.

## Publication-Blockers

- blazer close semantics must be verified directly.
- Upstream prior art is missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Verify blazer close semantics
Action: Read blazer v0.7.2 `Writer.Close` and its cancellation handling in the module cache.
Done-When: Whether closing after a copy error commits data is recorded with exact source lines.
