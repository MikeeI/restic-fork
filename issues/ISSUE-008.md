# ISSUE-008 — internal/repository: prune traverses index entries separately to calculate pack headers

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Prune first visits pack entries to calculate header sizes, then visits the same entries for classification.
[S] Header size is an aggregation of format-owned entry sizes and one fixed header base per pack.

## Reach-and-Impact

[S] Pack discovery performs the additional pass across the index and retains a temporary header-size map.
[A] The pass's share of complete prune runtime and memory consumption is unmeasured.

## Evidence

- [S] `internal/repository/prune.go:220-228`: separate `pack.Size` aggregation.
- [S] `internal/repository/prune.go:232-279`: subsequent blob classification.
- [S] `internal/repository/pack/pack.go:423-457`: format entry sizes, header base, and size calculation.

## Prior-Art

Coverage: the complete local ledger was screened for prune header aggregation on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Aggregate header sizes during the existing classification pass, using size logic owned by `pack`.
Count every stored entry even when its classification takes an early branch.

## Scope-and-Constraints

- Preserve compressed and mixed-pack sizes, missing-blob handling, duplicate selection, and prune decisions.
- Keep format constants and entry-size rules in `pack`; do not duplicate them in prune.
- Exclude repository format, integrity checks, and unrelated prune policy changes.
- Review correspondence: P5.

## Verification

Planned, not run: `go test ./internal/repository -run '^TestPrune$'` after an authorized fix.
Compare pack decisions and size statistics while confirming the separate header pass is gone.

## Publication-Blockers

- No observed prune measurement or correction verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure prune header pass
Action: Count index-entry visits and profile header aggregation in one disposable prune dry run.
Done-When: pack count, entry count, pass counts, command, environment, and runtime are recorded.
