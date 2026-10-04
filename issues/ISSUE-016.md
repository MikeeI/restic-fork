# ISSUE-016 — internal/restorer: failed subtree reads allow deletion of existing target files

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

[S] A suppressed tree-load error returns no valid filename list, but the selected parent still invokes `leaveDir`.
[S] With `--delete`, cleanup treats that unknown subtree as empty and deletes existing children.

## Reach-and-Impact

Review correspondence: E1; severity: Critical.
[O] A missing subtree pack caused deletion of an existing target file before restore exited with an error.
[S] The deleted file may contain the only remaining local copy when repository data is unavailable.
The correction must prevent unsupported deletion, not promise rollback of earlier restore changes.

## Evidence

- [S] `internal/restorer/restorer.go:159-162`: failed loading returns the result of `sanitizeError` with no filename list.
- [S] `internal/ui/restore/text.go:43-45`: the CLI error callback logs the error and returns `nil`.
- [S] `internal/restorer/restorer.go:243-257,467-470`: the selected parent passes the missing list to deletion.
- [S] `internal/restorer/restorer.go:488-540`: entries absent from that list become deletion candidates.
- [S] `cmd/restic/cmd_restore.go:180-184,249-257`: final error reporting occurs after restoration work.
- [O] Output: `Fatal: There were 2 errors`, followed by `missing_subtree exit=1 existing_child=DELETED`.
- [O] On 2026-10-05, `git ls-remote upstream refs/heads/master` returned the recorded source revision.
- [O] `git diff upstream/master -- internal/restorer/restorer.go internal/ui/restore/text.go cmd/restic/cmd_restore.go` was empty.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured.
The fixture and binary were removed after the investigation.

1. Back up a source containing `dir/keep` with `saved-child\n`, then add root file `marker` and back up again.
2. Resolve the latest root tree through `snapshots --latest 1 --json` and read it using `cat blob <root-tree-ID>`.
3. Find the pack containing the unchanged `dir` subtree using `list index` and `cat index <index-ID>`.
4. Move only that disposable pack aside, leaving the latest root tree readable and the index intact.
5. Create target file `dir/keep` containing `irreplaceable-local\n`.
6. Run `restic --no-cache restore latest --target <target> --delete --quiet`.
7. Observe two subtree-load errors, exit status `1`, and deletion of target `dir/keep`; return the withheld pack.

Observed root tree: `ccd488366a18a0c67182063bfdb9d22497b3cbc338efd19460939933e52ada63`.
Observed missing subtree: `553147b009ab97e8cb1225f9dcf6edcdc4e61d5706f95d525b266b3cfdf90142`.
Observed withheld pack: `1ef111ba2b5dffd141d28354ddf3f2c538839698bb7f7f06e5fc80d5ac20d978`.

## Prior-Art

Coverage: the local index and open records were searched for this cause on 2026-10-05; no duplicate was found.
`ISSUE-007` owns repeated restore index lookups, not deletion after failed tree reads.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

When `Delete` is active, propagate tree-load and iteration errors instead of returning success without a valid list.
Abort traversal before cleanup can use an incomplete subtree listing.

## Scope-and-Constraints

- Preserve successful restore selection, metadata handling, cancellation, and error identity.
- Do not treat an unreadable subtree as evidence that its children are absent.
- Earlier restore changes remain nontransactional; this is not an atomic-restore proposal.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The defect was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat the missing-pack scenario and require failure with target `dir/keep` unchanged.
Use a disposable repository and target; do not remove packs from a live repository.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check deletion-failure prior art
Action: Search canonical upstream work for restore deletion after failed subtree reads.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
