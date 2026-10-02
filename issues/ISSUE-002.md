# ISSUE-002 — cmd/restic: recover loads every repository tree sequentially

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

Root-Cause [S]: `runRecover` resolves referenced trees by loading every tree blob of the repository in
one serial loop, although the set of trees is fixed before the loop and the loads are independent of
each other.

## Reach-and-Impact

Reach [S]: every `restic recover` invocation on a repository with tree blobs; the command prints the
loop size itself (`load %d trees`).
Impact [S]: wall time is the sum of all per-tree load latencies instead of a bounded fraction of it,
because no second load overlaps the first.
Impact [A]: on remote backends the serial round trips are assumed to dominate the command runtime; the
absolute duration and its share of total runtime are unmeasured here.

## Evidence

- [S] `cmd/restic/cmd_recover.go:88-111` — `for id := range trees { data.LoadTree(ctx, repo, id) }`
  loads and decodes exactly one tree per iteration.
- [S] `cmd/restic/cmd_recover.go:78-83` — `trees` is filled from all tree blobs in the index, so the
  key set is complete before the loop and does not need to change during it.
- [S] `internal/data/tree.go:145-151` — one `LoadBlob` per tree: backend/cache read, decryption,
  optional decompression, JSON decode.
- [S] `cmd/restic/cmd_recover.go:115` — the same function already uses the parallel
  `data.ForAllSnapshots`.
- [S] `internal/restic/parallel.go:56-81` — `ParallelRemove` bounds fan-out with `repo.Connections()`;
  `internal/data/tree_stream.go:204` uses the same pattern for tree loads.
- [S] `cmd/restic/cmd_forget.go:320` — snapshot removal in the neighbouring command is already
  parallelized.
- [S] Go specification, map iteration — an entry created during iteration may be skipped, so the
  current loop can leave subtrees of an added-referenced tree unmarked.

## Prior-Art

Coverage: issues(open+closed) titles and bodies, PRs(open+closed+merged), file history of
`cmd/restic/cmd_recover.go`; checked=2026-10-02. Gaps: none for this mechanism; searches for
`recover`, `restic recover slow`, and `parallel tree loading` returned only unrelated
recover-from-damage reports.

- `https://github.com/restic/restic/issues/22018` — Distinct; `recover` failure report without a
  loading-performance claim.
- `https://github.com/restic/restic/issues/5287` — Distinct; closed feature request for `recover`.
- `https://github.com/restic/restic/issues/1470` — Related precedent; parallelizing a serial
  traversal was accepted upstream for prune.

Contribution fit: New issue or pull request — no thread owns this root cause, the change is bounded to
one loop, and no active implementation owns it.

## Proposed-Change

Load the fixed tree set with a bounded worker pool (`errgroup` limit `repo.Connections()`), mark
subtree references under a mutex, and keep the progress counter plus the per-tree non-fatal error
reporting. Reference marking then no longer depends on map-iteration behavior.

## Scope-and-Constraints

- Preserve: root semantics of the documented intent (a tree is a root when nothing references it), the
  `trees` marks, progress output, `printer.E` per unreadable tree, and the snapshot written afterwards.
- Exclude: index repair, snapshot creation, and `restore`/`prune` code paths.
- Cost: worker state only; the reported root set can shrink by entries that the current loop may skip
  when the map is mutated during iteration.

## Verification

- `go test ./cmd/restic -run TestRecover` → the recovered snapshot contains the same tree structure.
- Counting test backend around `runRecover` → observed concurrent `LoadBlob` calls for trees and total
  load count equal to the tree count.
- Before/after comparison of `found %d unreferenced roots` on one fixture repository → identical or
  smaller, never larger.

## Publication-Blockers

- No measured wall time on a repository with many trees.
- No confirmation that upstream prefers a bounded `errgroup` over reusing `data.StreamTrees`.
- Authorized-Work and Publication-Target not selected.

## Next-Action

Summary: Measure recover tree-load runtime
Action: Time `restic recover` on a fixture repository with many trees and record tree count, wall
time, and load concurrency.
Done-When: the command, environment, counts, and timings are recorded in Evidence.
