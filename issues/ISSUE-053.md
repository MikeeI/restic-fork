# ISSUE-053 — internal/fuse: directory attributes read items without locking before open

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

[S] `dir.Attr` counts `d.items` without holding `d.m`, while `dir.open` assigns `d.items` under the lock.

## Reach-and-Impact

Review correspondence: A1; severity: Low.
[S] Concurrent lookup and attribute requests race on `d.items`.
[S] Before the first `open`, directories report link count 2 regardless of subdirectories.

## Evidence

- [S] `internal/fuse/dir.go:140` — `d.items` assigned under `d.m`.
- [S] `internal/fuse/dir.go:157,162-172` — unlocked read in `calcNumberOfLinks`.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-014 owns FUSE name collision performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Lock `d.m` in `calcNumberOfLinks` and open the directory in `Attr` before counting.

## Scope-and-Constraints

- Preserve: lazy tree loading semantics.
- Exclude: tree cache changes.
- Cost: `Attr` may trigger a tree load.

## Verification

- `go test -race ./internal/fuse` with concurrent `Lookup` and `Attr` → no race report.

## Publication-Blockers

- Race reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce FUSE directory race
Action: Run concurrent `Lookup` and `Attr` calls under the race detector against current upstream.
Done-When: Race detector output and versions are recorded.
