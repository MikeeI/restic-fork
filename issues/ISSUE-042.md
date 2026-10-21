# ISSUE-042 — cmd/restic: migrate returns success for unknown migration names

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `applyMigrations` prints `unknown migration` without setting `firsterr`.

## Reach-and-Impact

Review correspondence: E22; severity: Low.
[S] Mistyped migration names exit 0 although nothing was applied.

## Evidence

- [S] `cmd/restic/cmd_migrate.go:129-131` — unknown names only print.
- [S] `cmd/restic/cmd_migrate.go:134` — returns `firsterr`, which only `Apply` failures set.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Set `firsterr` to a fatal error for unknown migration names.

## Scope-and-Constraints

- Preserve: the not-applicable migration message and `--force` behavior.
- Exclude: changing the not-applicable exit status.
- Cost: minimal.

## Verification

- `restic migrate nosuchmigration; echo $?` → 1.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce unknown migrate exit
Action: Run `restic migrate nosuchmigration` on current upstream.
Done-When: Exit status, output, and versions are recorded.
