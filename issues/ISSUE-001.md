# ISSUE-001 — cmd/restic: stats loads every tree of every snapshot sequentially

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

Root-Cause [S]: the stats traversal has neither a parallel tree-loading stage nor a skip set for
already-walked subtrees, so every tree blob is fetched and decoded strictly one at a time and shared
subtrees are decoded again for each snapshot that references them.

## Reach-and-Impact

Reach [S]: `restic stats` in all modes that walk nodes — `restore-size` (default),
`files-by-contents`, `blobs-per-file` — for every snapshot matching the filter (default: all
snapshots). Mode `raw-data` already uses the parallel `data.FindUsedBlobs`.
Impact [S]: total tree loads = snapshots × trees per snapshot, executed serially, each with one
backend/cache read plus decryption and optional decompression plus JSON decode.
Impact [A]: the resulting wall time and its share of total runtime are unmeasured here; per-tree cost
on remote backends is assumed to be round-trip dominated.

## Evidence

- [S] `cmd/restic/cmd_stats.go:151-156` — the snapshot loop calls `statsWalkSnapshot` serially.
- [S] `cmd/restic/cmd_stats.go:238` — `statsWalkSnapshot` walks each snapshot with `walker.Walk`.
- [S] `internal/walker/walker.go:88-98` — each directory is entered by loading exactly one subtree and
  recursing immediately; no lookahead and no skip set exist.
- [S] `internal/data/tree.go:145-151` — `LoadTree` performs one `LoadBlob` (backend/cache read,
  decryption, optional decompression) plus a JSON decode.
- [S] `cmd/restic/cmd_stats.go:234` — `raw-data` delegates to `data.FindUsedBlobs`.
- [S] `internal/data/tree_stream.go:204` — `StreamTrees` already runs `Connections()+GOMAXPROCS+1`
  tree-load workers and takes a `skip` predicate.
- [S] `internal/data/find.go:15-23` — `FindUsedBlobs` skips already-referenced trees, which is why
  `prune` processes each tree part once while `stats` does not.
- [A] Per-tree latency on remote backends is dominated by the backend round trip; unmeasured.

## Prior-Art

Coverage: issues(open+closed), PRs(open+closed+merged), commit history of the touched files;
checked=2026-10-02. Gaps: none; keyword-searching `recover`, `rewrite`, `parallel tree loading` and
`stats slow walk tree` returned only unrelated findings.

- `https://github.com/restic/restic/issues/2126` — Related/Duplicate; open with labels
  `category: stats` and `category: optimization`. Maintainer comment 2022-08-20 states the same root
  cause ("has to walk the whole tree for each snapshot") and names `StreamTrees` as the direction.
  Three reporters document runtimes of 81–131 minutes for `restore-size`.
- `https://github.com/restic/restic/issues/693` — Related; proposes reading per-snapshot size metadata
  instead of walking trees. Different root cause.
- `https://github.com/restic/restic/issues/1470` — Related precedent; serial prune traversal was
  parallelized upstream.

Contribution fit: `https://github.com/restic/restic/issues/2126` — comment adding mechanism evidence
(the anchors above), the constraint that `StreamTrees` carries no node path for `blobs-per-file`, and
the warning that merging per-worker containers is not equivalent for the unique-file counters.

## Proposed-Change

Give the stats traversal a parallel tree-loading stage without changing visit order or counting
semantics: either bounded subtree prefetching inside `walker.Walk`, or a `StreamTrees`-based traversal
for the modes that do not need node paths combined with a mutex-shared statistics container.

## Scope-and-Constraints

- Preserve: output of all four modes, progress counter semantics (`statsui.Progress` is mutex
  protected; `ProcessSnapshot` resets per-snapshot counters), and non-fatal reporting for unreadable
  trees.
- Exclude: snapshot metadata, repository format, and `StreamTrees`/`walker` signature changes that
  would affect `prune`, `copy`, `ls`, `find`, or `dump`.
- Cost: `walker.Walk` is shared with `ls`, `find`, and `dump`, which require ordered progressive
  output; a prefetch stage must not reorder visits or delay node callbacks.

## Verification

- `go build ./... && go test ./cmd/restic ./internal/walker ./internal/data` → build and existing
  tests pass.
- `restic stats --json --mode <mode>` for all four modes on one fixture repository → identical field
  values before and after.
- `time restic stats --mode restore-size` on a multi-snapshot fixture → recorded wall time before and
  after.

## Publication-Blockers

- No measurement of current and proposed wall time on a representative multi-snapshot repository.
- No maintainer decision on the acceptable traversal change (`walker.Walk` prefetch versus
  `StreamTrees` per mode).
- Authorized-Work and Publication-Target not selected.

## Next-Action

Summary: Measure stats tree-load cost
Action: Run `restic stats` per mode on a fixture repository with many snapshots and record tree loads
and wall time.
Done-When: the exact commands, environment, load counts, and timings are recorded in Evidence.
