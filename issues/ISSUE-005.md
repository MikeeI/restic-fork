# ISSUE-005 — internal/backend/azure: single uploads allocate an additional full payload buffer

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

[S] The single-upload branch allocates `rd.Length()` bytes, reads the entire payload, then uploads a byte reader.
[S] The built-in byte and file readers are already seekable, so these inputs do not require that additional buffer.

## Reach-and-Impact

[S] Azure saves up to `singleUploadMaxSize` (256 MiB) use this branch, including ordinary pack uploads.
[S] The additional buffer is proportional to each active payload and is separate from caller-owned storage.
[A] Actual concurrent peak allocation and job-level savings are unmeasured.
[S] The required hash pass in `savePacker` remains; removing the Azure buffer is not zero-copy backup.

## Evidence

- [S] `internal/backend/azure/azure.go:263-280`: allocation, `io.ReadFull`, MD5 validation, and SDK upload.
- [S] `internal/backend/rewind_reader.go:12-24`: `RewindReader` does not require `io.Seeker`.
- [S] `internal/backend/rewind_reader.go:28-31,75-80`: built-in readers expose seekable underlying readers.
- [S] Azure SDK `azblob v1.8.0`, `blockblob/client.go:171-194`: `Upload` requires `io.ReadSeekCloser` and validates the stream.
- [S] `internal/repository/packer_manager.go:244-270`: required hash read, rewindable file reader, save, and caller-owned close.

## Prior-Art

Coverage: the complete local ledger was screened for Azure upload buffering on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Upload seekable readers directly through the SDK's no-op closer.
Retain the required buffer path for valid readers that only implement `RewindReader`.
Keep length validation, transactional MD5, retry rewind, and caller-owned reader cleanup.

## Scope-and-Constraints

- Preserve the `RewindReader` input contract; do not require every caller to implement `Seek`.
- Do not let SDK cleanup close the caller-owned pack file.
- Preserve short-read and upload failures; do not silently accept inconsistent lengths or hashes.
- Exclude multipart substitution, new buffering configuration, and removal of the pack hash pass.
- Review correspondence: P2, corrected for the nonseekable interface contract.

## Verification

Planned, not run: Azure Save smoke scenario with a seekable reader and a valid nonseekable rewind reader.
Confirm uploaded contents, MD5, retry rewind, close ownership, and removal of the additional buffer for seekable input.
Use a disposable Azure test target; any future check must cover the actual upload boundary.

## Publication-Blockers

- No observed upload, retry, or heap verification for a change.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure Azure upload allocation
Action: Profile one representative single-pack Azure save with a seekable reader on a disposable target.
Done-When: payload size, concurrent saves, heap allocation, exact command, environment, and runtime are recorded.
