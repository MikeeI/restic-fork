# ISSUE-003 — internal/walker: TreeRewriter loads and decodes each tree twice

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

Root-Cause [S]: `TreeRewriter.RewriteTree` serializes the loaded tree as an encoding check, which
consumes the single-use iterator, and then calls `data.LoadTree` again — repeating blob fetch,
decryption, and decompression for a blob whose bytes are still held in memory by the live iterator.

## Reach-and-Impact

Reach [S]: `restic rewrite` paths that build `walker.NewSnapshotSizeRewriter`, i.e. `--include`,
`--exclude`, and `--snapshot-summary`; one extra load per tree of every rewritten snapshot.
`restic repair snapshots` sets `AllowUnstableSerialization` and is unaffected.
Impact [S]: tree-blob reads, decryptions, and decompressions are doubled for the rewriting traversal;
the JSON decode still happens twice because the iterator design keeps only one node alive.
Impact [A]: the absolute wall-time share depends on backend latency and tree size and is unmeasured
here.

## Evidence

- [S] `internal/walker/rewriter.go:121-127` — `data.SaveTree(ctx, saver, curTree)` consumes the first
  iterator for the encoding check.
- [S] `internal/walker/rewriter.go:129-134` — `curTree, err = data.LoadTree(ctx, loader, nodeID)`
  fetches and decodes the same tree a second time.
- [S] `internal/data/tree.go:49-53` — the iterator panics on second use (`tree iterator is single use
  only`).
- [S] `internal/data/tree.go:145-151` — `LoadTree` loads the complete blob into memory and decodes from
  it, so the first load's bytes remain available while that iterator is alive.
- [S] `internal/walker/rewriter.go:69-84` — `NewSnapshotSizeRewriter` leaves
  `AllowUnstableSerialization` false, so `restic rewrite` always takes the reload path.
- [S] `350f29d921cceff60bc0de06b9df0cec33182712` — commit message: "it is no longer possible to iterate
  over a tree multiple times. Instead it must be loaded a second time. This only affects the tree
  rewriting code." The reload is an acknowledged consequence of the iterator refactor, whose stated
  goal was lower peak memory, not a performance goal.

## Prior-Art

Coverage: issues(open+closed), PRs(open+closed+merged) via `gh` search, commit history of
`internal/walker/rewriter.go` and `internal/data/tree.go`; checked=2026-10-02. Gaps: none.

- `https://github.com/restic/restic/commit/350f29d921cceff60bc0de06b9df0cec33182712` — Related;
  introduces the reload and documents the tradeoff.
- `https://github.com/restic/restic/issues/21929` — Distinct; open `rewrite` feature request about
  empty directories.
- `https://github.com/restic/restic/issues/5702` — Distinct; open `rewrite` JSON output request.

Contribution fit: New pull request — the change is bounded to `RewriteTree`, upstream already
documented the reload as a refactor consequence, and no open thread owns it.

## Proposed-Change

Create both iterators from the same in-memory blob before the encoding check: load the blob bytes once
(`loader.LoadBlob`, or a small `data` helper that returns the raw tree) and pass
`data.NewTreeNodeIterator(bytes.NewReader(buf))` twice. Peak memory keeps the refactor's goal — one
blob buffer plus one live node — while the second fetch, decryption, and decompression disappear.

## Scope-and-Constraints

- Preserve: the encoding check and its error text, the `RewriteFailedTree` path, node order, the
  `replaces` cache, `KeepEmptyDirectory`, and peak memory per processed tree.
- Exclude: `TreeNodeIterator` semantics, `SaveTree`, and unrelated loader call sites.
- Cost: a second iterator over the same buffer; the node-list JSON decode remains duplicated.

## Verification

- `go test ./internal/walker -run 'TestRewriter|TestSnapshotSizeQuery|TestRewriterFailOnUnknownFields'`
  → unchanged behavior and error handling.
- Counting test backend around `RewriteTree` → one tree `LoadBlob` per tree instead of two.

## Publication-Blockers

- No measured per-tree cost and no maintainer confirmation that the duplicate decode is acceptable.
- Authorized-Work and Publication-Target not selected.

## Next-Action

Summary: Measure rewrite tree loads
Action: Run `restic rewrite --exclude` on a fixture repository with a counting backend wrapper and
record `LoadBlob` calls per tree.
Done-When: the observed load count per tree is recorded in Evidence with command and environment.
