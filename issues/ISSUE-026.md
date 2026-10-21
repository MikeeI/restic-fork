# ISSUE-026 — cmd/restic: diff reports success after subtree load failures

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

[S] Recursive `diffTree`, `printDir`, and `collectDir` failures are printed through `printError` and never returned.

## Reach-and-Impact

Review correspondence: E5; severity: High.
[S] `restic diff` exits 0 and prints statistics that omit unreadable subtrees.
[S] Scripts and `--json` consumers cannot distinguish a damaged comparison from a complete one.

## Evidence

- [S] `cmd/restic/cmd_diff.go:319-321,333-335,347-349` — subtree errors are only printed.
- [S] `cmd/restic/cmd_diff.go:203-207,233-237` — nested helpers swallow errors the same way.
- [S] `cmd/restic/cmd_diff.go:354` — `diffTree` returns only `ctx.Err()`.
- [S] The command help promises exit status 1 on any error.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Record the first subtree error in `Comparer` and return it as a fatal error after printing statistics.

## Scope-and-Constraints

- Preserve: continuing the diff after a subtree error and the printed statistics.
- Exclude: changes to diff output format.
- Cost: small.

## Verification

- Remove a pack holding a subtree of the second snapshot, run `restic diff` → exit status 1.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check diff-error prior art
Action: Search upstream issues and pull requests for diff exit status after tree load errors.
Done-When: Search coverage and candidate classifications are recorded.
