# ISSUE-002 — cmd/restic: recover loads every repository tree sequentially

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

[S] `runRecover` loads indexed tree blobs in a serial loop to collect subtree references.
[S] Reference marking can insert missing subtree IDs into the map, but insertion does not prove missed recoverable roots.

## Reach-and-Impact

[S] Recovery reaches the scan after index repair when indexed tree blobs exist and prints `load %d trees`.
[S] Tree loads do not overlap within that loop.
[S] Metadata caching can avoid remote downloads, and index repair is another potentially substantial command stage.
[A] The scan's absolute duration and share of complete recovery runtime are unmeasured.
Review correspondence: P15 was dismissed as an actionable worker-pool recommendation; the mechanism remains Investigating.

## Evidence

- [S] `cmd/restic/cmd_recover.go:78-110`: index-derived tree map, serial loads, subtree marking, and distinct error paths.
- [S] `cmd/restic/cmd_recover.go:96-102`: blob-load failures are reported and skipped; iterator errors abort recovery.
- [S] `internal/data/tree.go:145-151`: tree loading processes the blob and constructs its iterator.
- [S] `internal/repository/repository.go:252-255`: `LoadBlob` cannot decode a missing indexed subtree merely inserted into the map.
- [S] `internal/backend/cache/backend.go:49-55,94-106`: metadata cache and coalesced downloads weaken a round-trip assumption.
- [S] `internal/data/tree_stream.go:128,187-230`: trees over 50 MiB use one dedicated huge-tree worker.
- [S] Go map iteration may omit newly inserted entries: https://go.dev/ref/spec#For_statements.
  This language rule alone does not establish a recovery correctness defect or a smaller correct root set.

## Prior-Art

Recorded coverage: upstream issue/PR searches and history of `cmd/restic/cmd_recover.go` checked on 2026-10-02.
Search terms included `recover`, `restic recover slow`, and `parallel tree loading`.
Gaps: lexical coverage is not exhaustive, and no current workload measurement establishes correction value.

- Distinct: https://github.com/restic/restic/issues/22018 reports recovery failure without this loading-performance mechanism.
- Distinct: https://github.com/restic/restic/issues/5287 is a closed recovery feature request.
- Related precedent: https://github.com/restic/restic/issues/1470 concerns parallel traversal.

Contribution fit is unresolved; the previous claim of a ready bounded change was too strong.

## Proposed-Change

Defer worker-pool design until profiling separates tree scanning from index repair and other recovery stages.
Any later design must process the initial indexed IDs and serialize reference publication with explicit lifecycle ownership.
Do not substitute recursive `StreamTrees` blindly; its scheduling and huge-tree policy differ from this scan.

## Scope-and-Constraints

- Preserve the recovered root set exactly; a smaller set is not an accepted performance-only outcome.
- Preserve missing-reference bookkeeping, successful-load progress, diagnostics, and snapshot contents.
- Preserve non-fatal blob-load failures versus fatal iterator failures.
- Resolve huge-tree memory, shared-map safety, cancellation, worker draining, and error precedence before implementation.
- Exclude changes to index repair, snapshot creation, and `restore` or `prune`.

## Verification

Planned, not run: profile recovery against a disposable repository with recorded cache state.
Separate index repair and tree scanning; record indexed IDs, actual loads, concurrency, peak memory, and errors.
Only an authorized correction would require existing `TestRecover` and a recovery smoke comparison with identical roots.
No new test or source implementation is part of this ledger update.

## Publication-Blockers

- No current profile proving the tree scan dominates recovery.
- No complete worker-pool memory, cleanup, and error contract.
- The map-insertion argument does not establish a recovery correctness defect.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure recover tree-load runtime
Action: Profile recovery on one disposable repository, separating index repair from tree scanning.
Done-When: command, environment, cache state, stage timings, load counts, memory, and error behavior are recorded.
