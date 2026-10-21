# ISSUE-035 — internal/backend/sftp: failed saves leave temporary files after chmod or length errors

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `SFTP.Save` registers temp-file cleanup only after `Chmod`, and the length-mismatch path returns without setting `err`.

## Reach-and-Impact

Review correspondence: E14; severity: Medium.
[S] A failing `fsetstat` or short write leaves `*-restic-temp-*` files in repository directories.
[S] Each retry creates another random-suffix file, and `List` returns these names.

## Evidence

- [S] `internal/backend/sftp/sftp.go:365-370` — chmod failure returns before the cleanup `defer`.
- [S] `internal/backend/sftp/sftp.go:373-376` — cleanup runs only when `err` is non-nil.
- [S] `internal/backend/sftp/sftp.go:395-397` — the mismatch returns a new error while `err` stays nil.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Register cleanup immediately after `OpenFile` using a named error result assigned on every failure path.

## Scope-and-Constraints

- Preserve: atomic rename and read-only file handling.
- Exclude: listing filters for temporary names.
- Cost: minimal.

## Verification

- Use an SFTP server whose `fsetstat` fails, call `Save` → no temporary file remains.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check sftp temp-file prior art
Action: Search upstream issues and pull requests for leftover sftp temporary files.
Done-When: Search coverage and candidate classifications are recorded.
