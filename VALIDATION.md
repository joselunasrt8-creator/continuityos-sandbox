# External Consumer Validation (Issue #1)

This document records the first external-consumer validation of
ContinuityOS Merge Guard, per
[Issue #1](https://github.com/joselunasrt8-creator/continuityos-sandbox/issues/1).

## Topology

```
continuityos-          (joselunasrt8-creator/ContinuityOS-)
  -> source repo / Merge Guard implementation
     actions/continuity-merge-guard/

continuityos-sandbox   (this repo)
  -> external consumer repo / test surface / adoption proof
     .github/workflows/continuity-merge-guard.yml
     .github/workflows/continuity-merge-guard-null-check.yml
```

## Validation path

```
Repository (continuityos-sandbox)
  -> Install ContinuityOS Merge Guard
  -> Trigger VALID PR result
  -> Trigger NULL case
  -> Capture MERGE_GUARD_PROOF.json
  -> Document adoption friction
  -> Determine whether removing Merge Guard would degrade the repo workflow
```

## 1. Install

Added `.github/workflows/continuity-merge-guard.yml`, referencing the
action directly from the source repo:

```yaml
uses: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@main
```

No other dependencies, secrets, or configuration were required. The action
is a self-contained composite action (no external npm packages).

## 2. VALID result

_Status: pending — to be filled in once the install PR's
`continuity-merge-guard / merge-guard` check has run._

- PR: TBD
- Check run: TBD
- `result`: TBD
- `proof_id`: TBD
- `canonical_hash`: TBD
- `MERGE_GUARD_PROOF.json` artifact: TBD

## 3. NULL result

Real pull-request context always supplies all five identity fields
(`repo, pr_number, head_sha, base_sha, actor`), so a normal PR cannot
naturally produce a NULL result. To exercise the fail-closed path without
forking or modifying the action, `.github/workflows/continuity-merge-guard-null-check.yml`
calls the same composite action via `workflow_dispatch` with `actor: ''`.

_Status: pending — to be filled in once this workflow has been run._

- Run: TBD
- `result`: TBD (expected `NULL`)
- `missing_fields`: TBD (expected `["actor"]`)
- `canonical_hash`: TBD
- `MERGE_GUARD_PROOF.json` artifact: TBD

## 4. Adoption friction notes

- The source repo's name includes a trailing dash (`ContinuityOS-`), which
  is easy to mistype in a `uses:` reference.
- The action's own README install example references a different repo
  (`joselunasrt8-creator/mindshift-demo/actions/continuity-merge-guard@main`),
  which does not match the actual source repo
  (`joselunasrt8-creator/ContinuityOS-`). This is a documentation defect
  that would confuse a first-time adopter.
- No version tag exists for the action yet, so consumers must pin to
  `@main` (a moving target) rather than a release.
- The "Test 3 — NULL" scenario from Issue #1 cannot be produced by a real
  PR, since GitHub always populates the identity fields for PR events. A
  second, manually-triggered workflow was needed to exercise that path.

## 5. Recommendations

- Publish a `v0.1.0` tag on `joselunasrt8-creator/ContinuityOS-` so external
  consumers can pin to a stable ref instead of `@main`.
- Fix the install example in
  `actions/continuity-merge-guard/README.md` to reference
  `joselunasrt8-creator/ContinuityOS-` instead of `mindshift-demo`.
- As a follow-up (out of scope for this milestone): add
  `continuity-merge-guard / merge-guard` as a required status check on
  this repo's `main` branch, so Merge Guard becomes part of the
  operational definition of "mergeable".

## 6. Exit-condition assessment

_Status: pending — to be completed after the VALID/NULL runs above._

ContinuityOS Merge Guard v1 is an object-identity legitimacy check: it
verifies that `{repo, pr_number, head_sha, base_sha, actor}` are all
present before a PR is considered mergeable, and produces a signed-shape
proof artifact. Removing it from this repo would mean: TBD.
