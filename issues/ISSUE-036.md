# ISSUE-036 — cmd/restic: snapshots --json returns success after output errors

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

[S] The JSON branch of `runSnapshots` prints the `printSnapshotGroupJSON` error and returns nil.

## Reach-and-Impact

Review correspondence: E15; severity: Medium.
[S] Write failures such as ENOSPC or EPIPE yield truncated JSON with exit status 0.
[S] The text branch returns equivalent errors.

## Evidence

- [S] `cmd/restic/cmd_snapshots.go:125-130` — error printed, then `return nil`.
- [S] `cmd/restic/cmd_snapshots.go:144-147` — text output errors are returned.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Return the `printSnapshotGroupJSON` error.

## Scope-and-Constraints

- Preserve: JSON output shape.
- Exclude: other commands' JSON error handling.
- Cost: minimal.

## Verification

- `restic snapshots --json > /dev/full; echo $?` → nonzero.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce snapshots JSON exit
Action: Run `restic snapshots --json > /dev/full` on current upstream and capture the exit status.
Done-When: Exit status, stderr, and versions are recorded.
