# Issue and Pull Request Tracking

Read this index at the start of every agent session before repository work.
`FORMAT.md` owns research, lifecycle, drafting, implementation, and publication rules.
Each linked `issues/ISSUE-NNN.md` is the complete authoritative record for one root cause.
This file owns `Next finding ID` and projects current issue-file state.
`Next-Action` is the `Next-Action/Summary` projection from the issue record.
When a row disagrees with its issue file, correct the row from the issue file in the same task.

Next finding ID: ISSUE-054

## Open-Findings

| ID | Finding | State | Authorized-Work | Publication-Target | Contribution-Priority | Next-Action | External-Reference |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-001](issues/ISSUE-001.md) | cmd/restic: stats loads every tree of every snapshot sequentially | Investigating | Not-Selected | Not-Selected | Low | Measure stats tree-load cost | Not published. |
| [ISSUE-002](issues/ISSUE-002.md) | cmd/restic: recover loads every repository tree sequentially | Investigating | Not-Selected | Not-Selected | Low | Measure recover tree-load runtime | Not published. |
| [ISSUE-003](issues/ISSUE-003.md) | internal/walker: TreeRewriter reloads tree bytes after its serialization check | Investigating | Not-Selected | Not-Selected | Medium | Measure rewrite tree loads | Not published. |
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
| [ISSUE-022](issues/ISSUE-022.md) | internal/repository: prune drops the last copy of a blob duplicated in an ignored missing pack | Investigating | Not-Selected | Not-Selected | High | Reproduce prune blob loss | Not published. |
| [ISSUE-023](issues/ISSUE-023.md) | cmd/restic: repair snapshots deletes snapshots with unreadable roots without --forget | Investigating | Not-Selected | Not-Selected | Medium | Check repair-root deletion prior art | Not published. |
| [ISSUE-024](issues/ISSUE-024.md) | cmd/restic: recover panics on an unreadable snapshot file | Investigating | Not-Selected | Not-Selected | High | Reproduce recover snapshot panic | Not published. |
| [ISSUE-025](issues/ISSUE-025.md) | internal/fs: ownership-by-name restores unresolvable owners as root | Investigating | Not-Selected | Not-Selected | High | Reproduce ownership-by-name root fallback | Not published. |
| [ISSUE-026](issues/ISSUE-026.md) | cmd/restic: diff reports success after subtree load failures | Investigating | Not-Selected | Not-Selected | High | Check diff-error prior art | Not published. |
| [ISSUE-027](issues/ISSUE-027.md) | internal/repository: check reports every pack missing after a pack listing error | Investigating | Not-Selected | Not-Selected | Medium | Check pack-listing prior art | Not published. |
| [ISSUE-028](issues/ISSUE-028.md) | internal/backend/cache: cache write failures fail saves that already reached the repository | Investigating | Not-Selected | Not-Selected | Medium | Reproduce cache-full save failure | Not published. |
| [ISSUE-029](issues/ISSUE-029.md) | internal/restorer: partially written files receive snapshot metadata | Investigating | Not-Selected | Not-Selected | Medium | Reproduce partial-file mtime reuse | Not published. |
| [ISSUE-030](issues/ISSUE-030.md) | internal/restorer: later blobs write into the old target after file creation fails | Investigating | Not-Selected | Not-Selected | Medium | Reproduce failed-create blob writes | Not published. |
| [ISSUE-031](issues/ISSUE-031.md) | internal/data: large hour-based keep-within durations overflow and keep nothing | Investigating | Not-Selected | Not-Selected | Medium | Reproduce keep-within overflow | Not published. |
| [ISSUE-032](issues/ISSUE-032.md) | internal/data: FindAll resolves latest from a list mutated by earlier callbacks | Investigating | Not-Selected | Not-Selected | Medium | Reproduce tag latest ordering failure | Not published. |
| [ISSUE-033](issues/ISSUE-033.md) | internal/fs: non-root chown permission errors hide later metadata restore failures | Investigating | Not-Selected | Not-Selected | Medium | Reproduce masked metadata errors | Not published. |
| [ISSUE-034](issues/ISSUE-034.md) | cmd/restic: find --tree stops early on a cumulative hit counter | Investigating | Not-Selected | Not-Selected | Medium | Reproduce find tree early stop | Not published. |
| [ISSUE-035](issues/ISSUE-035.md) | internal/backend/sftp: failed saves leave temporary files after chmod or length errors | Investigating | Not-Selected | Not-Selected | Medium | Check sftp temp-file prior art | Not published. |
| [ISSUE-036](issues/ISSUE-036.md) | cmd/restic: snapshots --json returns success after output errors | Investigating | Not-Selected | Not-Selected | Medium | Reproduce snapshots JSON exit | Not published. |
| [ISSUE-037](issues/ISSUE-037.md) | cmd/restic: mount leaves a dead FUSE mount when the initial snapshot load fails | Investigating | Not-Selected | Not-Selected | Medium | Reproduce stale mount | Not published. |
| [ISSUE-038](issues/ISSUE-038.md) | internal/fs: Windows extended attribute restore ignores the xattr filter | Investigating | Not-Selected | Not-Selected | Low | Reproduce Windows EA filtering | Not published. |
| [ISSUE-039](issues/ISSUE-039.md) | internal/data: ParseDuration panics on out-of-range numbers | Investigating | Not-Selected | Not-Selected | Low | Check duration panic prior art | Not published. |
| [ISSUE-040](issues/ISSUE-040.md) | cmd/restic: dump --target truncates the target before resolving the dump path | Investigating | Not-Selected | Not-Selected | Low | Reproduce dump target truncation | Not published. |
| [ISSUE-041](issues/ISSUE-041.md) | cmd/restic: generate completion close overwrites generator errors | Investigating | Not-Selected | Not-Selected | Low | Reproduce generate error masking | Not published. |
| [ISSUE-042](issues/ISSUE-042.md) | cmd/restic: migrate returns success for unknown migration names | Investigating | Not-Selected | Not-Selected | Low | Reproduce unknown migrate exit | Not published. |
| [ISSUE-043](issues/ISSUE-043.md) | cmd/restic: cache --cleanup returns success after removal failures | Investigating | Not-Selected | Not-Selected | Low | Reproduce cache cleanup exit | Not published. |
| [ISSUE-044](issues/ISSUE-044.md) | internal/fs: swallowed getxattr errors record empty extended attribute values | Investigating | Not-Selected | Not-Selected | Low | Check empty-xattr prior art | Not published. |
| [ISSUE-045](issues/ISSUE-045.md) | internal/backend/b2: failed saves close the writer and may commit truncated objects | Investigating | Not-Selected | Not-Selected | Low | Verify blazer close semantics | Not published. |
| [ISSUE-046](issues/ISSUE-046.md) | internal/fuse: symlinks and special files report zero link count | Investigating | Not-Selected | Not-Selected | Low | Locate regular-file nlink fix | Not published. |
| [ISSUE-047](issues/ISSUE-047.md) | internal/fuse: single-tag path templates skip tag filename sanitization | Investigating | Not-Selected | Not-Selected | Low | Check FUSE tag sanitization prior art | Not published. |
| [ISSUE-048](issues/ISSUE-048.md) | internal/dump: top-level special files are emitted as regular archive entries | Investigating | Not-Selected | Not-Selected | Low | Reproduce dump special-file entries | Not published. |
| [ISSUE-049](issues/ISSUE-049.md) | cmd/restic: restore increments its error counter from concurrent workers | Investigating | Not-Selected | Not-Selected | Low | Reproduce restore counter race | Not published. |
| [ISSUE-050](issues/ISSUE-050.md) | cmd/restic: stats files-by-contents counts a directory as a unique file | Investigating | Not-Selected | Not-Selected | Low | Reproduce stats file overcount | Not published. |
| [ISSUE-051](issues/ISSUE-051.md) | internal/ui/backup: JSON seconds_remaining underflows when processed bytes exceed the scan total | Investigating | Not-Selected | Not-Selected | Low | Reproduce JSON ETA underflow | Not published. |
| [ISSUE-052](issues/ISSUE-052.md) | internal/repository: unlock aborts when a lock file was already removed | Investigating | Not-Selected | Not-Selected | Low | Reproduce unlock not-exist abort | Not published. |
| [ISSUE-053](issues/ISSUE-053.md) | internal/fuse: directory attributes read items without locking before open | Investigating | Not-Selected | Not-Selected | Low | Reproduce FUSE directory race | Not published. |

## Archived-Findings

| ID | Finding | Authorized-Work | Publication-Target | Contribution-Priority | Archive-Reason | External-Reference |
| --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-004](issues/archive/ISSUE-004.md) | internal/repository: CopyBlobs scans the full index for disabled debug output | Pull-Request-Implementation | New-pull-request | High | Merged | https://github.com/restic/restic/pull/22103 |
