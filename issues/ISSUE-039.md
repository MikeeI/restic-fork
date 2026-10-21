# ISSUE-039 — internal/data: ParseDuration panics on out-of-range numbers

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `nextNumber` panics when `strconv.Atoi` rejects the collected digits.

## Reach-and-Impact

Review correspondence: E19; severity: Low.
[S] `forget --keep-within 99999999999999999999d` crashes with a stack trace instead of a usage error.
[S] No repository state changes before the crash.

## Evidence

- [S] `internal/data/duration.go:66-69` — `panic(err)` on Atoi failure.
- [S] `internal/data/duration.go:63` — other invalid input returns errors.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Return the Atoi error instead of panicking.

## Scope-and-Constraints

- Preserve: accepted duration syntax.
- Exclude: range validation for the later time arithmetic, owned by ISSUE-031.
- Cost: minimal.

## Verification

- `data.ParseDuration("99999999999999999999d")` → error without panic.

## Publication-Blockers

- Upstream prior art is missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check duration panic prior art
Action: Search upstream issues and pull requests for ParseDuration panics.
Done-When: Search coverage and candidate classifications are recorded.
