# ISSUE-014 — internal/fuse: snapshot name collisions restart suffix search from one

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] `uniqueName` retries suffixes from one for each collision while a directory build only adds entries.
[S] A large homogeneous collision group therefore repeats checks of already occupied lower suffixes.

## Reach-and-Impact

[S] Snapshot-directory construction reaches this search for template paths that collide.
[S] Custom templates such as `hosts/%h` can create large groups; standard time-based templates can also collide.
[S] A homogeneous 10,000-name model performs 50,005,000 occupancy probes, not a measured default mount workload.
[A] Real group sizes, mount cost, and reload-frequency impact are unmeasured.

## Evidence

- [S] `internal/fuse/snapshots_dirstruct.go:200-209`: suffix loop restarts from one.
- [S] `internal/fuse/snapshots_dirstruct.go:265-288`: insertion-only build and generation update.
- [S] `internal/fuse/root.go:59-66`: default snapshot path templates.

## Prior-Art

Coverage: the complete local ledger was screened for FUSE suffix scanning on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Use a cursor per canonical prefix/name pair within one `makeDirs` build.
Retain the legacy search for noncanonical cases and reset cursors for every rebuilt directory generation.

## Scope-and-Constraints

- Preserve exact names, sorting, alias collisions, `latest` links, and generation behavior.
- Do not assume raw template strings and `path.Clean` keys always coincide.
- Exclude template changes, persistent counters, and filesystem API changes.
- Review correspondence: P12; low priority reflects unmeasured real collision pressure.

## Verification

Planned, not run: existing `TestMakeDirs` plus an actual disposable FUSE directory listing after an authorized fix.
Confirm identical names, links, and generation behavior with approximately linear probes in the collision scenario.

## Publication-Blockers

- No Go-level collision measurement or actual changed FUSE-surface verification.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure FUSE suffix collisions
Action: Measure one directory build with a supported template producing a large canonical collision group.
Done-When: template, snapshot count, collision sizes, probes, command, environment, and runtime are recorded.
