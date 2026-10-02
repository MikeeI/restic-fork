# ISSUE-006 — internal/filter: multiple double wildcards expand before a failing literal tail check

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

[S] Multiple double-wildcard parts are recursively expanded before the trailing simple literal is compared.
[S] If that literal occurs in no input component, every expanded full-match candidate must fail.

## Reach-and-Impact

[S] Prepared include/exclude matching reaches this expansion while scanning and archiving paths.
[S] Cost grows combinatorially with applicable path depth and the number of double-wildcard parts.
[S] The review's source-derived model used 20 components and five double wildcards: 40,480 leaf comparisons.
This model is not a Go benchmark, measured heap allocation, or a general bound for all glob patterns.
[A] Real workload frequency and absolute backup slowdown are unmeasured.

## Evidence

- [S] `internal/filter/filter.go:149-176`: recursive expansion and candidate allocation.
- [S] `internal/filter/filter.go:199-212`: backward matching reaches the literal tail only after expansion.
- [S] `internal/filter/filter.go:80-118`: full match and descendant-match entry points have distinct semantics.

## Prior-Art

Coverage: the complete local ledger was screened for filter expansion on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved; no global linear-time matcher claim is made.

## Proposed-Change

Reject a full match before expansion when its nonempty simple tail literal occurs in no path component.
Retain the existing matcher for every other case.
Do not use this full-match condition to reject possible descendants in `ChildMatch`.

## Scope-and-Constraints

- Preserve negation, anchoring, component semantics, short paths, and malformed-pattern error timing.
- Check absence across all components, not only the basename.
- Keep descendant matching and pattern-validation order unchanged.
- Exclude collapsing double wildcards or replacing the glob engine.
- Review correspondence: P3; priority applies to this costly pattern configuration.

## Verification

Planned, not run: reproduce the source-derived scenario at the Go matcher boundary and run existing filter tests.
A future correction must preserve match results and errors while eliminating expansion in the safe negative case.

## Publication-Blockers

- No Go-level runtime or allocation measurement for the scenario.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure double wildcard expansion
Action: Measure one full-match scenario with several double wildcards and an absent simple tail literal.
Done-When: exact pattern, path components, expansion counts, runtime, allocations, and environment are recorded.
