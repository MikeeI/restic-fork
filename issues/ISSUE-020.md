# ISSUE-020 — cmd/restic: tag persistence failures return a successful exit status

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

[S] The `runTag` snapshot callback logs errors from `changeTags` and returns `nil`.
[S] The command therefore returns success despite failed snapshot persistence.

## Reach-and-Impact

Review correspondence: E5; severity: High.
[O] A permission-denied snapshot write left the requested tag absent while `tag` exited with status `0`.
[S] Automation using the documented exit status cannot distinguish this failure from success.
The observed result establishes missing tags and false success, not downstream retention loss.

## Evidence

- [S] `cmd/restic/cmd_tag.go:36-37`: the CLI promises exit status `1` if any error occurs.
- [S] `cmd/restic/cmd_tag.go:103-114`: snapshot save and removal errors propagate out of `changeTags`.
- [S] `cmd/restic/cmd_tag.go:166-169,176-182`: the caller suppresses the error and returns success.
- [O] Output included `permission denied`, `no snapshots were modified`, and `tag_write_failure exit=0`.
- [O] A subsequent `snapshots --latest 1 --json` query projected `{"tags":null}`.
- [O] On 2026-10-05, the canonical upstream head matched the recorded revision; `cmd/restic/cmd_tag.go` matched it.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository copy.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured; temporary artifacts were removed.

1. Give `nobody:nogroup` ownership of a disposable repository containing an untagged snapshot.
2. Ensure its lock directory is writable and its parent path is traversable; set `snapshots/` permissions to `555`.
3. As `nobody`, run `restic --no-cache tag --add important latest` with that repository's password.
4. Capture exit status and inspect tags using `restic --no-cache snapshots --latest 1 --json`.
5. Observe a failed snapshot write, exit status `0`, and no persisted `important` tag.

The original invocation used `runuser -u nobody -- env RESTIC_PASSWORD=semantic-review-disposable RESTIC_REPOSITORY="$base/tag-repo" "$base/restic" --no-cache tag --add important latest`.

## Prior-Art

Coverage: the local index and open records were searched for tag error suppression on 2026-10-05; no duplicate was found.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

Return the `changeTags` error from the snapshot callback instead of replacing it with `nil`.
Leave final error presentation to the existing CLI boundary rather than logging and returning it twice.

## Scope-and-Constraints

- Preserve successful tag mutation, snapshot replacement ordering, and existing error identity.
- No rollback, transactional batch tagging, or new error aggregation abstraction is required.
- Do not claim an observed retention consequence or snapshot deletion from repeated tag setting.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The failed-write exit contract was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat the unprivileged write-denial scenario and require a nonzero exit status.
Keep the repository lock path writable so the failure reaches tag persistence rather than repository opening.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check tag-failure prior art
Action: Search canonical upstream work for tag mutation failures reported with exit status zero.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
