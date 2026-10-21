# ISSUE-041 — cmd/restic: generate completion close overwrites generator errors

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

[S] The deferred `Close` in `writeCompletion` assigns its result to the named error unconditionally.

## Reach-and-Impact

Review correspondence: E21; severity: Low.
[S] Completion generation write errors are replaced by a successful `Close`, producing exit 0 with a truncated file.

## Evidence

- [S] `cmd/restic/cmd_generate.go:86` — `defer func() { err = outFile.Close() }()`.
- [S] `cmd/restic/cmd_generate.go:92` — generator error assigned before the deferred overwrite.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Assign the `Close` error only when `err` is nil.

## Scope-and-Constraints

- Preserve: stdout completion output.
- Exclude: man page generation.
- Cost: minimal.

## Verification

- `restic generate --bash-completion /dev/full; echo $?` → 1.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce generate error masking
Action: Run `restic generate --bash-completion /dev/full` on current upstream.
Done-When: Exit status, output, and versions are recorded.
