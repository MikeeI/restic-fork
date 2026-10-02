# ISSUE-007 — internal/restorer: large-file downloads repeat index lookups for already planned blobs

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

[S] Large-file planning retains each blob ID, pack selection, and file offset.
[S] Download preparation nevertheless looks up the ID again to recover the same `DataBlob` handle.

## Reach-and-Impact

[S] The large-file path applies above 25 content chunks and repeats work for each planned occurrence.
[S] `LookupBlob` allocates and returns candidate index entries that the loop scans again.
[A] Total restore savings depend on chunk count and index size and are unmeasured.

## Evidence

- [S] `internal/restorer/filerestorer.go:138-150`: initial lookup and retained pack/blob/offset plan.
- [S] `internal/restorer/filerestorer.go:290-299`: repeated lookup and pack-candidate search.
- [S] `internal/repository/repository.go:681-687`: lookup result construction.
- [S] `internal/repository/repository.go:1076`: `LoadBlobsFromPack` remains the pack-reading validation boundary.

## Prior-Art

Coverage: the complete local ledger was screened for restore lookup repetition on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Construct the `DataBlob` handle directly from the planned blob ID instead of repeating the index lookup.
Keep the planned pack and every file offset unchanged.

## Scope-and-Constraints

- Preserve repeated chunks, duplicate file offsets, pack order, sparse handling, and error attribution.
- Keep pack membership and payload validation in the existing download path.
- Exclude new caches, plan structures, and changes to index lookup semantics.
- Review correspondence: P4.

## Verification

Planned, not run: `go test ./internal/restorer -run '^TestFileRestorerFrequentBlob$'` after an authorized fix.
A disposable restore must preserve contents and errors while avoiding repeated lookups for planned chunks.

## Publication-Blockers

- No observed lookup-count or restore verification for a correction.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure planned restore lookups
Action: Count index lookups during one disposable restore containing a file with more than 25 chunks.
Done-When: chunk count, repeated-chunk coverage, lookup count, command, environment, and runtime are recorded.
