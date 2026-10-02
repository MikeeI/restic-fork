# ISSUE-011 — cmd/restic: text-mode forget constructs JSON-only report objects

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

[S] Forget constructs JSON snapshot and keep reports independently of `gopts.JSON`.
[S] Only JSON output consumes those report objects.

## Reach-and-Impact

[S] Text-mode retention-policy runs build wrappers and short-ID strings that they do not print.
[A] Overall memory pressure is unmeasured; existing snapshot and policy state may dominate it.

## Evidence

- [S] `cmd/restic/cmd_forget.go:290-303`: unconditional report construction.
- [S] `cmd/restic/cmd_forget.go:338`: report consumption under `gopts.JSON`.
- [S] `cmd/restic/cmd_forget.go:372-403`: wrapper and short-ID construction.

## Prior-Art

Coverage: the complete local ledger was screened for unused forget reporting on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Construct JSON reports only in JSON mode.
Keep group-key decoding, policy evaluation, removal IDs, refusal handling, and text output outside that guard.

## Scope-and-Constraints

- Preserve retention decisions and both output modes.
- Do not guard policy state merely because report state is unnecessary.
- Exclude retention-policy changes and snapshot metadata caching.
- Review correspondence: P9; priority is low because no job-level memory impact was measured.

## Verification

Planned, not run: text and JSON `forget --dry-run` against a disposable fixture after an authorized fix.
Confirm identical retention decisions and no JSON report allocations in text mode.

## Publication-Blockers

- No representative report-allocation measurement or correction verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure forget report allocation
Action: Profile JSON-only report allocation in one text-mode retention-policy dry run.
Done-When: snapshot and group counts, report allocations, command, environment, and runtime are recorded.
