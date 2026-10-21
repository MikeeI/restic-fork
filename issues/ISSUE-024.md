# ISSUE-024 — cmd/restic: recover panics on an unreadable snapshot file

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] The `recover` callback for `ForAllSnapshots` ignores its error argument and dereferences `*sn.Tree`.
[S] `ForAllSnapshots` passes `sn == nil` together with the load error.

## Reach-and-Impact

Review correspondence: E3; severity: High.
[S] Any corrupt, truncated, or undecryptable snapshot file makes `restic recover` panic with exit status 2.
[S] `RepairIndex` has already run before the panic.
[S] The repair tool fails precisely in the damaged-repository scenario it exists for.

## Evidence

- [S] `cmd/restic/cmd_recover.go:115-117` — callback signature discards the error and reads `*sn.Tree`.
- [S] `internal/data/snapshot.go:113-116` — `fn(id, sn, err)` with nil `sn` on load failure.
- [S] `internal/data/snapshot_find.go:55-58` — other callers check `err` first.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Return the callback error, and reject a snapshot whose `Tree` is nil, before dereferencing.

## Scope-and-Constraints

- Preserve: root detection for readable snapshots.
- Exclude: skipping unreadable snapshots, which would misclassify their roots as unreferenced.
- Cost: minimal.

## Verification

- Corrupt one file under `snapshots/` in a disposable repository, run `restic recover` → error instead of panic.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce recover snapshot panic
Action: Corrupt a snapshot file in a disposable repository and run current upstream `restic recover`.
Done-When: Panic output, exit status, and versions are recorded.
