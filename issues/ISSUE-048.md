# ISSUE-048 — internal/dump: top-level special files are emitted as regular archive entries

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

[S] `sendTrees` forwards top-level nodes without the file, directory, and symlink filter applied to nested nodes.

## Reach-and-Impact

Review correspondence: E30; severity: Low.
[S] FIFOs, sockets, and devices directly in the dumped directory appear in tar as empty regular files, while nested ones are omitted.

## Evidence

- [S] `internal/dump/common.go:58-65` — top-level nodes are sent unfiltered.
- [S] `internal/dump/common.go:93-95` — nested nodes are filtered.
- [S] `internal/dump/tar.go:82-94` — no type flag is set for other node types.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Apply the nested node type filter in `sendTrees` before `sendNodes`.

## Scope-and-Constraints

- Preserve: dumping a single named special file if that is intended.
- Exclude: adding special-file support to archives.
- Cost: minimal.

## Verification

- Dump a tree containing a top-level FIFO as tar → no entry for the FIFO.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Reproduce dump special-file entries
Action: Back up a directory containing a FIFO and run `restic dump --archive tar` on it with current upstream.
Done-When: The tar listing and versions are recorded.
