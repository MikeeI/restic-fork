# ISSUE-025 — internal/fs: ownership-by-name restores unresolvable owners as root

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `lookupUid` and `lookupGid` return 0 for any lookup failure, including empty names, and cache that 0.
[S] `lchown` passes the result to `os.Lchown` without falling back to the recorded numeric IDs.

## Reach-and-Impact

Review correspondence: E4; severity: High.
[S] Root restores with `--ownership-by-name` assign `root:root` to every node whose user or group is missing on the target host.
[S] `chmod` runs after `lchown`, so setuid or setgid files owned by unmapped accounts become setuid-root or setgid-root.
[S] No error or warning is emitted because the chown to 0 succeeds.

## Evidence

- [S] `internal/fs/node_unix.go:11-22` — lookups feed `os.Lchown` directly.
- [S] `internal/fs/node.go:141-163,199-221` — IDs default to 0 and failures are cached.
- [S] `internal/fs/node.go:344-349` — mode bits are applied after ownership.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream search not performed; feature origin reported as https://github.com/restic/restic/pull/5449 but not inspected.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Make the lookups return `(id, ok)` and fall back to `node.UID`/`node.GID` when a name is empty or unresolvable.
Do not cache failed lookups as 0.

## Scope-and-Constraints

- Preserve: name-based mapping when names resolve and numeric restore without the flag.
- Exclude: new warnings or flags.
- Cost: small; maintainers may prefer an error over a numeric fallback.

## Verification

- As root, restore a file owned by a user absent on the target with `--ownership-by-name` → owner must not be root.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce ownership-by-name root fallback
Action: Restore a snapshot with an unknown owner as root using `--ownership-by-name` on current upstream.
Done-When: Resulting owner, command, and versions are recorded.
