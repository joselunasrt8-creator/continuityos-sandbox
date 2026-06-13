# OPERATOR_RUN / Step 1 — Dependency Mechanism Verification

## Intent

Verify that `continuityos-sandbox` has a Merge Guard path capable of affecting
merge eligibility before proceeding to VALID vs NULL PR observation.

## Scope

Sandbox repo only. This record is evidence capture and classification only; it
does not change runtime behavior, workflow authority, proof semantics, canon
semantics, or branch-protection configuration.

## Preserved invariants

- `validated_object == executed_object` remains unchanged.
- Merge Guard action inputs, action version, and workflow triggers remain
  unchanged.
- Branch protection is observed/documented here, not mutated here.
- No ContinuityOS runtime, validator, proof registry, or canon semantics are
  changed.

## Mutation-capable surfaces reviewed

- `.github/workflows/continuity-merge-guard.yml`
- `.github/workflows/continuity-merge-guard-null-check.yml`
- Existing proof records:
  - `LOAD_BEARING_READINESS.md`
  - `NULL_ENFORCEMENT_PROOF.md`
  - `EXTERNAL_DEPENDENCY_PROOF.md`

## Check results

| Required check | Evidence | Result |
|---|---|---|
| 1. Is Merge Guard installed/configured? | `.github/workflows/continuity-merge-guard.yml` installs `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0` and supplies the PR identity fields. | yes |
| 2. Does the workflow run on `pull_request`? | `.github/workflows/continuity-merge-guard.yml` is triggered by `pull_request` events for `opened`, `synchronize`, and `reopened` against `main`. | yes |
| 3. Is the check visible on PRs? | `LOAD_BEARING_READINESS.md` records PR #8 showing `continuity-merge-guard / merge-guard` as required with the `merge-guard` job successful. `NULL_ENFORCEMENT_PROOF.md` records PR #9 with the `merge-guard` check completed as `failure`. | yes |
| 4. Is branch protection configured to require the Merge Guard check before merge? | `LOAD_BEARING_READINESS.md` records owner-applied branch protection requiring the exact check name `merge-guard` on `main`; `EXTERNAL_DEPENDENCY_PROOF.md` consolidates that condition as `REQUIRED`. | yes, documented evidence |
| 5. Can evidence be captured without changing runtime, authority, proof, or canon semantics? | This evidence pass uses existing workflow files and existing proof records only. No workflow enforcement, action code, branch protection, runtime, proof registry, or canon surface is changed. | yes |

## Required evidence

```text
workflow file path/name: .github/workflows/continuity-merge-guard.yml / continuity-merge-guard
uses reference: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0
required check name: merge-guard
branch protection config note: main requires merge-guard; applied by repo owner outside session tooling; recorded in LOAD_BEARING_READINESS.md Section 6 and consolidated in EXTERNAL_DEPENDENCY_PROOF.md Condition 2
PR where check is visible: PR #8 for VALID/success; PR #9 for NULL/failure
result state: pass observed on PR #8; fail/block observed on PR #9; no pending state captured in local evidence
```

## Branch protection screenshot/config note

A screenshot is not present in the local repository. The local, reviewable
config note is the recorded owner-applied branch-protection evidence in
`LOAD_BEARING_READINESS.md` Section 6: branch protection on `main` requires the
exact status check `merge-guard`; PR #8 showed the check as required and passed.
`EXTERNAL_DEPENDENCY_PROOF.md` repeats this as Condition 2.

## Replay and dependency implications

The dependency mechanism is present because the emitted `merge-guard` check is
not merely advisory: it is documented as a required status check on protected
`main`. Therefore GitHub branch protection can make merge eligibility depend on
whether Merge Guard reports success.

This Step 1 record proves the governing path first:

```text
PR -> continuity-merge-guard workflow -> merge-guard check -> branch protection required status check -> merge eligibility
```

Only after this mechanism is accepted should VALID vs NULL observations be used
as dependency evidence.

## Classification

```text
OPERATOR_RUN_READY
```

Reason: branch protection is documented as requiring the `merge-guard` check,
and existing PR evidence records both visibility and required-check effect. The
next operator step may proceed to VALID vs NULL PR observation without widening
runtime, authority, proof, or canon semantics.
