# ISSUE-004 — internal/repository: CopyBlobs scans the full index for disabled debug output

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

[S] `CopyBlobs` evaluates `keepBlobs.Len()` before `debug.Log` checks whether logging is enabled.
[S] For the associated blob set used by `copy`, this count traverses the complete source index.

## Reach-and-Impact

[S] `copyTree` creates an associated blob set and invokes `CopyBlobs` for each copied snapshot.
[S] Even a small or empty selected set requires the debug count to visit the source index.
[S] Removing it eliminates one additional full scan; `copyStats` still scans through `Keys()`.
[A] Wall-time savings and their share of a multi-snapshot copy are unmeasured.

## Evidence

- [S] `internal/repository/repack.go:17-42`: private `repackBlobSet` requires `Len`; `CopyBlobs` computes it for logging.
- [S] `internal/debug/debug.go:164-166`: the enabled check occurs inside `debug.Log`, after argument evaluation.
- [S] `internal/repository/index/associated_data.go:145-185`: `Len` and `Keys` use the full-index `All` traversal.
- [S] `cmd/restic/cmd_copy.go:279,324-348`: per-snapshot associated set, `CopyBlobs`, and remaining `copyStats` traversal.

## Prior-Art

Coverage: all three existing records and the complete index were screened for this root cause on 2026-10-02.
No local duplicate owns this debug-only scan.
Gaps: upstream issue, pull request, and history searches for this root cause are not recorded.
Contribution fit remains unresolved; this record does not establish upstream novelty.

## Proposed-Change

Remove the unconditional blob count from the debug message and the then-unused `Len()` interface requirement.
Do not remove the concrete set's `Len` method or alter required copy statistics and batching.

## Scope-and-Constraints

- Preserve selected blobs, pack processing, cancellation, progress, and copied snapshot contents.
- Keep the `copyStats` size computation; suppressing it would also affect batching.
- Exclude a generic lazy-logging API and unrelated associated-set redesign.
- Review correspondence: P1; the correction does not eliminate every snapshots-times-index traversal.

## Verification

Planned, not run: instrument a disposable copy with debug disabled and record the extra count traversal.
A future fix must preserve copied contents while removing that traversal; the required statistics scan remains.

## Publication-Blockers

- No representative copy measurement or implementation verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure copy debug scan
Action: Measure the debug-count traversal during one disposable multi-snapshot copy with debug disabled.
Done-When: source-index size, snapshot count, traversal counts, command, environment, and runtime are recorded.
