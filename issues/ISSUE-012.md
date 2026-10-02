# ISSUE-012 — cmd/restic: restore rebuilds xattr matchers inside per-attribute callbacks

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

[S] Xattr-filter callbacks construct matchers repeatedly from options fixed for the restore invocation.

## Reach-and-Impact

[S] Restores with configured xattr include or exclude patterns repeat preparation for processed attributes.
[A] Attribute counts and total restore impact are unmeasured; unfiltered restores do not establish this pressure.

## Evidence

- [S] `cmd/restic/cmd_restore.go:284-316`: option validation, callbacks, and constructors at lines 298 and 310.
- [S] `internal/filter/filter.go:255-265`: pattern preparation used by matcher construction.

## Prior-Art

Coverage: the complete local ledger was screened for xattr matcher preparation on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Construct each configured matcher before its attribute callback and reuse it during the restore.

## Scope-and-Constraints

- Preserve selected attributes, warnings, validation, and include/exclude precedence.
- Keep matcher lifetime within the command; no global cache or new abstraction is required.
- Exclude filesystem xattr semantics and unrelated restore filters.
- Review correspondence: P10; priority is low without a material workload measurement.

## Verification

Planned, not run: actual restore smoke scenario with xattr filters after an authorized fix.
Confirm identical attributes and warnings with matcher construction once rather than once per attribute.

## Publication-Blockers

- No observed constructor-count or restore verification for a correction.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure xattr matcher construction
Action: Count matcher construction in one disposable restore with xattrs and configured filters.
Done-When: attribute count, patterns, constructor count, command, environment, and runtime are recorded.
