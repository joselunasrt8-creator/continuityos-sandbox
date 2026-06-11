# Required Check Failure Enforcement Proof (Issue #9)

This document records the Loop 4 proof: that a failing required
`merge-guard` check actually blocks merge on `continuityos-sandbox`'s
`main`, completing the inverse of
[`LOAD_BEARING_READINESS.md`](./LOAD_BEARING_READINESS.md) Section 6 / PR #8
(`result: VALID` → `success` → merge eligible).

## Question

> Can a failing / missing required `merge-guard` check actually block merge?

## Honesty constraint

A normal `pull_request` event always populates all five identity fields
(`repo`, `pr_number`, `head_sha`, `base_sha`, `actor`), per
[`VALIDATION.md`](./VALIDATION.md) Issue #1. So an ordinary PR cannot
*naturally* drive the required `merge-guard` check to `result: NULL`. To
exercise the enforcement path, this proof used a **configuration-induced**
NULL: a test PR (#9) whose
`.github/workflows/continuity-merge-guard.yml` was deliberately edited (on
that PR's branch only) to pass `actor: ''` to
`continuity-merge-guard@v0.1.0`. This is a test of the required-check
enforcement plumbing, not a claim that ordinary PRs can go NULL.

## Test PR

[PR #9](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/9)
— "Issue #9: Required Check Failure Enforcement Proof (test PR — do not
merge)". Closed without merging once evidence was captured; the
NULL-inducing workflow change never reached `main`.

## Evidence

Workflow run:
[27330487516](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27330487516)
(`continuity-merge-guard`, triggered by PR #9's `pull_request` event).

`merge-guard` job log
([job 80741452310](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27330487516/job/80741452310)):

```text
ContinuityOS Merge Guard — result=NULL
proof_id=MERGE_GUARD-9-a85d0592
canonical_hash=691ecc7ac5a411a5c3f3e9d6e5214a5d94a37a7aa50f76a64528f484da31865f
NULL — missing required field(s): actor
missing_fields=actor
##[error]Process completed with exit code 1.
```

`MERGE_GUARD_PROOF` artifact uploaded (artifact id `7557289130`, 498 bytes)
from the same run, containing `result: "NULL"`, `missing_fields: ["actor"]`,
`proof_id: "MERGE_GUARD-9-a85d0592"`.

Check run for the head commit (`a85d0592...`):

| Check | Status | Conclusion |
|---|---|---|
| `merge-guard` | `completed` | `failure` |

PR #9 `mergeable_state`: `"blocked"` — GitHub reported the PR as not
mergeable while the required `merge-guard` check was failing.

## Result classification

**`BLOCKED_NULL_CONFIRMED`**

All conditions for the strictest classification (per Issue #9's acceptance
criteria) are met:

1. The required `merge-guard` job itself emitted
   `result: NULL`, `missing_fields: ["actor"]`.
2. The check run reported `conclusion: failure`.
3. GitHub reported the PR's `mergeable_state` as `"blocked"` because of that
   required-check failure.

## Compression

- Loop 3 (PR #8) proved: `VALID → merge can proceed`.
- Loop 4 (PR #9, this document) proves: `NULL (not-VALID) → merge cannot
  proceed`.

Together these complete the enforcement pair for `merge-guard` as a required
status check on `main`: the result of the identity check is not merely
informational — it is part of what "mergeable" means for this repo, in both
directions.

## Notes / scope

- No validator semantics in `ContinuityOS-` were changed.
- No branch-protection configuration was changed (it was already configured
  per Loop 3 / PR #8).
- The independent "1 approving review required" gate observed on PR #8
  (Section 6 of `LOAD_BEARING_READINESS.md`) is unrelated to this proof and
  was not a factor in PR #9's `blocked` state — `merge-guard`'s `failure`
  conclusion alone was sufficient.
- Break-glass remains an open, separately-tracked item (Section 4 item 6 of
  `LOAD_BEARING_READINESS.md`); out of scope here.
