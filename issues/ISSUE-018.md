# ISSUE-018 — internal/restorer: replacing hardlinked targets invalidates matching blob reuse

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: High
Root-Cause-Confidence: High
Finding-Category: Correctness
Created: 2026-10-05
Updated: 2026-10-05
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Restore planning skips blocks matching the existing target, assuming their bytes survive later writes.
[S] `createFile` replaces a target with multiple hardlinks, invalidating those matches without replanning its content.

## Reach-and-Impact

Review correspondence: E3; severity: High.
[O] A hardlinked target containing all snapshot bytes plus an extra suffix became a zero-filled file.
[O] Restore exited successfully without `--verify`; the backup repository remained available for recovery.
[S] The same ownership mismatch affects planned reuse of matching blocks when the target inode is replaced.
This is not the unrelated repeated-index-lookup optimization tracked in `ISSUE-007`.

## Evidence

- [S] `doc/050_restore.rst:130-135`: incremental restore verifies existing content and restores mismatching parts.
- [S] `internal/restorer/restorer.go:715-776`: content verification records block and size matches without link ownership.
- [S] `internal/restorer/filerestorer.go:145-155,188-190`: matching blocks are skipped even on the resize-only path.
- [S] `internal/restorer/filerestorer.go:254-259`: resize-only restoration calls `createFile`.
- [S] `internal/restorer/fileswriter.go:112-141`: a link count above one causes removal and fresh file creation.
- [O] Expected bytes were `73 6e 61 70 73 68 6f 74 2d 64 61 74 61 0a`.
- [O] Actual bytes were `00 00 00 00 00 00 00 00 00 00 00 00 00 00`; restore exited with status `0`.
- [O] On 2026-10-05, the canonical upstream head matched the recorded revision; cited source files matched it.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured; temporary artifacts were removed.

1. Back up source file `a` containing exactly `snapshot-data\n`.
2. Create target `a` containing `snapshot-data\nEXTRA\n` and hardlink target `alias` to it.
3. Run `restic restore latest --target <target> --include /a --quiet`.
4. Compare bytes using `od -An -tx1` on the source and restored file.
5. Observe `Summary: Restored 1 files/dirs (14 B) in 0:00` and fourteen zero bytes in restored `a`.

## Prior-Art

Coverage: the local index and open records were searched for this cause on 2026-10-05; no duplicate was found.
The complete `ISSUE-007` record was checked and owns a distinct performance cause.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

Before blob planning, discard existing-content matches whenever restoration requires replacing the target inode.
Schedule every snapshot block for that replacement rather than relying on bytes in the removed inode.

## Scope-and-Constraints

- Preserve the existing decision not to modify other paths sharing the target's original inode.
- Preserve incremental reuse when the target inode remains usable, along with sparse and size handling.
- Do not globally disable partial restoration or add a second content cache.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The resize-only corruption was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat the suffix-plus-hardlink scenario and compare restored bytes and length.
Require the sibling hardlink to retain its original content and check partial-match replacement through the same boundary.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check replacement-corruption prior art
Action: Search canonical upstream work for reused restore blocks becoming invalid after hardlink replacement.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
