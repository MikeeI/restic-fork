# ISSUE-027 — internal/repository: check reports every pack missing after a pack listing error

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `Checker.Packs` sends the `List` error and continues comparing the index against a partial pack map.

## Reach-and-Impact

Review correspondence: E6; severity: Medium.
[S] A failed pack listing yields `does not exist` errors for every unlisted indexed pack and inflates `NumErrors`.
[S] `check` then recommends `restic repair packs <ids>` and `restic repair snapshots --forget`.
[S] Exit status is already nonzero, and repairing intact packs salvages all blobs, so the impact is misleading diagnosis and unnecessary repair work, not data loss.

## Evidence

- [S] `internal/repository/checker.go:192-196` — the earlier error path returns.
- [S] `internal/repository/checker.go:208-210` — the listing error path does not return.
- [S] `internal/repository/checker.go:212-224` — unlisted packs are reported as missing.
- [S] `cmd/restic/cmd_check.go:409-414` — repair commands are printed for `salvagePacks`.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Return after sending the listing error, using the existing context-aware `select`.

## Scope-and-Constraints

- Preserve: missing, truncated, and orphaned pack reporting after a complete listing.
- Exclude: retry policy changes.
- Cost: minimal.

## Verification

- Wrap the backend so `List(PackFile)` fails after N entries, run `Checker.Packs` → only the listing error is emitted.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check pack-listing prior art
Action: Search upstream for check reporting missing packs after listing failures.
Done-When: Search coverage and candidate classifications are recorded.
