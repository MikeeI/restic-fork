# ISSUE-006 — internal/filter: multiple double wildcards expand before a failing literal tail check

State: Submitted
Authorized-Work: Pull-Request-Implementation
Publication-Target: New-pull-request
External-Reference: https://github.com/restic/restic/pull/22101
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-05
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Recursive double-wildcard expansion precedes comparison of the final simple literal.
[S] If a nonempty literal tail occurs in no path component, every expanded full-match candidate must fail.

## Reach-and-Impact

[S] Prepared include/exclude lists reach this matcher while scanning and archiving paths.
[O] `**/**/**/**/**/missing` against 20 `a` components took 2.599–3.223 ms/op upstream and 0.635–1.294 µs/op after correction.
[O] That prepared-match case fell from approximately 4,019,360 B and 8,374 allocations to 320 B and one allocation.
[A] Real workload frequency and end-to-end backup speedup remain unmeasured.
[S] Other expensive wildcard cases remain; this is not a general linear-time glob engine.

## Evidence

- [S] Baseline `internal/filter/filter.go:149-176`: recursive expansion and candidate allocation.
- [S] `internal/filter/filter.go:199-212`: backward matching reaches the literal tail after expansion.
- [S] `internal/filter/filter.go:80-118`: full and descendant matches have distinct semantics.
- [S] Relative matching can match an ancestor, so the check must inspect every component, not just the basename.
- [O] A deterministic disposable differential check compared 60,000 old/new Match and ChildMatch cases, including exact errors.
- [O] The check caught `Match(\"/.\", \"/b/./..\")`; empty root/expansion sentinels must not be treated as absent literals.
- [O] After excluding empty tails, all differential results and error strings matched.
- [O] Commands: `go test ./internal/filter -run '^TestIssue006Differential$' -v` and `go test ./internal/filter -run '^$' -bench '^BenchmarkIssue006Match$' -benchtime=5x -count=3`.
- [O] Environment: Ubuntu 24.04.5, Linux amd64, Go 1.27.1, AMD Ryzen 9 5950X; baseline and branch ran on the same host.

## Prior-Art

Coverage: local index, targeted upstream issue/PR/forum searches, relevant diffs, and matcher history.
Historical optimization `17c53efb0d4d9e0f38ba91231a92dea27315d3c5` reduced allocations but did not add this negative-tail guard.
Related pattern-list work: https://github.com/restic/restic/pull/2997 and https://github.com/restic/restic/issues/2694.
The inspected open trie proposal targets wildcard-free pattern lists, not recursive matching: https://github.com/restic/restic/pull/22076.
Forum discussion describes costly wildcard use without measuring this root cause: https://forum.restic.net/t/how-to-exclude-only-subdirs-not-files-but-keep-one-certain-subdir/5640.
No same-root implementation was identified in inspected work; searches were bounded rather than exhaustive.

## Proposed-Change

Before expansion, reject a full match whose nonempty simple literal tail is absent from every path component.
Keep the existing matcher for every other case, including empty sentinels and nonsimple tails.
Do not apply the full-match rejection to possible descendants.

## Scope-and-Constraints

- Preserve negation, anchoring, relative matching, short paths, and malformed-pattern error order.
- Preserve ChildMatch: relative patterns return their existing descendant answer; absolute patterns truncate before the first double wildcard.
- The presence check adds an O(path-depth) scan where the literal is present; it does not add allocations in the measured controls.
- Present-tail and nonsimple-tail cases are not claimed to gain the negative-case speedup.
- Exclude collapsing wildcards, changing validation, or replacing the glob engine.
- Test decision: none; existing tests and disposable differential/CLI checks were used without permanent new tests.
- Keep the PR-numbered changelog entry and its published URL aligned with this finding.

## Verification

- [O] `go test ./internal/filter` passed; `golangci-lint run --fix ./internal/filter` reported zero issues.
- [O] `TestIssue006Differential` passed for 60,000 deterministic cases after the empty-tail correction.
- [O] Prepared negative-case benchmark ran on upstream and corrected branches with identical inputs.
- [O] `BenchmarkIssue006PresentTail` used `missing/**/**/**/**/**/z` against 19 `a` components plus `z`.
- [O] Present-tail timings overlapped: 11.25–14.19 ms upstream and 11.86–14.91 ms corrected, with 6,877 allocations each.
- [O] Real baseline/fix backup commands with `--exclude '**/**/**/**/**/missing'` each selected one 69-byte file and 22 directories.
- [O] Fixed restore preserved the selected file byte-for-byte, omitted the excluded file, and `check --read-data` reported no errors.
- [O] Aggregate filter race tests passed; the final integrated CLI race binary passed as a non-root user.
- [O] Independent GPT-6.1 Sol xhigh review found no remaining correctness blocker.
- [O] Disposable harnesses were removed; contribution diff contains only source and changelog changes.
- [A] Windows runtime and representative production backups were not measured.

## Publication-Blockers

None.

## Next-Action

Summary: Monitor filter PR review
Action: Inspect CI and maintainer feedback on https://github.com/restic/restic/pull/22101 before proposing a scoped follow-up.
Done-When: review or CI feedback is recorded and any necessary next action is evidence-backed.

## Pull-Request-Implementation

Branch: `fix/filter-literal-tail`
Base: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`
Scope: add the safe negative-tail guard without changing other glob or descendant semantics; add the changelog.
Commit: `981f8ae06f06e4bd488ca883e6febf533cb4f70a`; implementation `974756cee184820b71534fd66edf67d3074c9833`.
Push: `MikeeI/restic-fork:fix/filter-literal-tail` pushed successfully.
Checks:
- Existing filter/race tests, 60,000-case differential validation, and actual backup/restore/check scenarios passed.
- Contribution diff contains `internal/filter/filter.go` and `changelog/unreleased/pull-22101` only.
- Published title/body match the approved text; master base, final head, and enabled maintainer edits were verified.
- GitHub CI rollup was PENDING after the changelog-only head update; no successful upstream CI result is claimed.

## Publication-Draft

Target: `restic/restic`, base `master`, head `MikeeI:fix/filter-literal-tail`.
Title: filter: skip recursive expansion when the literal tail cannot match
Body:

```markdown
### What does this PR change? What problem does it solve?

Patterns with several `**` parts can expand recursively even when their final literal occurs nowhere in the path.
This PR rejects that impossible full match before expansion.
The check covers every component, preserves empty sentinels, and leaves ChildMatch and other matching behavior unchanged.

For a prepared `**/**/**/**/**/missing` match against 20 components, an isolated benchmark fell from 2.60–3.22 ms and about 4.02 MB to 0.64–1.29 µs and 320 B per call.
This is not a general glob-complexity or end-to-end backup claim.
Existing filter/race tests, 60,000 old/new Match and ChildMatch comparisons including errors, and real backup/restore/read-data checks passed.
Literal-present cases still perform the original expansion plus a presence scan.

### Was the change previously discussed in an issue or on the forum?

No dedicated issue.
The inspected wildcard-free trie proposal addresses a different path: https://github.com/restic/restic/pull/22076.

### Checklist

- [ ] Added permanent tests for all code changes; existing tests and disposable checks were used instead.
- [ ] Updated the manual; not applicable because matching semantics are unchanged.
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

## Submitted-Text

Published: https://github.com/restic/restic/pull/22101
The approved title and body in `Publication-Draft` were submitted unchanged on 2026-10-05.
`Publication-Draft` preserves the immutable submitted snapshot, not a pending revision.
