# ISSUE-047 — internal/fuse: single-tag path templates skip tag filename sanitization

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

[S] The `%t` template branch sanitizes tags with `filenameFromTag` only when a snapshot has multiple tags.

## Reach-and-Impact

Review correspondence: E29; severity: Low.
[S] A sole tag `..` places the snapshot outside `tags/`, and tags containing `/` create nested pseudo-directories.
[S] The same tag renders differently depending on whether other tags exist.

## Evidence

- [S] `internal/fuse/snapshots_dirstruct.go:105` — multi-tag branch sanitizes.
- [S] `internal/fuse/snapshots_dirstruct.go:113` — single-tag branch uses `sn.Tags[0]` raw.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-014 owns FUSE name collision performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Use `filenameFromTag(sn.Tags[0])` in the single-tag branch.

## Scope-and-Constraints

- Preserve: existing names for ordinary tags.
- Exclude: sanitizing `%h` and `%u`.
- Cost: minimal.

## Verification

- `pathsFromSn` with tags `[".."]` → the path stays under `tags/`.

## Publication-Blockers

- Upstream prior art is missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check FUSE tag sanitization prior art
Action: Search upstream issues and pull requests for FUSE tag path sanitization.
Done-When: Search coverage and candidate classifications are recorded.
