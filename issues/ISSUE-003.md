# ISSUE-003 — internal/walker: TreeRewriter reloads tree bytes after its serialization check

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

[S] Stable-serialization rewriting consumes a tree iterator for its encoding check, then loads the same blob again.
[S] Explicitly retaining the first load's raw bytes would avoid the second fetch, decryption, and decompression.
[S] Both JSON passes remain necessary with the current single-use iterator contract.

## Reach-and-Impact

[S] Rewrite paths using `NewSnapshotSizeRewriter` keep stable serialization enabled.
[S] A tree reaching the check-and-replay path incurs the extra load; excluded child subtrees need not reach it.
[S] Repetition across snapshots depends on rewriter cache policy and shared-tree occurrences, not every possible tree.
[S] `restic repair snapshots` enables `AllowUnstableSerialization` and does not use this second-load path.
[A] Absolute runtime savings and the blob-processing share are unmeasured.
Review correspondence: P13; the correction removes repeated blob processing, not the protective JSON passes.

## Evidence

- [S] `internal/walker/rewriter.go:69-84`: snapshot-size rewriting keeps stable serialization enabled.
- [S] `internal/walker/rewriter.go:101,117-134`: initial tree load, encoding/hash check, and second load.
- [S] `internal/data/tree.go:49-53`: a consumed iterator cannot be reused.
- [S] `internal/data/tree.go:145-151`: `LoadTree` loads blob bytes and creates an iterator from them.
- [S] `350f29d921cceff60bc0de06b9df0cec33182712` explains the iterator-refactor reload.
  Its exact rationale is: "it is no longer possible to iterate over a tree multiple times. Instead it must be loaded a second time. This only affects the tree rewriting code."
  The refactor targeted lower peak memory, which a correction must preserve.

## Prior-Art

Recorded coverage: upstream issue/PR searches and touched-file history checked on 2026-10-02.
Gaps: lexical coverage does not prove the absence of active related work, and performance remains unmeasured.

- Related: https://github.com/restic/restic/commit/350f29d921cceff60bc0de06b9df0cec33182712 introduces and explains the reload.
- Distinct: https://github.com/restic/restic/issues/21929 concerns empty-directory rewriting.
- Distinct: https://github.com/restic/restic/issues/5702 concerns rewrite JSON output.

Contribution fit may be a bounded pull request after verification and active-work checking; no target is selected.

## Proposed-Change

Retain the raw buffer from the first verified blob load and build a fresh iterator only after the encoding check succeeds.
Use `data.NewTreeNodeIterator(bytes.NewReader(buf))` for each pass rather than reusing a consumed iterator.
Keep one buffer and bounded live-node state; do not materialize all nodes.

## Scope-and-Constraints

- Preserve encoding/hash checks, unknown-field rejection, error text, and the `RewriteFailedTree` path.
- Preserve node order, cache policy, excluded subtrees, `KeepEmptyDirectory`, and cancellation before replay.
- Preserve peak-memory intent; the second JSON decode remains.
- Exclude changes to iterator semantics, `SaveTree`, and unrelated loader callers.

## Verification

Planned, not run: existing `TestRewriter`, `TestRewriterFailOnUnknownFields`, and `TestRewriterTreeLoadError`.
After an authorized correction, run `go test ./internal/walker -run '^(TestRewriter|TestRewriterFailOnUnknownFields|TestRewriterTreeLoadError)$'`.
A disposable rewrite must retain output and errors while performing one blob load per affected tree processing.
No new test is requested.

## Publication-Blockers

- No observed per-tree load count or representative runtime measurement.
- No implementation, memory, or cancellation verification.
- Upstream active-work coverage must be refreshed before contribution.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure rewrite tree loads
Action: Count blob loads for trees reaching stable check-and-replay in one disposable rewrite.
Done-When: command, environment, cache policy, affected tree occurrences, load counts, and runtime are recorded.
