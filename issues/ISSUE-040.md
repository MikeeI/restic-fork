# ISSUE-040 — cmd/restic: dump --target truncates the target before resolving the dump path

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

[S] `runDump` creates the target file before `printFromTree` resolves the requested path and discards the deferred `Close` error.

## Reach-and-Impact

Review correspondence: E20; severity: Low.
[S] A mistyped dump path truncates an existing target file to zero bytes.
[A] Write errors surfacing only at `Close` produce a truncated file with exit 0 on filesystems that defer errors.

## Evidence

- [S] `cmd/restic/cmd_dump.go:185` — `os.Create` runs first.
- [S] `cmd/restic/cmd_dump.go:189-191` — `Close` result is discarded.
- [S] `cmd/restic/cmd_dump.go:198` — path resolution happens afterwards.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Resolve the dump path before creating the target and return the `Close` error when no earlier error exists.

## Scope-and-Constraints

- Preserve: stdout dumping and archive formats.
- Exclude: atomic temp-file replacement.
- Cost: small restructuring of `printFromTree` usage.

## Verification

- `restic dump latest /missing -t existing.tar` → `existing.tar` keeps its size.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce dump target truncation
Action: Run `restic dump latest /missing -t existing.tar` on current upstream and compare sizes.
Done-When: File sizes, output, and versions are recorded.
