# ISSUE-004 — internal/repository: CopyBlobs scans the full index for disabled debug output

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

[S] `CopyBlobs` evaluates `keepBlobs.Len()` before `debug.Log` checks whether logging is enabled.
[S] The associated blob set used by `copy` counts through the complete source index.

## Reach-and-Impact

[S] `copyTree` invokes `CopyBlobs` for each copied snapshot, including small or empty selected sets.
[O] An isolated 100,000-entry index benchmark removed one `Len` traversal per `CopyBlobs` call.
[O] That entry overhead fell from 18.169–21.372 ms/op to 3.000–4.846 µs/op.
[S] Required `copyStats` traversal and batching remain; this does not eliminate every per-snapshot index scan.
[A] End-to-end copy speedup and representative multi-snapshot workload frequency remain unmeasured.

## Evidence

- [S] Baseline `internal/repository/repack.go:17-42`: private interface requirement and eager debug count.
- [S] `internal/debug/debug.go:164-166`: argument evaluation precedes the enabled check.
- [S] `internal/repository/index/associated_data.go:145-185`: `Len` and `Keys` traverse `All`.
- [S] `cmd/restic/cmd_copy.go:279,324-348`: per-snapshot associated set and required statistics traversal.
- [S] LSP references covered copy, prune, and existing repack tests; concrete prune integrity checks remain.
- [O] Disposable benchmark used the real associated set with 10,000 or 100,000 indexed blobs and zero selected blobs.
- [O] At 100,000 entries, allocations fell from approximately 9.60 MB and 100,034–100,035 allocations to 1,432–1,561 B and 27 allocations per call.
- [O] Benchmark command: `go test ./internal/repository -run '^$' -bench '^BenchmarkIssue004Copy$' -benchtime=10x -count=3`.
- [O] Environment: Ubuntu 24.04.5, Linux amd64, Go 1.27.1, AMD Ryzen 9 5950X; baseline and branch ran on the same host.

## Prior-Art

Coverage: local index, targeted upstream issue/PR searches, relevant PR diffs, and introduction history were checked.
The `copy index slow` and `copy debug` searches were bounded; exact `CopyBlobs` and `keepBlobs.Len` searches supplemented them.
No same-root implementation was identified in the inspected work; this is not an exhaustive novelty claim.
Related batching work is merged: https://github.com/restic/restic/pull/5472.
The inspected rechunk-copy proposal does not change `repack.go`: https://github.com/restic/restic/pull/5529.
History traces the interface-count introduction to `c4fc5c97f`; the current canonical baseline still evaluates it.

## Proposed-Change

Log the pack count without computing the selected-blob count.
Remove only the unused `Len()` requirement from private `repackBlobSet`.
Retain concrete set counts, required statistics, batching, and prune integrity checks.

## Scope-and-Constraints

- Preserve selected blobs, pack processing, cancellation, progress, and copied snapshot contents.
- Exclude generic lazy logging, associated-set redesign, and claims of eliminating all index scans.
- Test decision: none; existing tests and a disposable benchmark cover the change without permanent new tests.
- Rename the descriptive changelog file to the PR number after publication assigns one.

## Verification

- [O] `go test ./internal/repository -run '^(TestRepack.*|TestPrune.*)$'` passed.
- [O] `go test ./cmd/restic -run '^TestCopy.*$'` passed.
- [O] `golangci-lint run --fix ./internal/repository/... ./cmd/restic/...` reported zero issues.
- [O] Aggregate `go test -race ./internal/repository ./internal/filter ./internal/backend/azure` passed.
- [O] Aggregate `go test ./...` covered all packages but failed root-dependent permission fixtures in CLI and archiver.
- [O] Both complete affected package test binaries passed as UID/GID 65534; the final aggregate CLI race binary also passed.
- [O] Independent GPT-6.1 Sol xhigh review found no remaining correctness blocker.
- [O] Disposable benchmark files were removed; contribution diff contains only source and changelog changes.

## Publication-Blockers

- Exact current draft and `restic/restic:master` target require the user's approval before external publication.

## Next-Action

Summary: Approve copy PR publication
Action: Show the exact draft below and obtain approval for its recorded upstream target.
Done-When: the user approves the current title, body, base, and fork head; then publish and record the PR URL.

## Pull-Request-Implementation

Branch: `fix/copy-debug-index-scan`
Base: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`
Scope: remove the debug-only count and private interface requirement; add the user-facing changelog.
Commit: `e779259705fc1ab1afa436c329e61621d56e9fd4`
Push: `MikeeI/restic-fork:fix/copy-debug-index-scan` pushed successfully.
Checks:
- Focused copy, repack, and prune tests passed; race and aggregate permission-sensitive checks passed as described above.
- Contribution diff contains `internal/repository/repack.go` and `changelog/unreleased/copy-debug-index-scan` only.

## Publication-Draft

Target: `restic/restic`, base `master`, head `MikeeI:fix/copy-debug-index-scan`.
Title: copy: avoid a full index scan for disabled debug output
Body:

```markdown
### What does this PR change? What problem does it solve?

`CopyBlobs` evaluates `keepBlobs.Len()` before debug logging checks whether it is enabled.
For copy's associated blob set, this traverses the complete source index for each snapshot, even with no selected blobs.
This PR removes that debug-only count and the unused private interface requirement.
Required copy statistics, batching, and prune integrity checks remain unchanged.

An isolated 100,000-entry benchmark reduced this overhead from 18.2–21.4 ms to 3.0–4.8 µs per call.
This is not an end-to-end copy speedup claim.
Existing copy/repack/prune tests and repository race checks passed.

### Was the change previously discussed in an issue or on the forum?

No dedicated issue; related copy batching work is https://github.com/restic/restic/pull/5472.

### Checklist

- [ ] Added permanent tests for all code changes; existing tests and disposable checks were used instead.
- [ ] Updated the manual; not applicable because user-facing behavior is unchanged.
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
