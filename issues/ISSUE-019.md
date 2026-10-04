# ISSUE-019 — internal/restorer: skipped files become sources for missing hardlink members

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

[S] The first selected hardlink member is registered before `withOverwriteCheck` decides whether to restore it.
[S] Later missing members are linked to that local file even when overwrite policy skipped its differing content.

## Reach-and-Impact

Review correspondence: E4; severity: High.
[O] With `--overwrite never --verify`, a missing member received local content instead of snapshot content.
[O] The command exited successfully, and the two target paths referenced the same inode.
[S] Verification omits the new alias because the first pass never registered it as a restored file.
[S] `--overwrite if-newer` reaches the same skipped-source branch when the first local member is newer.
Only the `never` case was executed; the repository remained intact.

## Evidence

- [S] `internal/restorer/restorer.go:396-405`: hardlink registration precedes the overwrite decision.
- [S] `internal/restorer/restorer.go:564-574`: rejected overwrites return without invoking the restoration callback.
- [S] `internal/restorer/restorer.go:454-458`: later members link to the registered source.
- [S] `internal/restorer/restorer.go:645-652`: verification omits paths absent from the restored-file map.
- [O] Output: `skipped_hardlink exit=0 expected=snapshot-data`, followed by `skipped_hardlink actual=local-newer`.
- [O] `stat -c` reported inode `39868689` and link count `2` for both target `a` and `b`.
- [O] On 2026-10-05, the canonical upstream head matched the recorded revision; cited source files matched it.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured; temporary artifacts were removed.

1. Create source `a` containing `snapshot-data\n` and source `b` as a hardlink to `a`; back them up.
2. Create target `a` containing `local-newer\n` and leave target `b` absent.
3. Run `restic restore latest --target <target> --overwrite never --verify --quiet`.
4. Read target `b` and inspect both paths with `stat -c '%n inode=%i links=%h'`.
5. Observe success, wrong content in `b`, and both target paths sharing one inode.

## Prior-Art

Coverage: the local index and open records were searched for this cause on 2026-10-05; no duplicate was found.
`ISSUE-018` owns invalidated block reuse after inode replacement, not selection of a skipped hardlink source.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

Move `idx.Add` for multiply linked files into the callback actually executed by `withOverwriteCheck`.
Keep skipped files out of the source index so the first eligible member uses the normal restore and verification path.
The reviewed correction does not require hashing every alias separately.

## Scope-and-Constraints

- Preserve existing files excluded from overwriting and retain each overwrite mode's documented content assumptions.
- Preserve hardlink grouping, progress accounting, error reporting, and representative-file verification.
- Do not split the missing verification symptom into a separate root-cause finding.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The `never` scenario was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat it and require target `a` unchanged while target `b` contains snapshot bytes.
Use the existing representative-file verification rather than introducing unconditional alias rechecks.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check skipped-hardlink prior art
Action: Search canonical upstream work for missing hardlinks restored from overwrite-skipped local files.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
