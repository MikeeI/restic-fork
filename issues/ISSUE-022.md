# ISSUE-022 — internal/repository: prune drops the last copy of a blob duplicated in an ignored missing pack

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: Medium
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `PlanPrune` removes blobs from `keepBlobs` when any copy lives in a pack outside `removePacks` and `repackPacks`.
[S] Missing-but-unneeded packs land in `ignorePacks`, so their copies count as kept although the pack file does not exist.

## Reach-and-Impact

Review correspondence: E1; severity: Critical.
[S] Precondition: the index lists a missing pack M with no used blobs, M holds an unused copy of used blob B plus another unused blob, and the used copy of B lives in a repacked pack R.
[S] `CopyBlobs` skips B, R is deleted, and the index rewrite drops M and R, so B has no remaining copy.
[S] The post-repack check `plan.keepBlobs.Len() != 0` cannot detect the loss because B was already removed from the set.
[A] How often indexes reference missing unneeded packs with such duplicates is unmeasured.

## Evidence

- [S] `internal/repository/prune.go:250-256` — duplicates initially count as `unusedBlobs` and `duplicateBlobs`.
- [S] `internal/repository/prune.go:305` — with M holding B and U, `duplicateBlobs(1) != unusedBlobs(2)`, so B becomes used in R in either iteration order.
- [S] `internal/repository/prune.go:476-484` — missing packs with `usedBlobs == 0` go to `ignorePacks`, not `removePacks`.
- [S] `internal/repository/prune.go:144-150` — the `keepBlobs` trim skips only `removePacks` and `repackPacks`.
- [S] `internal/repository/prune.go:626,663,672` — repack filters by `keepBlobs`, the index rewrite ignores M and R, and R is deleted.

## Prior-Art

Coverage: local ledger searched for prune blob-selection defects on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Also skip packs in `plan.ignorePacks` inside the `keepBlobs` trim at `prune.go:146`.

## Scope-and-Constraints

- Preserve: duplicate selection, repack limits, and the existing integrity check.
- Exclude: redesign of duplicate handling or missing-pack recovery.
- Cost: one condition; review must confirm `ignorePacks` is complete before the trim, which `decidePackAction` guarantees.

## Verification

- Repository test: store B in M (plus unused U) and R (plus used blobs and unused U2), delete M's pack file, run `PlanPrune` with `MaxUnused` 0 and `Execute` → `LoadBlob(B)` must still succeed.

## Publication-Blockers

- The data loss is source-proven only; a runtime reproduction is required.
- Upstream prior art is unchecked.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce prune blob loss
Action: Write a disposable repository test for the M/R duplicate scenario against current upstream and record the result.
Done-When: The test output shows whether B survives `PlanPrune` plus `Execute`, with Go version and revision recorded.
