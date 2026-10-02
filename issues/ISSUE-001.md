# ISSUE-001 — cmd/restic: stats loads every tree of every snapshot sequentially

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Stats walks snapshot trees serially and revisits shared subtrees for each snapshot occurrence.
[S] The source proves repeated loading and decoding, not that remote round trips dominate current runtime.

## Reach-and-Impact

[S] Node-walking modes are `restore-size` (default), `files-by-contents`, and `blobs-per-file`.
[S] `raw-data` already uses parallel `data.FindUsedBlobs` and is outside this proposed traversal correction.
[S] Total loads follow visited tree occurrences, not a fixed trees-per-snapshot product for arbitrary inputs.
[S] Metadata caching can avoid or coalesce remote downloads; cached loads still involve local blob processing.
[A] Current wall time and the loading stage's share of total runtime are unmeasured.
Review correspondence: P14 was dismissed as an actionable parallelization recommendation, not invalidated as a serial source mechanism.

## Evidence

- [S] `cmd/restic/cmd_stats.go:151-156`: the snapshot loop invokes `statsWalkSnapshot` serially.
- [S] `cmd/restic/cmd_stats.go:237-250`: per-snapshot hardlink state, ordered node processing, and fatal traversal errors.
- [S] `internal/walker/walker.go:88-98`: directory traversal loads a subtree before visiting its descendants.
- [S] `internal/data/tree.go:145-151`: tree loading performs blob processing and creates a single-use iterator.
- [S] `cmd/restic/cmd_stats.go:234`: `raw-data` delegates to `data.FindUsedBlobs`.
- [S] `internal/backend/cache/backend.go:49-55,94-106,153-195`: metadata caching and coalesced pack downloads.
- [S] `internal/data/tree_stream.go:128,187-230`: bounded workers include a single-worker path for trees over 50 MiB.
- [S] `internal/data/find.go:15-23`: unique-tree skipping is valid for blob reachability, not automatically for stats counters.

## Prior-Art

Recorded coverage: upstream issues, pull requests, and touched-file history checked on 2026-10-02.
The earlier claim of no relevant search result contradicted the matching stats thread below.
Gaps: lexical coverage is not exhaustive, and historical reports do not establish the current loading-cost share.

- Related/Duplicate: https://github.com/restic/restic/issues/2126 owns slow stats traversal.
  A maintainer comment dated 2022-08-20 names walking each snapshot and `StreamTrees`.
  Recorded reports include 81–131 minute `restore-size` runs and a separate CPU/memory report.
  These reports predate the current traversal and are not measurements of this checkout.
- Related: https://github.com/restic/restic/issues/693 proposes per-snapshot size metadata rather than tree walking.
- Related precedent: https://github.com/restic/restic/issues/1470 concerns parallel prune traversal.

Contribution fit remains a possible comment on the existing stats thread, not a selected publication target.

## Proposed-Change

Defer parallelization until a current profile shows a material loading-stage bottleneck.
If justified, consider bounded prefetch with serial visitation rather than assuming a shared mutex preserves semantics.
Do not skip snapshot tree occurrences merely because unique-tree skipping is safe for blob reachability.

## Scope-and-Constraints

- Preserve all four modes, per-snapshot hardlink state, global unique-file state, and visit order.
- Preserve fatal traversal errors; unreadable trees are not generally non-fatal in this command.
- Preserve progress reset and counting semantics.
- Any later prefetch design must bound bytes, handle huge trees, cancel and drain workers, and retain error precedence.
- Exclude snapshot metadata and repository-format changes.
- Do not alter shared walker callers such as `ls`, `find`, and `dump` without resolving their ordered-output contracts.

## Verification

Planned, not run: profile default stats on a representative multi-snapshot repository with cache state recorded.
Separate index loading, cache/backend work, decryption/decompression, decoding, and node-counter processing.
Only an authorized correction would require focused affected tests and CLI output comparisons across all four modes.
No new test or source implementation is part of this ledger update.

## Publication-Blockers

- No current representative profile proving material tree-load cost.
- No complete prefetch memory, cancellation, ordering, or error contract.
- Historical timing reports do not close these gaps.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure stats tree-load cost
Action: Profile default stats on a representative multi-snapshot repository with its cache state recorded.
Done-When: command, environment, visited trees, stage timings, and cache/backend activity are recorded.
