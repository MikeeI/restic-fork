# Issue and Pull Request Tracking

Read this index at the start of every agent session before repository work.
`FORMAT.md` owns research, lifecycle, drafting, implementation, and publication rules.
Each linked `issues/ISSUE-NNN.md` is the complete authoritative record for one root cause.
This file owns `Next finding ID` and projects current issue-file state.
`Next-Action` is the `Next-Action/Summary` projection from the issue record.
When a row disagrees with its issue file, correct the row from the issue file in the same task.

Next finding ID: ISSUE-004

## Open-Findings

| ID | Finding | State | Authorized-Work | Publication-Target | Contribution-Priority | Next-Action | External-Reference |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [ISSUE-001](issues/ISSUE-001.md) | cmd/restic: stats loads every tree of every snapshot sequentially | Investigating | Not-Selected | Not-Selected | Medium | Measure stats tree-load cost | Not published. |
| [ISSUE-002](issues/ISSUE-002.md) | cmd/restic: recover loads every repository tree sequentially | Investigating | Not-Selected | Not-Selected | High | Measure recover tree-load runtime | Not published. |
| [ISSUE-003](issues/ISSUE-003.md) | internal/walker: TreeRewriter loads and decodes each tree twice | Investigating | Not-Selected | Not-Selected | High | Measure rewrite tree loads | Not published. |

## Archived-Findings

| ID | Finding | Authorized-Work | Publication-Target | Contribution-Priority | Archive-Reason | External-Reference |
| --- | --- | --- | --- | --- | --- | --- |
