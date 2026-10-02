# ISSUE-009 — cmd/restic: find prepares the same path patterns for every visited node

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Finder calls single-pattern `Match` and `ChildMatch` inside node-processing loops.
[S] Each call prepares the unchanged pattern again.

## Reach-and-Impact

[S] File-pattern searches across snapshot trees repeat preparation per pattern and visited node.
[A] The share of overall find CPU and allocation is unmeasured; priority is low.

## Evidence

- [S] `cmd/restic/cmd_find.go:319-340`: full-match loop followed by the directory child-match loop.
- [S] `internal/filter/filter.go:80-118`: per-call pattern and path preparation.
- [S] `internal/filter/filter.go:255-265`: `ParsePatterns` drops empty patterns and is not a single-match substitute.

## Prior-Art

Coverage: the complete local ledger was screened for find pattern preparation on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Prepare single `Pattern` values once for the finder and retain single-pattern matching semantics.
Leave path preparation and the existing full-match/child-match control-flow order unchanged.

## Scope-and-Constraints

- Preserve empty patterns, literal negation syntax, malformed-pattern errors, and their evaluation order.
- Do not replace single matching with list matching or introduce a prepared-path abstraction.
- Keep preparation scoped to one finder invocation rather than a global cache.
- Review correspondence: P7, narrowed from pattern-and-path preparation.

## Verification

Planned, not run: existing filter tests and an actual find smoke scenario after an authorized fix.
Confirm identical matches and errors with preparation once per input pattern rather than once per node.

## Publication-Blockers

- No representative find profile or correction verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure find pattern preparation
Action: Profile repeated single-pattern preparation in one multi-snapshot file search.
Done-When: patterns, visited-node count, preparation count, allocations, command, and environment are recorded.
