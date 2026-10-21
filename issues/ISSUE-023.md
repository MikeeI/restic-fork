# ISSUE-023 — cmd/restic: repair snapshots deletes snapshots with unreadable roots without --forget

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

[S] `filterAndReplaceSnapshot` removes the original snapshot whenever the filtered root is null and `keepEmptySnapshot` is false, without consulting `forget`.
[S] `repair snapshots` returns a null root for an unreadable `/` tree and passes `keepEmptySnapshot=false`.

## Reach-and-Impact

Review correspondence: E2; severity: Medium.
[S] Running `restic repair snapshots` without `--forget` deletes every snapshot whose root tree cannot be loaded.
[S] The deletion is intentional upstream design since `e17ee40a3` ("remove snapshots with invalid root node"), so this is a contract conflict with the documentation, not an accidental branch.
[A] Whether a transient load failure can reach this path after backend retries is unmeasured.

## Evidence

- [S] `cmd/restic/cmd_repair_snapshots.go:167-172` — the root failure returns `nil, nil`.
- [S] `cmd/restic/cmd_repair_snapshots.go:191-195` — passes `opts.Forget` and `keepEmptySnapshot=false`.
- [S] `cmd/restic/cmd_rewrite.go:208-222` — removal happens before and independent of the `forget` branch.
- [S] `doc/077_troubleshooting.rst:224-226` — without `--forget`, users must delete modified snapshots manually.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: likely a documentation or behavior discussion issue rather than a direct pull request.

## Proposed-Change

Remove the original only when `forget` is set; otherwise report the unreadable root and keep the snapshot.
Alternatively, document that unreadable-root snapshots are always removed.

## Scope-and-Constraints

- Preserve: `rewrite` empty-snapshot handling and `--dry-run` output.
- Exclude: changes to subtree replacement semantics.
- Cost: behavior change for users relying on current automatic cleanup.

## Verification

- Delete the pack holding a snapshot's root tree, then run `restic repair snapshots` without flags → the snapshot must remain listed.

## Publication-Blockers

- Maintainer intent must be confirmed through prior art.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check repair-root deletion prior art
Action: Search upstream issues and pull requests for unreadable-root deletion by repair snapshots.
Done-When: Search coverage and candidate classifications are recorded.
