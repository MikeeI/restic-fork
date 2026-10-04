# ISSUE-021 — internal/restorer: opening existing FIFOs blocks regular-file restoration

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Reliability
Created: 2026-10-05
Updated: 2026-10-05
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] Content verification opens the target with blocking `O_RDONLY` before checking whether it is a regular file.
[S] An existing FIFO without a writer blocks before the subsequent `f.Stat` type check can run.
[S] The creation path also opens the existing target for writing before replacing nonregular files.

## Reach-and-Impact

Review correspondence: E6; severity: Medium.
[O] A Linux restore over a FIFO made no progress and was killed after five seconds; the target remained a FIFO.
[S] FIFO opening waits for its opposite endpoint, so this is not merely slow repository I/O.
The process was forcibly stopped; the reproduction does not establish graceful cancellation behavior.
No equivalent Windows runtime behavior was established.

## Evidence

- [S] `internal/restorer/restorer.go:716-730`: blocking read-open precedes regular-file validation.
- [S] `internal/restorer/fileswriter.go:82-112`: blocking write-open precedes the replacement decision.
- [S] `internal/fs/const_unix.go:15-16`: Unix flags are passed through unchanged.
- [S] `internal/fs/const.go:16`: `fs.O_NONBLOCK` already owns the nonblocking flag.
- [S] Linux documents blocking FIFO opens and `ENXIO` for nonblocking write-open without a reader: https://man7.org/linux/man-pages/man2/open.2.html.
- [O] Output after forced termination: `fifo_restore exit=137 type=fifo`.
- [O] On 2026-10-05, the canonical upstream head matched the recorded revision; cited source files matched it.

## Bug-Reproduction

Observed on 2026-10-04 in Ubuntu 24.04.5 LTS, Linux x64, using a disposable local repository.
The binary was built with `go build -o /tmp/restic-semantic-review.1GUxok/restic ./cmd/restic`.
The exact executable version and Go compiler version were not captured; temporary artifacts were removed.

1. Back up a regular file `a` containing `snapshot-data\n`.
2. Create target FIFO `a` with `mkfifo` and do not attach a reader or writer.
3. Run `timeout --signal=KILL 5s "$base/restic" --no-cache restore latest --target "$base/fifo" --include /a --quiet`.
4. Observe forced termination with status `137` and verify target type using `stat -c 'type=%F'`.

`base` was `/tmp/restic-semantic-review.1GUxok`; repository and password were supplied through environment variables.
The API contract, not the five-second observation alone, establishes the missing-endpoint wait.

## Prior-Art

Coverage: the local index and open records were searched for FIFO restoration hangs on 2026-10-05; no duplicate was found.
Gaps: canonical upstream issue, pull request, discussion, release, and history searches remain unrecorded.
Contribution fit remains unresolved.

## Proposed-Change

Use `fs.O_NONBLOCK` in the affected Unix opens and retain validation through `f.Stat` after successful opening.
For `ENXIO`, verify the target type and route only a confirmed FIFO through the existing `mustReplace` path.
Propagate other `ENXIO` causes instead of treating every such error as permission to remove the target.

## Scope-and-Constraints

- Preserve `O_NOFOLLOW`, exclusive replacement creation, overwrite policy, and native error propagation.
- Adding the flag alone is incomplete because write-opening a FIFO without a reader then returns `ENXIO`.
- Keep the read and write openings in one root-cause record, not separate symptom findings.
- Do not add a general filesystem wrapper or claim cross-platform verification that was not performed.
- No source implementation or external publication is authorized by this tracking update.

## Verification

The blocking-read scenario was observed; the proposed correction has not been implemented or verified.
After an authorized fix, repeat restoration over a FIFO without any peer and require bounded successful completion.
Verify a regular target file with exact snapshot bytes and ensure unrelated opening errors still propagate.

## Publication-Blockers

- Upstream prior art and contribution ownership remain unresolved.
- Preserve a fresh reproduction with executable and compiler versions before external submission.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check FIFO-restore prior art
Action: Search canonical upstream work for regular-file restore blocked by an existing FIFO.
Done-When: Search coverage and matching candidates are recorded with root-cause classifications and ownership.
