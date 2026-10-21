# ISSUE-034 — cmd/restic: find --tree stops early on a cumulative hit counter

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

[S] `Finder.itemsFound` counts every tree hit across snapshots and paths and is compared with the number of distinct requested trees.

## Reach-and-Impact

Review correspondence: E13; severity: Medium.
[S] `find --tree` and tree-only `find --pack` stop walking snapshots early and omit later matches with exit 0.
[S] Example: S1 contains A, S2 contains A and B; `find --tree A B` misses B in S2.

## Evidence

- [S] `cmd/restic/cmd_find.go:378-383` — per-hit increment and early `errFindDone`.
- [S] `cmd/restic/cmd_find.go:696-701` — `errFindDone` continues to the next snapshot without resetting the counter.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-009 owns pattern preparation performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Track found tree IDs in a per-snapshot set and stop only when that set covers all requested trees.

## Scope-and-Constraints

- Preserve: early termination within one snapshot and blob search behavior.
- Exclude: output format changes.
- Cost: small.

## Verification

- Create S1 with tree A and S2 with A and B, run `find --tree A B` → B is reported for S2.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce find tree early stop
Action: Build the two-snapshot scenario and run `find --tree A B` on current upstream.
Done-When: Output and versions are recorded.
