# ISSUE-010 — internal/data: snapshot prefix batches repeatedly scan the memorized ID list

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Medium
Root-Cause-Confidence: High
Finding-Category: Performance
Created: 2026-10-02
Updated: 2026-10-02
Source: `upstream/master@5127c4abf921857fde4ae51f566c86028c8c2911`

## Root-Cause

[S] `SnapshotFilter.FindAll` memorizes the snapshot listing once but calls linear prefix resolution per argument.
[S] Resolution traverses the memorized IDs and converts them to strings again for each prefix.

## Reach-and-Impact

[S] A large batch of uniquely resolving abbreviated IDs can perform arguments-times-snapshots local work.
[S] Full IDs take a direct parsing path; the root cause is not repeated network listing.
[A] The crossover between a search index and linear lookup is unmeasured, especially for small batches.

## Evidence

- [S] `internal/data/snapshot_find.go:97-106,130-181`: full-ID path, memorized listing, argument loop, and callbacks.
- [S] `internal/restic/backend_find.go:27-54`: linear prefix comparison and typed missing/ambiguous errors.
- [S] `internal/restic/lister.go:22-31`: memorized listing still iterates its entries and checks cancellation.

## Prior-Art

Coverage: the complete local ledger was screened for repeated prefix resolution on 2026-10-02.
No local duplicate owns this cause.
Gaps: upstream issue, pull request, and history searches are not recorded.
Contribution fit remains unresolved.

## Proposed-Change

Use a command-local ID search structure for large prefix batches, amortizing ID string conversion and lookup.
Keep snapshot metadata loading and callback invocation in their existing argument order.

## Scope-and-Constraints

- Preserve missing and ambiguous prefix error types, null-ID handling, cancellation, and callback order.
- Do not cache snapshot metadata or reorder externally visible callbacks through ID sorting.
- Avoid imposing index-building cost on every one-prefix command without crossover evidence.
- Exclude network-listing changes and persistent caches.
- Review correspondence: P8; impact is specific to large abbreviated-ID batches.

## Verification

Planned, not run: actual CLI batch with uniquely resolving prefixes after an authorized fix.
Confirm identical snapshots, ordering, and typed errors without a complete ID scan for each prefix.

## Publication-Blockers

- No representative batch benchmark or measured crossover.
- Upstream prior art is unresolved.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Measure snapshot prefix batches
Action: Measure one large, uniquely resolving abbreviated-ID batch against a memorized snapshot list.
Done-When: argument and snapshot counts, conversions, scans, command, environment, and runtime are recorded.
