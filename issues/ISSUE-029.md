# ISSUE-029 — internal/restorer: partially written files receive snapshot metadata

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

[S] The restorer tracks files before content is written and does not record per-file content failures.
[S] The second pass restores metadata, including mtime, for every tracked file.

## Reach-and-Impact

Review correspondence: E8; severity: Medium.
[S] After a tolerated blob load error, the target keeps its preallocated full size and receives the snapshot mtime.
[S] A rerun with `--overwrite if-changed` trusts size plus mtime and keeps the corrupt content.
[S] The first run reports an error, so the damage persists only through the resume workflow.

## Evidence

- [S] `internal/restorer/restorer.go:421` — `trackFile` runs in the first pass.
- [S] `internal/restorer/filerestorer.go:354-360` — blob errors are passed to `sanitizeError` per file.
- [S] `internal/restorer/restorer.go:461-463` — metadata is restored for tracked files without a failure check.
- [S] `internal/restorer/restorer.go:739-741` — `trustMtime` accepts size plus mtime matches.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-016 through ISSUE-019 and ISSUE-021 own distinct restorer root causes.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Mark files whose content restoration failed and skip their metadata in the second pass.

## Scope-and-Constraints

- Preserve: tolerated per-file errors and metadata for successfully restored files.
- Exclude: rollback of partially written content.
- Cost: moderate; requires failure state propagation from `fileRestorer`.

## Verification

- Restore with a deleted data pack, then rerun with `--overwrite if-changed` → the affected file must be rewritten.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce partial-file mtime reuse
Action: Restore with a missing pack, rerun with `--overwrite if-changed`, and inspect content on current upstream.
Done-When: Both runs' output, file content check, and versions are recorded.
