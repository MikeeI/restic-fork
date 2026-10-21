# ISSUE-031 — internal/data: large hour-based keep-within durations overflow and keep nothing

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

[S] `ApplyPolicy` computes `time.Hour * time.Duration(-Hours)` in int64 nanoseconds, which wraps above about 2.56 million hours.

## Reach-and-Impact

Review correspondence: E10; severity: Medium.
[S] The wrapped cutoff lies in the future, so `--keep-within` and `--keep-within-*` keep no snapshot.
[S] Combined with another keep policy, `forget --prune` deletes history the user intended to keep.
[A] Use of very large hour values as "forever" is unmeasured.

## Evidence

- [S] `internal/data/snapshot_policy.go:242,274` — cutoff arithmetic for both within paths.
- [S] `cmd/restic/cmd_forget.go:169-174` — validation rejects only negative components.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Reject hour components above `math.MaxInt64 / int64(time.Hour)` in `verifyForgetOptions`.

## Scope-and-Constraints

- Preserve: accepted duration syntax and policy semantics for valid ranges.
- Exclude: changing duration storage types.
- Cost: minimal.

## Verification

- `ApplyPolicy` with `Within{Hours: 3000000}` over recent snapshots → all snapshots are kept or the option is rejected.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce keep-within overflow
Action: Run `forget --dry-run --keep-within 9999999h --keep-last 1` on a disposable repository with current upstream.
Done-When: The keep and remove lists and versions are recorded.
