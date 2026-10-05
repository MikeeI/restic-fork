# Issue and Pull Request Tracking

Read this index at the start of every agent session before repository work.
`FORMAT.md` owns research, lifecycle, drafting, implementation, and publication rules.
Each linked `issues/ISSUE-NNN.md` is the complete authoritative record for one root cause.
This file owns `Next finding ID` and projects current issue-file state.
`Next-Action` is the `Next-Action/Summary` projection from the issue record.
When a row disagrees with its issue file, correct the row from the issue file in the same task.

Next finding ID: ISSUE-022

## Open-Findings

| ID | Finding | State | Authorized-Work | Publication-Target | Contribution-Priority | Next-Action | External-Reference |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-001](issues/ISSUE-001.md) | cmd/restic: stats loads every tree of every snapshot sequentially | Investigating | Not-Selected | Not-Selected | Low | Measure stats tree-load cost | Not published. |
| [ISSUE-002](issues/ISSUE-002.md) | cmd/restic: recover loads every repository tree sequentially | Investigating | Not-Selected | Not-Selected | Low | Measure recover tree-load runtime | Not published. |
| [ISSUE-003](issues/ISSUE-003.md) | internal/walker: TreeRewriter reloads tree bytes after its serialization check | Investigating | Not-Selected | Not-Selected | Medium | Measure rewrite tree loads | Not published. |
| [ISSUE-004](issues/ISSUE-004.md) | internal/repository: CopyBlobs scans the full index for disabled debug output | Submitted | Pull-Request-Implementation | New-pull-request | High | Monitor copy PR review | https://github.com/restic/restic/pull/22103 |
| [ISSUE-005](issues/ISSUE-005.md) | internal/backend/azure: single uploads allocate an additional full payload buffer | Submitted | Pull-Request-Implementation | New-pull-request | High | Monitor Azure PR review | https://github.com/restic/restic/pull/22102 |
| [ISSUE-006](issues/ISSUE-006.md) | internal/filter: multiple double wildcards expand before a failing literal tail check | Submitted | Pull-Request-Implementation | New-pull-request | High | Monitor filter PR review | https://github.com/restic/restic/pull/22101 |
| [ISSUE-007](issues/ISSUE-007.md) | internal/restorer: large-file downloads repeat index lookups for already planned blobs | Investigating | Not-Selected | Not-Selected | Medium | Measure planned restore lookups | Not published. |
| [ISSUE-008](issues/ISSUE-008.md) | internal/repository: prune traverses index entries separately to calculate pack headers | Investigating | Not-Selected | Not-Selected | Medium | Measure prune header pass | Not published. |
| [ISSUE-009](issues/ISSUE-009.md) | cmd/restic: find prepares the same path patterns for every visited node | Investigating | Not-Selected | Not-Selected | Low | Measure find pattern preparation | Not published. |
| [ISSUE-010](issues/ISSUE-010.md) | internal/data: snapshot prefix batches repeatedly scan the memorized ID list | Investigating | Not-Selected | Not-Selected | Medium | Measure snapshot prefix batches | Not published. |
| [ISSUE-011](issues/ISSUE-011.md) | cmd/restic: text-mode forget constructs JSON-only report objects | Investigating | Not-Selected | Not-Selected | Low | Measure forget report allocation | Not published. |
| [ISSUE-012](issues/ISSUE-012.md) | cmd/restic: restore rebuilds xattr matchers inside per-attribute callbacks | Investigating | Not-Selected | Not-Selected | Low | Measure xattr matcher construction | Not published. |
| [ISSUE-013](issues/ISSUE-013.md) | internal/backend: failed RoundTrip retains its watchdog until later cancellation | Investigating | Not-Selected | Not-Selected | Medium | Reproduce watchdog failure retention | Not published. |
| [ISSUE-014](issues/ISSUE-014.md) | internal/fuse: snapshot name collisions restart suffix search from one | Investigating | Not-Selected | Not-Selected | Low | Measure FUSE suffix collisions | Not published. |
| [ISSUE-015](issues/ISSUE-015.md) | internal/repository/index: incremental reload decodes known index files before skipping them | Investigating | Not-Selected | Not-Selected | Medium | Count incremental index reloads | Not published. |
| [ISSUE-016](issues/ISSUE-016.md) | internal/restorer: failed subtree reads allow deletion of existing target files | Investigating | Not-Selected | Not-Selected | High | Check deletion-failure prior art | Not published. |
| [ISSUE-017](issues/ISSUE-017.md) | internal/restorer: recursive deletion removes excluded descendants | Investigating | Not-Selected | Not-Selected | High | Check filtered-deletion prior art | Not published. |
| [ISSUE-018](issues/ISSUE-018.md) | internal/restorer: replacing hardlinked targets invalidates matching blob reuse | Investigating | Not-Selected | Not-Selected | High | Check replacement-corruption prior art | Not published. |
| [ISSUE-019](issues/ISSUE-019.md) | internal/restorer: skipped files become sources for missing hardlink members | Investigating | Not-Selected | Not-Selected | High | Check skipped-hardlink prior art | Not published. |
| [ISSUE-020](issues/ISSUE-020.md) | cmd/restic: tag persistence failures return a successful exit status | Investigating | Not-Selected | Not-Selected | High | Check tag-failure prior art | Not published. |
| [ISSUE-021](issues/ISSUE-021.md) | internal/restorer: opening existing FIFOs blocks regular-file restoration | Investigating | Not-Selected | Not-Selected | Medium | Check FIFO-restore prior art | Not published. |

## Archived-Findings

| ID | Finding | Authorized-Work | Publication-Target | Contribution-Priority | Archive-Reason | External-Reference |
| --- | --- | --- | --- | --- | --- | --- |
