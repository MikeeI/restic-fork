# ISSUE-046 — internal/fuse: symlinks and special files report zero link count

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

[S] `link.Attr` and `other.Attr` copy `node.Links` without the minimum of one applied to regular files.

## Reach-and-Impact

Review correspondence: E28; severity: Low.
[S] Non-POSIX snapshots with `Links` 0 expose symlinks and special files with `st_nlink` 0.
[S] The regular-file fix exists because tools such as Samba reject such stat results.

## Evidence

- [S] `internal/fuse/file.go:58-61` — regular files clamp to at least 1.
- [S] `internal/fuse/link.go:46` — symlinks do not clamp.
- [A] `internal/fuse/other.go:44` — reported without clamp by review, not read directly.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-014 owns FUSE name collision performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: likely a follow-up to the regular-file nlink fix.

## Proposed-Change

Apply `max(uint32(1), uint32(node.Links))` in `link.Attr` and `other.Attr`.

## Scope-and-Constraints

- Preserve: hardlink counts above one.
- Exclude: directory link counting.
- Cost: minimal.

## Verification

- Call `link.Attr` for a node with `Links` 0 → `Nlink` equals 1.

## Publication-Blockers

- The regular-file fix history and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Locate regular-file nlink fix
Action: Find the upstream change that added the regular-file nlink clamp and its discussion.
Done-When: The change URL and whether symlinks were considered are recorded.
