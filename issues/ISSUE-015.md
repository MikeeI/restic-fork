# ISSUE-015 — internal/repository/index: incremental reload decodes known index files before skipping them

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

[S] `MasterIndex.Load` checks `loadedIDs` inside the callback after `ForAllIndexes` has loaded and decoded the file.
[S] Known index results are discarded only after their expensive work has completed.

## Reach-and-Impact

[S] Mount initially loads the index, then initializes snapshot directories through another load of the same index.
[S] Later snapshot-hash changes can trigger further incremental loads; unchanged periodic checks do not always reload.
[S] Cache can avoid remote reads, but the repeated `LoadUnpacked` and decode calls still occur for known IDs.
[A] Startup delay, decoded allocation, and reload cost are unmeasured.

## Evidence

- [S] `internal/repository/index/master_index.go:283-331`: listing, known-ID accounting, and late skip at lines 308-310.
- [S] `internal/repository/index/index_parallel.go:26-33`: load and decode occur before callback invocation.
- [S] `cmd/restic/cmd_mount.go:152-155,189-192`: initial load and snapshot-root initialization.
- [S] `internal/fuse/snapshots_dirstruct.go:294-343`: hash-gated snapshot refresh calls `LoadIndex` at line 335.
- [S] `internal/repository/repository.go:716-727`: `LoadIndex` reuses the repository's existing master index.
- [S] `internal/repository/index/master_index.go:334-363`: a removed known ID resets the index for a full reload.
- [S] `internal/repository/index/master_index_test.go:521-563`: incremental checks count progress, not actual loader calls.

## Prior-Art

Coverage: all local records were screened for incremental index loading on 2026-10-02.
No local duplicate owns this cause; serialized checker pack hashing is a different root cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved; this was missing from the earlier review, not proven novel upstream.

## Proposed-Change

Apply the existing known-ID exclusion before `LoadUnpacked` and `DecodeIndex`, only for incremental master-index loads.
Preserve full loading for other consumers, callback serialization, new-index failures, and deleted-ID reset behavior.

## Scope-and-Constraints

- Keep initial mount loading and its fail-fast validation; do not merely delete that call.
- Keep index listing, merge completion, repository validation, cache preparation, and cancellation.
- Treat the exclusion set as immutable during parallel loading.
- Migrate every affected loader caller, including debug-only consumers, if the internal signature changes.
- Exclude parallel checker hashing and general index caching.
- Review correspondence: A1; no implementation or runtime gain has been observed.

## Verification

Planned, not run: update existing `TestMasterIndexIncrementalLoad` to count actual `LoadUnpacked` calls.
A no-op reload must perform zero known-index loads; adding one new index must perform one load and preserve contents.
This update reproduces the repeated-load defect; no new test is requested.
Run only the focused existing test after an authorized correction.

## Publication-Blockers

- No observed loader-count reproduction, mount profile, or implementation verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Count incremental index reloads
Action: Count actual index-file loads during one no-op reload of an already loaded master index.
Done-When: known IDs, actual load/decode counts, progress count, command, environment, and runtime are recorded.
