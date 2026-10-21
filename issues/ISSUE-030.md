# ISSUE-030 — internal/restorer: later blobs write into the old target after file creation fails

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: Medium
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `file.inProgress` is set before `createFile` succeeds, so later blobs open the existing path without truncation or replacement.

## Reach-and-Impact

Review correspondence: E9; severity: Medium.
[S] When replacing a hardlinked or existing target fails, remaining blobs are written into the old inode.
[S] Hardlinked siblings that restic intends to protect can be modified, and other targets mix old and new content.
[A] Realistic triggers such as non-writable parent directories for non-root users are unmeasured.

## Evidence

- [S] `internal/restorer/filerestorer.go:378-385` — `inProgress` and `createSize` are set before the write result.
- [S] `internal/restorer/fileswriter.go:214-221` — `createSize < 0` opens the existing path.
- [S] `internal/restorer/fileswriter.go:112-139` — replacement failure returns before the old node is removed.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-018 owns blob reuse after successful hardlink replacement, not failed creation.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Record the creation error on `fileInfo` and make later blobs for that file return it without opening the path.

## Scope-and-Constraints

- Preserve: single creation and concurrent writes for successful files.
- Exclude: retrying creation.
- Cost: small locking change in the write path.

## Verification

- As non-root, restore over a hardlink pair in a non-writable parent → the sibling link must keep its content.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce failed-create blob writes
Action: Restore over a hardlinked target in a non-writable directory as non-root on current upstream.
Done-When: Sibling content, restore output, and versions are recorded.
