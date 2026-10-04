# ISSUE-017 — internal/restorer: recursive deletion removes excluded descendants

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

[S] Cleanup evaluates the selection filter only for an unexpected parent and then calls `RemoveAll` on it.
[S] The descendant walk collects reporting paths without applying exclusion rules.

## Reach-and-Impact

Review correspondence: E2; severity: Critical.
[O] `restore --delete` removed an explicitly excluded file beneath a directory absent from the snapshot.
[S] Excluded local-only data can be lost even when restore reports success.
The reproduced cause is filtered deletion, distinct from deletion after unreadable trees in `ISSUE-016`.

## Evidence

- [S] `doc/050_restore.rst:153-156`: include and exclude options restrict which target entries may be deleted.
- [S] `internal/restorer/restorer.go:521-524`: only the parent path is passed to `SelectFilter`.
- [S] `internal/restorer/restorer.go:527-540`: descendants are collected without filtering and removed recursively.
- [S] `cmd/restic/cmd_restore.go:195-215`: an exact descendant exclusion does not exclude its parent.
- [O] Output: `Summary: Restored 4 files/dirs (26 B) in 0:00, deleted 3 files/dirs`.
- [O] Result: `excluded_descendant exit=0 protected=DELETED`.
- [O] On 2026-10-05, the canonical upstream head matched the recorded revision; cited source files matched it.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured; temporary artifacts were removed.

1. Back up a source containing ordinary files but no `orphan` directory.
2. In the restore target, create `orphan/protected.txt` containing `must-survive\n`.
3. Create sibling `orphan/delete.txt` containing `may-delete\n`.
4. Run `restic restore latest --target <target> --delete --exclude /orphan/protected.txt --quiet`.
5. Observe exit status `0` and deletion of both children and their parent.

## Prior-Art

Coverage: the local index and open records were searched for this cause on 2026-10-05; no duplicate was found.
`ISSUE-012` concerns xattr matcher preparation, not filesystem deletion selection.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

Apply selection rules to each descendant instead of recursively deleting an unchecked selected parent.
Delete eligible children bottom-up and retain directories containing protected descendants.

## Scope-and-Constraints

- Preserve include/exclude semantics, case-insensitive selection, dry-run behavior, and deletion reporting.
- Do not replace filtering with a blanket refusal to delete every nonempty directory.
- Preserve existing path checks and avoid following symlinks during traversal.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The defect was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat the mixed-child scenario and require only `delete.txt` to disappear.
The excluded child and its parent must remain, and dry-run reporting must agree with actual deletion.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check filtered-deletion prior art
Action: Search canonical upstream work for recursive restore deletion bypassing descendant exclusions.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
