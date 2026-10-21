# ISSUE-050 — cmd/restic: stats files-by-contents counts a directory as a unique file

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] Content-based stats modes hash every node, so directories and symlinks with empty content share one file ID.

## Reach-and-Impact

Review correspondence: E32; severity: Low.
[S] `Total File Count` is one higher when a snapshot contains directories but no empty regular file.
[S] Total size is unaffected.

## Evidence

- [S] `cmd/restic/cmd_stats.go:257-267` — no node type filter before counting.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-001 owns stats tree-load performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Count only `data.NodeTypeFile` nodes in the content-based modes.

## Scope-and-Constraints

- Preserve: restore-size and raw-data modes.
- Exclude: blob accounting changes.
- Cost: minimal.

## Verification

- Back up one non-empty file and one directory, run `restic stats --mode files-by-contents` → file count 1.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce stats file overcount
Action: Run `restic stats --mode files-by-contents` on a one-file, one-directory snapshot with current upstream.
Done-When: Reported counts and versions are recorded.
