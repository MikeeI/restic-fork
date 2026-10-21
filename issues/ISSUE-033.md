# ISSUE-033 — internal/fs: non-root chown permission errors hide later metadata restore failures

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

[S] `nodeRestoreMetadata` keeps only the first error, which is the `lchown` permission error for non-root restores of foreign owners.
[S] `NodeRestoreMetadata` then discards that error, hiding later xattr, timestamp, and chmod failures.

## Reach-and-Impact

Review correspondence: E12; severity: Medium.
[S] Non-root restores of files owned by other users report success even when later metadata steps fail.
[S] The intended exemption covers ownership errors only.

## Evidence

- [S] `internal/fs/node.go:316-318` — the `lchown` error becomes `firsterr`.
- [S] `internal/fs/node.go:320-350` — later errors are kept only when `firsterr` is nil.
- [S] `internal/fs/node.go:302-305` — permission errors are discarded for non-root users.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Drop the non-root `lchown` permission error inside `nodeRestoreMetadata` instead of storing it as `firsterr`.

## Scope-and-Constraints

- Preserve: ignoring ownership permission errors for non-root users.
- Exclude: reporting every metadata error separately.
- Cost: minimal.

## Verification

- As non-root, restore a foreign-owned node with an xattr that fails for a non-permission reason → an error is reported.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce masked metadata errors
Action: Restore a foreign-owned node with a failing xattr as non-root on current upstream.
Done-When: Restore output, exit status, and versions are recorded.
