# ISSUE-043 — cmd/restic: cache --cleanup returns success after removal failures

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

[S] The cleanup loop prints `RemoveAll` failures and returns nil unconditionally.

## Reach-and-Impact

Review correspondence: E23; severity: Low.
[S] Scheduled cleanup jobs report success while old cache directories remain.

## Evidence

- [S] `cmd/restic/cmd_cache.go:98-101` — errors are printed only.
- [S] `cmd/restic/cmd_cache.go:104` — `return nil`.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Remember the first removal error and return it after the loop.

## Scope-and-Constraints

- Preserve: attempting every directory.
- Exclude: changes to cache age selection.
- Cost: minimal.

## Verification

- Make one old cache directory immutable, run `restic cache --cleanup` → exit 1.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce cache cleanup exit
Action: Run `restic cache --cleanup` with an immutable old cache directory on current upstream.
Done-When: Exit status, output, and versions are recorded.
