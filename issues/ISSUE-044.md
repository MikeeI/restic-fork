# ISSUE-044 — internal/fs: swallowed getxattr errors record empty extended attribute values

State: Investigating
Authorized-Work: Not-Selected
Publication-Target: Not-Selected
External-Reference: Not published.
Contribution-Priority: Low
Root-Cause-Confidence: Medium
Finding-Category: Correctness
Created: 2026-10-21
Updated: 2026-10-21
Source: `upstream/master@dd09b6bd01af220b1d5d95c0d4d7d472e05e05c4`

## Root-Cause

[S] `getxattr` maps ENOATTR, ENOTSUP, and EOPNOTSUPP to nil and returns a nil value, which `nodeFillExtendedAttributes` records as an attribute.

## Reach-and-Impact

Review correspondence: E26; severity: Low.
[S] Listed attributes whose read fails with these errors are stored with empty values.
[S] Restore then sets those attributes empty instead of omitting them.
[A] Frequency on SMB/CIFS mounts or list/get races is unmeasured.

## Evidence

- [S] `internal/fs/node_xattr.go:17-20` — `getxattr` returns `b, handleXattrErr(err)`.
- [S] `internal/fs/node_xattr.go:55-56` — the errors map to nil.
- [S] `internal/fs/node_xattr.go:110-120` — the attribute is appended when the error is nil.

## Prior-Art

Coverage: local ledger searched on 2026-10-21; no duplicate found.
Gaps: canonical upstream issues, pull requests, discussions, and releases not searched.

Contribution fit: unresolved until prior art is checked.

## Proposed-Change

Skip attributes whose get error was swallowed.

## Scope-and-Constraints

- Preserve: tolerating unsupported xattr filesystems.
- Exclude: restore-side xattr handling.
- Cost: small change to the `getxattr` result contract.

## Verification

- Stub `xattr.LGet` to return ENOATTR for a listed name → the node omits the attribute.

## Publication-Blockers

- Runtime reproduction and upstream prior art are missing.
- Authorized-Work and Publication-Target are not selected.

## Next-Action

Summary: Check empty-xattr prior art
Action: Search upstream issues and pull requests for empty xattrs recorded from SMB/CIFS sources.
Done-When: Search coverage and candidate classifications are recorded.
