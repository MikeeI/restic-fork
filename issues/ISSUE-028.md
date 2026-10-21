# ISSUE-028 — internal/backend/cache: cache write failures fail saves that already reached the repository

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: Medium
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `cacheBackend.Save` returns the local cache write error after the backend save succeeded.
[S] `Cache.list` treats in-flight `tmp-*` files as cache entries, so a concurrent `Clear` can delete them before rename.

## Reach-and-Impact

Review correspondence: E7; severity: Medium.
[S] ENOSPC in the cache directory, or a concurrent `Clear` from another process sharing the cache, fails index, snapshot, or metadata pack saves.
[S] The object already exists remotely while the command exits with an error.
[S] The cache sits above the retry backend, so the error is not retried.
[A] The race window frequency between `CreateTemp` and `Rename` is unmeasured.

## Evidence

- [S] `internal/backend/cache/backend.go:74-89` — remote save, then cache error returned.
- [S] `internal/backend/cache/file.go:93,111` — temp file in the type directory, then rename.
- [S] `internal/backend/cache/file.go:171-211` — `Clear` removes every listed name absent from `valid`.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Log cache write failures in `cacheBackend.Save` and return nil.
Skip `tmp-` names in `Cache.list`.

## Scope-and-Constraints

- Preserve: backend save errors and cache read fallback behavior.
- Exclude: cache locking redesign.
- Cost: small; maintainers may want a warning instead of a debug log.

## Verification

- Place the cache on a full tmpfs and run `restic backup` → exit 0 and the snapshot exists remotely.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce cache-full save failure
Action: Run a backup with the cache directory on a full tmpfs against current upstream.
Done-When: Exit status, remote snapshot presence, and versions are recorded.
