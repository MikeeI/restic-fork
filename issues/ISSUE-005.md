# ISSUE-005 — internal/backend/azure: single uploads allocate an additional full payload buffer

State: PR-Ready
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-05
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Single uploads allocate `rd.Length()` bytes and copy the entire input before calling the SDK.
[S] Ordinary byte and file readers already support the seekable upload interface.
[O] Reusing that input safely also required retiring HTTP request readers before SDK rewinds or caller cleanup.
[O] An early-503 race between SDK rewind and the previous HTTP writer was reproduced on unchanged upstream.
This race is not attributed to the optimization and does not establish repository corruption.

## Reach-and-Impact

[S] Azure saves up to `singleUploadMaxSize` (256 MiB), including ordinary pack uploads, use this branch.
[O] One 8-MiB seekable Save allocated 8,481,597–8,492,794 B upstream and 85,605–85,717 B with the final policy.
[O] Valid nonseekable readers retained the approximately payload-sized buffered allocation.
[A] Concurrent peak RSS, production Azure performance, and job-level savings remain unmeasured.
[S] The pack hash pass remains; this is not zero-copy backup.

## Evidence

- [S] Baseline `internal/backend/azure/azure.go:263-280`: allocation, `ReadFull`, MD5 validation, and upload.
- [S] `internal/backend/rewind_reader.go:12-24,28-31,75-80`: seekability is optional; built-in readers provide it.
- [S] `internal/repository/packer_manager.go:244-270`: stable flushed input, hash read, Save, and caller-owned close.
- [S] Pinned SDKs: `azblob v1.8.0`, `azcore v1.23.1`, SDK `internal v1.12.0`.
- [S] `azcore/runtime/pipeline.go:79-85`: SDK retry precedes custom per-retry policies and the body-download policy follows them.
- [S] The body-download policy preserves SDK response caching and download-error translation before input retirement.
- [O] Real Put Blob and downloads were verified against Azurite 3.35.0, pinned image digest `647c63a91102a9d8e8000aab803436e1fc85fbb285e7ce830a82ee5d6661cf37`.
- [O] Tests used a disposable loopback account/container and a valid Hot access tier; no production Azure credentials were used.
- [O] Environment: Ubuntu 24.04.5, Linux amd64, Go 1.27.1, AMD Ryzen 9 5950X; one Save per benchmark iteration.
- [O] Final allocation command: `go test ./internal/backend/azure -run '^$' -bench '^BenchmarkIssue005PolicySave$' -benchtime=3x -count=3`.
- [O] Final benchmark runtime was 38.815–43.494 ms/op; allocation reduction, not throughput, is the measured contribution claim.

## Prior-Art

Coverage: local index, targeted upstream issue/PR searches, relevant diffs, SDK sources, and introduction history.
The buffer belongs to the merged single-Put-Blob change intended to reduce Azure transactions.
This is a compatible follow-up, not a request to replace single uploads with multipart uploads.
Related issue: https://github.com/restic/restic/issues/5531.
Related merged implementation: https://github.com/restic/restic/pull/5544.
Historical PR 3407 used an older SDK and was not merged: https://github.com/restic/restic/pull/3407.
The inspected SDK-update proposal changes dependencies, not this upload implementation: https://github.com/restic/restic/pull/22088.
No same-root implementation was identified in inspected work; searches were bounded rather than exhaustive.
Revalidate SDK ordering if the dependency update is merged before this contribution.

## Proposed-Change

Upload a seekable input directly only when its current offset is zero and its seek-reported size equals `Length()`.
Restore the original zero position after probing; reject a failed restoration.
Retain `ReadFull` buffering for nonseekable inputs, consumed or declared-prefix inputs, and failed eligibility probes.
Use the SDK no-op closer so request cleanup never closes the caller-owned input.
Install a per-retry policy that retires each HTTP reader after SDK response download completes.
Retire native `GetBody` readers before rewinding; reject late reads and late rewinds of retired attempts.

## Scope-and-Constraints

- Preserve optional seekability, declared-length buffering behavior, transactional MD5, nil-hash behavior, and retries.
- Inputs selected for direct upload must remain stable and repeatedly seekable throughout Save and its retries.
- Source read errors now arise within Upload rather than the preliminary ReadFull on the direct path.
- The guard waits for an active Read; it does not promise prompt cancellation of an arbitrary blocked custom reader.
- Preserve native nil-transport behavior and bodyless streaming downloads.
- Multipart buffer reuse is protected by the same lifecycle policy; its upload strategy and block sizes remain unchanged.
- Exclude new buffering settings, SDK upgrades, removing the hash pass, or changes to the access-tier contract.
- Test decision: none; existing tests plus disposable public-boundary checks were used without permanent new tests.
- Rename the descriptive changelog file to the PR number after publication assigns one.

## Verification

