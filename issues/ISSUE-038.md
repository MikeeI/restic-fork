# ISSUE-038 — internal/fs: Windows extended attribute restore ignores the xattr filter

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: Medium
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] Windows EA restore runs only when the filtered snapshot set is non-empty and then clears every unexpected EA without applying `xattrSelectFilter`.

## Reach-and-Impact

Review correspondence: E18; severity: Low.
[S] Restoring a node without EAs leaves stale target EAs, unlike the Unix implementation.
[S] Restoring a node with EAs deletes target EAs excluded by `--exclude-xattr` or not matched by `--include-xattr`.
[A] Windows behavior was not executed.

## Evidence

- [S] `internal/fs/node_windows.go:74,82` — restore is skipped for empty sets.
- [S] `internal/fs/node_windows.go:170-181` — clearing loop without a filter.
- [S] `internal/fs/node_xattr.go:83-92` — Unix removes only filtered unexpected attributes.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-012 owns matcher construction performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Pass `xattrSelectFilter` to `restoreExtendedAttributes`, clear only matching EAs, and run clearing for empty sets.

## Scope-and-Constraints

- Preserve: the single `fsetEA` call required by Windows.
- Exclude: Unix xattr changes.
- Cost: requires Windows verification.

## Verification

- Windows test with existing EA `FOO`: restore with `--exclude-xattr FOO` keeps it, restoring a node without EAs removes it.

## Publication-Blockers

- Windows reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce Windows EA filtering
Action: Run the two EA restore scenarios on a Windows host with current upstream.
Done-When: Resulting EAs and versions are recorded.
