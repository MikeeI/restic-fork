# ISSUE-032 — internal/data: FindAll resolves latest from a list mutated by earlier callbacks

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

[S] `FindAll` memorizes the snapshot list once and resolves `latest` lazily after earlier callbacks ran.
[S] `findLatest` loads every memorized snapshot and fails on any load error, including snapshots deleted by earlier callbacks.

## Reach-and-Impact

Review correspondence: E11; severity: Medium.
[S] `restic tag --add x <id> latest` rewrites `<id>`, deletes its old file, then fails while resolving `latest`.
[S] The command exits nonzero with the repository partially processed; reversing argument order succeeds.
[S] `rewrite` and `repair snapshots --forget` share the same callback pattern.

## Evidence

- [S] `internal/data/snapshot_find.go:135` — the list is memorized.
- [S] `internal/data/snapshot_find.go:148-155` — `latest` is resolved when reached.
- [S] `internal/data/snapshot_find.go:55-58` — load errors are fatal.
- [S] `cmd/restic/cmd_tag.go:112,162` — the old snapshot is removed inside the `FindAll` callback.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; ISSUE-010 owns prefix batching performance only.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Resolve and deduplicate `latest` before invoking any callback.

## Scope-and-Constraints

- Preserve: explicit ID order and filter warnings.
- Exclude: changing callback semantics.
- Cost: small restructuring of the argument loop.

## Verification

- With three snapshots, run `restic tag --add x <oldest> latest` → exit 0 and both snapshots tagged.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce tag latest ordering failure
Action: Run `restic tag --add x <oldest> latest` on a disposable repository with current upstream.
Done-When: Output, exit status, resulting tags, and versions are recorded.
