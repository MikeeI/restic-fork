# ISSUE-051 — internal/ui/backup: JSON seconds_remaining underflows when processed bytes exceed the scan total

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: UI
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] The progress updater subtracts processed from total bytes as `uint64` without checking order.

## Reach-and-Impact

Review correspondence: E33; severity: Low.
[S] Files growing during backup make JSON status report roughly 1e19 seconds remaining.
[S] The text printer guards the same case.

## Evidence

- [S] `internal/ui/backup/progress.go:76` — unguarded subtraction.
- [S] `internal/ui/backup/json.go:46` — the value is emitted unchanged.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Compute the estimate only when processed bytes are below total bytes.

## Scope-and-Constraints

- Preserve: text output and percent calculation.
- Exclude: estimator redesign.
- Cost: minimal.

## Verification

- Grow a file during a `--json` backup → `seconds_remaining` stays plausible.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce JSON ETA underflow
Action: Run a `--json` backup while appending to a source file on current upstream.
Done-When: Status lines and versions are recorded.
