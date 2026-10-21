# ISSUE-052 — internal/repository: unlock aborts when a lock file was already removed

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

[S] `RemoveStaleLocks` and `RemoveAllLocks` return every removal error, including not-exist errors for locks deleted concurrently.

## Reach-and-Impact

Review correspondence: E34; severity: Low.
[S] `unlock --remove-all` racing with another process's lock refresh or exit can fail and leave remaining locks.
[A] Backend-specific not-exist behavior is unverified beyond the local backend.

## Evidence

- [S] `internal/repository/lock.go:280-284` — stale lock removal returns errors.
- [S] `internal/repository/lock.go:296-300` — forced removal returns errors and aborts the parallel listing.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Treat backend not-exist errors as successful removal in both callbacks.

## Scope-and-Constraints

- Preserve: reporting other removal errors.
- Exclude: locking protocol changes.
- Cost: minimal.

## Verification

- Delete a lock file between listing and removal, run `restic unlock --remove-all` → exit 0.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce unlock not-exist abort
Action: Force a lock removal race with a disposable local repository on current upstream.
Done-When: Exit status, remaining locks, and versions are recorded.
