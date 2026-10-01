# Issue, Comment, and Pull Request Format

## Authority

`AGENTS.md` owns project identity, fork intent, branch roles, repository rules, and approval scope.
`ISSUES.md` owns `Next finding ID` and the compact overview of every finding.
Each `issues/ISSUE-NNN.md` is authoritative for that finding's complete state and evidence.
This file owns the workflow connecting tracking, research, drafting, implementation, and publication.
Current upstream contribution guides, issue forms, and pull request templates override these generic shapes.

- Investigate before drafting or implementing.
- Let the user choose `Authorized-Work` and `Publication-Target`.
- `Research-and-Reporting` permits issues and comments but no source implementation.
- `Pull-Request-Implementation` permits only the implementation scope recorded for the finding.
- Show the exact current draft and target before publication; publish only after the user approves them.
- Verify source claims against the current canonical upstream branch.
- Keep fork-only tracking content out of upstream contribution diffs.

## Evidence Vocabulary

Use evidence labels at the claim they qualify:

- `[O]` Observed: reproduced behavior with command, version, environment, and result.
- `[S]` Source-proven: current control flow, API ownership, or deterministic data flow proves the claim.
- `[A]` Assumed: an unverified premise is required by the claim.

Never use one entry-wide label to upgrade weaker claims.
Never convert `[S]` behavior into `[O]` impact.
Preserve exact paths, symbols, commands, outputs, URLs, revisions, dates, and drafts.

## Finding IDs and Duplicate Prevention

- IDs use `ISSUE-NNN`, start at `ISSUE-001`, have at least three digits, and remain permanent.
- `Next finding ID` in `ISSUES.md` is the only allocator.
- Re-read the complete current `ISSUES.md` immediately before allocating.
- Search IDs, titles, symptoms, root causes, symbols, references, and proposed owners across `issues/`.
- Read every plausible matching issue record completely.
- Update the existing record when it already owns the root cause.
- Allocate the current ID, create its issue file, add its index row, and increment the allocator together.
- Never reuse, renumber, or create subsystem-, status-, session-, or contribution-specific sequences.
- External numbers and URLs belong in `External-Reference`; they never replace the internal ID.

## `ISSUES.md` Projection

`ISSUES.md` provides an overview, not the complete research record.
Open rows project ID, title, State, Authorized-Work, Publication-Target, Contribution-Priority, Next-Action/Summary, and External-Reference.
Archived rows project ID, title, Authorized-Work, Publication-Target, Contribution-Priority, Archive-Reason, and External-Reference.
The issue file is authoritative when a row disagrees with it.
Correct both in the same task and never leave a known projection mismatch.

## Issue Record Contract

Every issue file contains these fields in this order:

```text
State: Investigating | Draft-Ready | Implementing | PR-Ready | Submitted | Archived
Authorized-Work: Research-and-Reporting | Pull-Request-Implementation | Not-Selected
Publication-Target: New-issue | Existing-issue-comment | New-pull-request | Existing-pull-request-comment | Not-Selected
External-Reference: <exact external URL or identifier | Not published.>
Contribution-Priority: High | Medium | Low
Root-Cause-Confidence: High | Medium | Low
Finding-Category: Correctness | Reliability | Performance | Maintainability | API | UI | Build | Test | Other
Created: <YYYY-MM-DD>
Updated: <YYYY-MM-DD>
Source: `upstream/<branch>@<commit>`
```

Every issue file contains these sections in order:

- `Root-Cause`: one cause and the behavior it owns.
- `Reach-and-Impact`: affected callers, users, states, frequency, and honest impact boundary.
- `Evidence`: exact source, reproduction, history, command, result, contract, or external evidence.
- `Prior-Art`: search coverage, relevant candidates, classifications, gaps, and target fit.
- `Proposed-Change`: smallest complete correction or ownership change.
- `Scope-and-Constraints`: behavior to preserve, compatibility limits, adoption cost, and excluded scope.
- `Verification`: narrowest checks that prove the proposed or implemented contract.
- `Publication-Blockers`: exact publication evidence still unresolved, or `None.`
- `Next-Action`: one bounded action and one observable completion condition.

Use conditional sections only when applicable: `Bug-Reproduction`, `Performance-Evidence`, `Shared-Change-Pressure`, `API-and-Compatibility`, `Pull-Request-Implementation`, `Publication-Draft`, `Submitted-Text`, and `Archive`.

## Lifecycle

- `Investigating` means required evidence, authorization, target, or direction remains unresolved.
- `Draft-Ready` means research, authorization, target, and exact publication draft are complete; it does not authorize publication.
- `Implementing` applies only to authorized `Pull-Request-Implementation` while its bounded source change is active.
- `PR-Ready` requires implementation, focused verification, commit, push, and an exact pull request draft.
- `Submitted` requires an observable external issue, comment, or pull request URL.
- `Archived` requires an Archive-Reason, no remaining current action, and storage under `issues/archive/`.
- When a finding becomes Archived, move its file and update its `ISSUES.md` link in the same change.
- Never infer resolution from inactivity or branch deletion.
- Read the final thread and linked work before archiving an externally submitted contribution.

## Next-Action

Every open record contains one compact projection and one current continuation action:

```text
Summary: <2–6 word action summary>
Action: <single bounded action>
Done-When: <observable evidence that completes it>
```

`ISSUES.md/Next-Action` must equal `Next-Action/Summary` exactly.
Replace Next-Action after completing the action; do not accumulate a task log.
Archived records use `Summary: —`, `Action: None.`, and `Done-When: None.`.

## Implementation and Archive

Pull-request implementation records use:

```text
Branch: <contribution branch>
Base: `upstream/<branch>@<commit>`
Scope: <authorized source change>
Commit: <SHA | Pending.>
Push: <fork branch | Pending.>
Checks:
- <focused command> → <observed result>
```

Archived records use:

```text
Archive-Reason: Merged | Fixed-Elsewhere | Duplicate | Upstream-Declined | Superseded | Finding-Invalidated | Not-Worth-Pursuing | Withdrawn | Other
Detail: <exact reason when Archive-Reason is Other | None.>
Evidence: <external URL, commit, release, maintainer statement, or internal proof>
Checked: <YYYY-MM-DD>
```

## Ledger Validation

Run the read-only validator bundled with `skill-fork-contribution-tracking` after every ledger mutation and before completion.
Pass the target fork root as its positional argument.
The validator checks allocator continuity, unique IDs, index-to-record links, projection equality, and lifecycle sections.
It never edits files or claims that evidence, prior art, target fit, or publication value is true.

## Publication Gate

Before publication, verify the permanent ID, issue record, exact target, current upstream behavior, prior art, evidence labels, draft, and user approval.
`Pull-Request-Implementation` must be PR-Ready and its contribution diff must exclude fork-only tracking files.
Update External-Reference and State immediately after publication.