- [O] Baseline/fix Azurite smoke covered byte, file, nonseekable, empty, consumed-prefix, and declared-prefix inputs.
- [O] Short declared inputs failed with `ReadFull: unexpected EOF`; incorrect MD5 was rejected by the service.
- [O] Failed seek probes used buffering; a failed end probe restored position before reading.
- [O] Fully consumed 503 retries succeeded and preserved content; original file ownership remained with the caller.
- [O] An unchanged-upstream early-503 race was reproduced under `-race`, with SDK Seek racing HTTP Read.
- [O] Final `go test -race ./internal/backend/azure -run '^TestIssue005Policy(Lifecycle|NativeRetry)$' -count=3 -timeout=90s -v` passed.
- [O] Lifecycle cases used actual HTTP/1 and TLS HTTP/2, bytes/files, early 503, delayed nonempty responses, and file cancellation.
- [O] Native GetBody rewind with nil MD5 preserved downloaded bytes and rejected reads on the retired attempt.
- [O] `go test ./internal/backend/azure -run '^TestIssue005PolicyMultipart$' -timeout=90s -v` uploaded/downloaded a 300-MiB file with matching MD5.
- [O] `golangci-lint run --fix ./internal/backend/azure` reported zero issues; existing `go test -race ./internal/backend/azure` passed.
- [O] Aggregate package coverage and non-root permission-fixture recovery passed; the final integrated CLI race binary passed.
- [O] Independent GPT-6.1 Sol xhigh compatibility and lifecycle reviews found no remaining blocker after the SDK-policy correction.
- [O] Disposable harnesses were removed and the owned Azurite container was stopped.
- [A] Official cloud integration and Windows runtime checks were not performed.

## Publication-Blockers

- Exact current draft and `restic/restic:master` target require the user's approval before external publication.

## Next-Action

Summary: Approve Azure PR publication
Action: Show the exact draft below and obtain approval for its recorded upstream target.
Done-When: the user approves the current title, body, base, and fork head; then publish and record the PR URL.

## Pull-Request-Implementation

Branch: `fix/azure-single-upload-buffer`
Base: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`
Scope: stream eligible single-upload inputs with compatible fallback and safe per-attempt HTTP reader retirement.
Commit: `93b66eac4f3647834d500cfef38fb0bdc2ba2cd0`; initial optimization `7e77f6760f3d016779b5bc261aa3c13b151f5aaf`.
Push: `MikeeI/restic-fork:fix/azure-single-upload-buffer` pushed successfully.
Checks:
- Existing Azure race tests, emulator-backed lifecycle/native-retry/multipart checks, and final aggregate CLI race tests passed.
- Contribution diff contains `azure.go`, `upload-policy.go`, and `changelog/unreleased/azure-single-upload-buffer` only.

## Publication-Draft

Target: `restic/restic`, base `master`, head `MikeeI:fix/azure-single-upload-buffer`.
Title: azure: avoid redundant buffering of seekable single uploads
Body:

```markdown
### What does this PR change? What problem does it solve?

Single Azure uploads currently allocate and fill an additional payload-sized buffer.
This PR uploads compatible seekable inputs directly while retaining buffering for other RewindReaders and preserving declared-length handling, MD5, and caller ownership.
A per-retry policy retires previous HTTP readers before rewinds or caller cleanup, after the SDK has downloaded the response.
An early-503 reader/rewind race was independently reproduced on unchanged upstream; the lifecycle guard is required for safe streaming.

For one 8-MiB Save against Azurite 3.35.0, total allocation fell from 8.48–8.49 MB to 85.6–85.7 kB.
This is not a peak-RSS or production-throughput claim.
Byte/file/nonseekable uploads, MD5 failures, length handling, HTTP/1 and HTTP/2 retries/cancellation, native GetBody, and 300-MiB multipart contents were checked.
Existing Azure race tests and the integrated CLI race suite passed.
Cloud Azure and Windows runtime validation remain unperformed; arbitrary blocked custom readers are not covered by the cancellation claim.

### Was the change previously discussed in an issue or on the forum?

Follow-up to https://github.com/restic/restic/issues/5531 and https://github.com/restic/restic/pull/5544.
Single uploads still use Put Blob; multipart strategy and the pack hash pass are unchanged.

### Checklist

- [ ] Added permanent tests for all code changes; existing tests and disposable checks were used instead.
- [ ] Updated the manual; not applicable because configuration and upload semantics are unchanged.
- [x] Added a changelog entry; its filename will use this PR's number once assigned.
- [x] Ready for review.

### Disclosure

Investigated thoroughly with GPT-6.1 Sol (extra high reasoning effort), using [Oh My Pi](https://github.com/can1357/oh-my-pi) as the agent framework.
I reviewed this contribution with GPT-6.1 Sol at xhigh reasoning effort.

This report is not generic or unreviewed AI-generated output.
Its claims were checked against the cited evidence, and it includes the relevant detail intended to help maintainers resolve the issue.

If reports like this are not useful to the project, please let me know and I will refrain from submitting similar ones.
My intent is to help without wasting maintainer time or energy or discouraging their work.

Thank you for your work.
```
