# ISSUE-049 — cmd/restic: restore increments its error counter from concurrent workers

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

[S] The `res.Error` callback increments `totalErrors` without synchronization while restore workers invoke it concurrently.

## Reach-and-Impact

Review correspondence: E31; severity: Low.
[S] The reported error count can be wrong, and the race detector reports a data race.
[S] The exit status stays nonzero because at least one increment survives.

## Evidence

- [S] `cmd/restic/cmd_restore.go:180-184` — unsynchronized increment.
- [S] `internal/restorer/filerestorer.go:315` — `Error` is called from worker goroutines.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Use `atomic.Int64` for the counter.

## Scope-and-Constraints

- Preserve: error reporting through progress output.
- Exclude: restorer callback redesign.
- Cost: minimal.

## Verification

- Run a restore with many failing files under the race detector → no race report.

## Publication-Blockers

- Runtime race reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce restore counter race
Action: Run a race-enabled restore with multiple failing files on current upstream.
Done-When: Race detector output and versions are recorded.
