# ISSUE-037 — cmd/restic: mount leaves a dead FUSE mount when the initial snapshot load fails

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

[S] `runMount` mounts before loading snapshots and returns on a `ReadDirAll` error without unmounting or closing the connection.

## Reach-and-Impact

Review correspondence: E16; severity: Medium.
[S] After a load error or interruption during "Loading snapshots...", the mountpoint stays mounted without a server.
[A] Access then fails with ENOTCONN until manual `fusermount -u`; not observed here.

## Evidence

- [S] `cmd/restic/cmd_mount.go:178` — mount is established.
- [S] `cmd/restic/cmd_mount.go:192-195` — error return without cleanup.
- [S] `cmd/restic/cmd_mount.go:211-215` — the context path does unmount.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Unmount and close the connection on the `ReadDirAll` error path.

## Scope-and-Constraints

- Preserve: loading snapshots before announcing the mountpoint.
- Exclude: changes to the serve loop.
- Cost: minimal.

## Verification

- Interrupt `restic mount` during snapshot loading, then `mount | grep restic` → no remaining mount.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce stale mount
Action: Interrupt `restic mount` during snapshot loading on current upstream and inspect mounts.
Done-When: Mount table state, error output, and versions are recorded.
