# Load-Bearing Readiness (Issue #5)

This document assesses whether `continuityos-sandbox` is ready to make
ContinuityOS Merge Guard a **required status check** on `main`, without
actually enabling that requirement.

It builds on:

- [`VALIDATION.md`](./VALIDATION.md) — Issue #1, external-consumer proof
  (VALID / NULL / proof artifacts)
- [`DEPENDENCY_ASSESSMENT.md`](./DEPENDENCY_ASSESSMENT.md) — Issue #3,
  dependency boundary and `BLOCKED_*` fail-closed model, current state
  classified `INFORMATIONAL`

---

## 1. README Audit

| File | Reference | Issue | Classification | Action |
|---|---|---|---|---|
| `ContinuityOS-/README.md` | `uses: joselunasrt8-creator/mindshift-demo/actions/continuity-merge-guard@main` | Wrong source repo for the Merge Guard install example | **UPDATE** | Fixed — now `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@main` ([commit](https://github.com/joselunasrt8-creator/ContinuityOS-/commit/15e0e08753a2a99e45b236fbc548da2597291436)) |
| `ContinuityOS-/README.md` | Portability demo block: `"target_repo": "mindshift-demo"`, `mindshift-demo/issues/1954` | Recorded output of an *unrelated* portability demo (GitHub-issue-comment mutation surface), not a Merge Guard install path | **KEEP** | No change — this is a recorded transcript of a different demo against a different repo, not an installation instruction |
| `ContinuityOS-/actions/continuity-merge-guard/README.md` | `uses: joselunasrt8-creator/mindshift-demo/actions/continuity-merge-guard@main` | Same stale repo path, in the action's own canonical install snippet | **UPDATE** | Fixed — now `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0`, plus a new "Version reference" section documenting `@v0.1.0` vs `@main` ([commit](https://github.com/joselunasrt8-creator/ContinuityOS-/commit/0ffdce2df856a188b6514b1a0fc4e1def3bc3f5f)) |
| `continuityos-sandbox/README.md` | Topology statement (source repo provides Merge Guard / sandbox consumes it) | Already correct and matches the required topology statement in Issue #5 | **KEEP** | Added a pointer to this document |
| `continuityos-sandbox/VALIDATION.md` | Issue #1 record, including the friction notes that cite the `mindshift-demo` and "no version tag" issues | Historical record — friction it documents is now resolved/tracked here | **KEEP** | No change — left as the historical record; this document supersedes its open recommendations |
| `continuityos-sandbox/DEPENDENCY_ASSESSMENT.md` | Issue #3 record, recommends pinning to a tag and fixing the README before crossing the dependency boundary | Historical record — recommendations are addressed by this document | **KEEP** | No change — left as the historical record |
| `continuityos-sandbox/.github/workflows/continuity-merge-guard.yml` | `uses: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@main` | Repo path already correct; ref was floating (`@main`) | **UPDATE** | Pinned to `@v0.1.0` |
| `continuityos-sandbox/.github/workflows/continuity-merge-guard-null-check.yml` | Same as above | Same | **UPDATE** | Pinned to `@v0.1.0` |

### Required topology statement (confirmed)

```text
ContinuityOS source repo (joselunasrt8-creator/ContinuityOS-)
→ provides Merge Guard (actions/continuity-merge-guard/)

continuityos-sandbox (this repo)
→ consumes Merge Guard as an external validation surface
```

No `mindshift-demo` references remain in any Merge Guard install path.

---

## 2. Version Strategy Assessment

### Current state

```text
@main
```

All sandbox workflows and the action's own README install example
previously referenced `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@main`
— a floating reference to whatever `check.mjs` / `action.yml` happen to be
on `main` at workflow-run time.

### Target state

```text
@v0.1.0
```

`v0.1.0` is a tag on `joselunasrt8-creator/ContinuityOS-` pointing at
`83aac5a11104c6b2ef9c5ebdc23636f3f5b71d89`, which contains the v1 Merge
Guard implementation validated in Issue #1.

**Resolved:** the `v0.1.0` tag has been published. `action.yml` and
`check.mjs` at `v0.1.0` are byte-identical to the versions on `main` used
during the Issue #1 validation, so pinning to `@v0.1.0` is a no-op for
sandbox behavior. The sandbox workflows below now reference `@v0.1.0`.

> Note: `v0.1.0` was cut one commit before the `mindshift-demo` README fix
> (`c3d243f` on `ContinuityOS-`), so the action's *own* README at the
> `v0.1.0` ref still shows the stale install path. This does not affect
> the sandbox — the canonical install example in Section 3 of this
> document is correct and is the reference sandbox consumers should use.
> It is carried forward as a documentation-only item for a future
> `v0.1.1` tag on `ContinuityOS-`.

### Benefits of `@v0.1.0` over `@main`

- **Reproducible validation.** Same PR identity object + same Merge Guard
  code → same VALID/NULL result, every time. With `@main`, a later change
  to `check.mjs` could silently change the result for an unchanged PR.
- **Legitimacy boundary clarity.** If a result changes, a `@v0.1.0`
  consumer knows it's because the *PR object* changed, never because the
  *validator* changed underneath them.
- **Safe required-status-check pinning.** A required check should not
  depend on an unreviewed, unpinned upstream `main`.

### Migration path

1. Maintainer publishes tag `v0.1.0` on `ContinuityOS-` at
   `83aac5a11104c6b2ef9c5ebdc23636f3f5b71d89`.
2. Sandbox workflows (`continuity-merge-guard.yml`,
   `continuity-merge-guard-null-check.yml`) updated to
   `@v0.1.0` (done in this change set).
3. `actions/continuity-merge-guard/README.md` canonical install example
   updated to `@v0.1.0` (done in this change set).
4. Future Merge Guard changes land on `main` and are validated there;
   a new tag (`v0.1.1`, `v0.2.0`, ...) is cut only when the change is
   ready for consumers who pin to a version.

### Consumer impact

- **No behavior change.** `v0.1.0` is cut from the same commit currently
  served by `@main`, so `result`, `proof_id`, `proof_hash`, and
  `MERGE_GUARD_PROOF.json` schema are unchanged.
- **One-line `uses:` update** for any existing consumer (this repo
  included) to move from `@main` to `@v0.1.0`.
- **`@main` remains usable** for consumers who explicitly want
  bleeding-edge / pre-release behavior, with the understanding that it is
  not appropriate once a consumer makes the result load-bearing.

---

## 3. Canonical Installation Path

This is the single, copy/paste-ready installation example for a new
external consumer.

```yaml
name: continuity-merge-guard

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]

permissions:
  contents: read

jobs:
  merge-guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0
        id: merge-guard
        with:
          repo: ${{ github.repository }}
          pr-number: ${{ github.event.pull_request.number }}
          head-sha: ${{ github.event.pull_request.head.sha }}
          base-sha: ${{ github.event.pull_request.base.sha }}
          actor: ${{ github.event.pull_request.user.login }}
```

| Element | Value |
|---|---|
| Source repository | `joselunasrt8-creator/ContinuityOS-` |
| Action path | `actions/continuity-merge-guard` |
| Version reference | `@v0.1.0` (pinned; `@main` documented as an acceptable floating alternative for exploration only) |
| Required inputs | `repo`, `pr-number`, `head-sha`, `base-sha`, `actor` — all populated automatically from `github.event.pull_request.*` for a normal PR |
| Proof artifact output | `MERGE_GUARD_PROOF.json`, uploaded as workflow artifact `MERGE_GUARD_PROOF`; `result` (`VALID`/`NULL`), `proof_id`, `proof_hash`, `proof_url` exposed as action outputs |
| Required secrets / config | None |

This repo's `.github/workflows/continuity-merge-guard.yml` and
`continuity-merge-guard-null-check.yml` have been updated to this pattern.

---

## 4. Adoption Friction Report

| # | Blocker (from Issue #1 / #3) | Status | Notes |
|---|---|---|---|
| 1 | Trailing-dash source repo name (`ContinuityOS-`) easy to mistype | **Documented** | Cannot be renamed without breaking every existing `uses:` reference across the org; the canonical install snippet (Section 3) is the typo-safe copy/paste source of truth. Still Open as a structural naming risk, but mitigated by providing one canonical, copyable reference. |
| 2 | Stale `mindshift-demo` README references | **Resolved** | Fixed in `ContinuityOS-/README.md` and `actions/continuity-merge-guard/README.md` (Section 1). No remaining `mindshift-demo` references in any Merge Guard install path. |
| 3 | No version tag for stable pinning | **Resolved** | `v0.1.0` is published on `ContinuityOS-` (Section 2). Sandbox workflows now reference `@v0.1.0`. |
| 4 | NULL path requires a dedicated workflow trigger | **Documented** | Unchanged from Issue #1/#3 — real `pull_request` events always populate all five identity fields, so `continuity-merge-guard-null-check.yml` remains the only way to exercise the NULL/fail-closed path pre-merge. This is a property of GitHub's `pull_request` event payload, not something Merge Guard or this repo can fix; out of scope per Issue #5 non-goals (no validator semantics changes). |
| 5 | Merge Guard not yet a required status check | **Documented** | This is the INFORMATIONAL → LOAD-BEARING_ACTIVE transition itself (Section 5). Explicitly out of scope for Issue #5 by design — documented, not applied. |
| 6 | Break-glass unresolved | **Still Open** | No governed `BLOCKED_BREAK_GLASS_REQUIRED` mechanism exists in `ContinuityOS-` or this repo. Per Issue #5 non-goals, break-glass logic is not implemented here. Remains a tracked prerequisite for `LOAD-BEARING_ACTIVE`, not for `LOAD-BEARING_READY`. |

---

## 5. Required Status Check Documentation

This section documents — but does not apply — the configuration that
would move this repo from `LOAD-BEARING_READY` to `LOAD-BEARING_ACTIVE`.

### State machine

```text
INFORMATIONAL
  Merge Guard runs on every PR, emits VALID/NULL + proof artifact.
  Branch protection on `main` does not reference the check.
  Result has no effect on merge eligibility.

  ↓ (this issue: stable install path, version pin, README fixes,
     friction documented)

LOAD-BEARING_READY
  Install path is canonical and copy/pasteable (Section 3).
  Version reference is pinned (`@v0.1.0`, Section 2).
  Stale references are resolved (Section 1).
  Adoption blockers are resolved or explicitly scoped (Section 4).
  Required-status-check configuration is documented (this section).
  Branch protection on `main` is UNCHANGED — still does not require
  the check.

  ↓ (future issue: apply branch protection — explicitly out of scope here)

LOAD-BEARING_ACTIVE
  `continuity-merge-guard / merge-guard` is a required status check on
  `main`. VALID → merge eligible. NULL → BLOCKED_NULL (merge blocked).
  Missing/errored check → BLOCKED_UNKNOWN (merge blocked, covered for
  free by GitHub's required-status-check semantics).
```

### Configuration to apply (documentation only — NOT applied by this issue)

1. Repo Settings → Branches → branch protection rule for `main`.
2. Enable **"Require status checks to pass before merging"**.
3. Add required check: **`continuity-merge-guard / merge-guard`**
   (job name `merge-guard` from `continuity-merge-guard.yml`, exactly as
   it appears on a PR's check list).
4. Do **not** enable "Include administrators" as a substitute for
   break-glass (per `DEPENDENCY_ASSESSMENT.md`) — that would silently
   reintroduce an ungoverned bypass and is excluded from this
   recommendation.
5. Result once applied:
   - `result: VALID` → required check reports `success` → merge eligible.
   - `result: NULL` → `check.mjs` exits 1 → required check reports
     `failure` → `BLOCKED_NULL`.
   - No result (job errored / action unresolved / offline) → required
     check non-`success` → `BLOCKED_UNKNOWN` (no new logic needed —
     GitHub blocks merge on any non-`success` required check).

No settings are changed by this document. Applying step 1–4 is the
explicit, separately-approved next step that crosses
`LOAD-BEARING_READY → LOAD-BEARING_ACTIVE`.

---

## Classification

```text
INFORMATIONAL
LOAD-BEARING_READY   <-- this assessment
BLOCKED
```

**Decision: `LOAD-BEARING_READY`**

Justification against the issue's classification criteria:

- **Install path is clear** — Section 3 is a single, copy/paste canonical
  example identifying source repo, action path, version, inputs, and
  output artifact.
- **Stale references are resolved or explicitly documented** — all
  `mindshift-demo` references fixed (Section 1); the trailing-dash repo
  name and NULL-path friction are documented, not silently ignored
  (Section 4).
- **Versioning strategy is clear and applied** — `@v0.1.0` is published
  and the sandbox workflows are pinned to it (Section 2).
- **Required-status-check setup is documented** — Section 5 documents the
  exact branch-protection configuration and resulting `VALID`/`BLOCKED_*`
  behavior, without applying it.
- **Unresolved blockers are identified and scoped** — break-glass (#6) and
  the trailing-dash name (#1) are explicitly carried forward as known,
  scoped, non-blocking-for-readiness items.

No branch protection, required status checks, or validator semantics were
changed by this issue.
