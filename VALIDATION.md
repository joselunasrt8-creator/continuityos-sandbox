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
is a self-contained composite action (no external npm packages). It was
consumable on the very first try, with the `uses:` line above being the
entire integration surface.

## 2. VALID result

Source: PR [#2](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/2),
`continuity-merge-guard / merge-guard` check, run
[27321536222](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27321536222).

- `result`: `VALID`
- `proof_id`: `MERGE_GUARD-2-c7fe2a61`
- `canonical_hash`: `703ab58e6ca63166cefea404c0d6ef74819325e570b1a50d1fff6f7b4e9755c0`
- `canonical_payload`:
  ```json
  {
    "repo": "joselunasrt8-creator/continuityos-sandbox",
    "pr_number": "2",
    "head_sha": "c7fe2a61ee30c41e6186544f65fed34c1bc34687",
    "base_sha": "044535804a5ac80964476b10ee7e498d91f20408",
    "actor": "joselunasrt8-creator"
  }
  ```
- `missing_fields`: `[]`
- `MERGE_GUARD_PROOF.json` artifact: `MERGE_GUARD_PROOF` (artifact id
  `7553991151`), uploaded by the run above.

The job completed successfully (`conclusion: success`) with all five
identity fields populated automatically from `github.event.pull_request.*`
— no extra configuration was required to reach VALID.

## 3. NULL result

Real pull-request context always supplies all five identity fields
(`repo, pr_number, head_sha, base_sha, actor`), so a normal PR cannot
naturally produce a NULL result. To exercise the fail-closed path without
forking or modifying the action,
`.github/workflows/continuity-merge-guard-null-check.yml` calls the same
composite action with `actor: ''`.

Source: workflow run
[27321579808](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27321579808)
(`continuity-merge-guard-null-check`, triggered on push since
`workflow_dispatch` could not be triggered via the API token used for this
session — see friction notes below).

- `result`: `NULL`
- `proof_id`: `MERGE_GUARD-0-10fb4693`
- `canonical_hash`: `3fed7e9a572d49437da341abb3842ec240ac956c130514c61e62a8438e2d2242`
- `missing_fields`: `["actor"]`
- `canonical_payload`:
  ```json
  {
    "repo": "joselunasrt8-creator/continuityos-sandbox",
    "pr_number": "0",
    "head_sha": "10fb469313758da4e1f225ea8160de60a5da7690",
    "base_sha": "10fb469313758da4e1f225ea8160de60a5da7690",
    "actor": null
  }
  ```
- `MERGE_GUARD_PROOF.json` artifact: `MERGE_GUARD_PROOF` (artifact id
  `7554008759`), uploaded by the run above.

The action's "Run legitimacy check" step exited with code 1 (as designed
for a fail-closed NULL result, per `check.mjs`); `continue-on-error: true`
on that step let the job — and the artifact-upload step inside the
composite action, which runs `if: always()` — complete successfully so the
NULL proof was still captured.

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
  PR, since GitHub always populates the identity fields for `pull_request`
  events. A second workflow was needed to exercise that path by passing an
  empty `actor` input directly.
- The GitHub API token available to this validation session could not call
  `workflow_dispatch` (`403 Resource not accessible by integration`), so the
  NULL-check workflow was triggered via `push` instead. A human adopter with
  normal repo permissions would not hit this; it's an artifact of this
  session's token scope, not of the Merge Guard action itself.

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

ContinuityOS Merge Guard v1 is an object-identity legitimacy check: it
verifies that `{repo, pr_number, head_sha, base_sha, actor}` are all
present before a PR is considered mergeable, and produces a
canonicalized, hashed proof artifact (`MERGE_GUARD_PROOF.json`) for each
run.

In this repo, both paths were exercised successfully from a single
`uses:` line referencing the upstream action directly — no copied code,
no extra dependencies. That is the adoption proof: **a repository other
than the one where ContinuityOS was built can install Merge Guard and
produce both VALID and NULL, proof-bound results.**

Today, removing Merge Guard from this repo would have no *enforced*
effect, since it is not yet a required status check — PRs could still
merge regardless of `result`. Its current value is informational/auditable
(a proof artifact per PR). Its value would become load-bearing — i.e.
removing it would degrade the repo's workflow — once
`continuity-merge-guard / merge-guard` is configured as a required status
check (see Recommendations), at which point a NULL result would block
merges and removing the check would silently drop that gate.
