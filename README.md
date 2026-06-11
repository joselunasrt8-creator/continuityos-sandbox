# continuityos-sandbox

External consumer / adoption-proof environment for
[ContinuityOS](https://github.com/joselunasrt8-creator/ContinuityOS-).

This repo is intentionally separate from the ContinuityOS source repo. Its
purpose is to validate that ContinuityOS components — starting with the
Continuity Merge Guard GitHub Action — can be installed and used by a repo
that is not where ContinuityOS was built.

```text
ContinuityOS source repo (joselunasrt8-creator/ContinuityOS-)
→ provides Merge Guard (actions/continuity-merge-guard/)

continuityos-sandbox (this repo)
→ consumes Merge Guard as an external validation surface
```

## What is Merge Guard?

A small GitHub Action that checks whether a pull request's identity —
`{repo, pr_number, head_sha, base_sha, actor}` — is complete, before the
PR is treated as mergeable. It hashes that identity into a proof artifact
(`MERGE_GUARD_PROOF.json`) attached to every run.

## What problem does it solve?

PRs (especially agent-authored ones) can be merged without a verifiable,
reproducible record of *what exactly* was reviewed and merged. Merge Guard
gives every PR a canonical, hashed identity object and a pass/fail
judgment on whether that object is complete — a minimal, auditable
"this PR is what it claims to be" check.

## Why require it before merge?

Once added as a required status check, no PR can merge unless its
identity object is `VALID`. This makes "the identity object was checked
and complete" part of what "mergeable" means for the repo — not just an
informational badge.

## What does VALID mean?

All five identity fields (`repo`, `pr_number`, `head_sha`, `base_sha`,
`actor`) are present and non-empty. The action exits `0` and uploads
`MERGE_GUARD_PROOF.json` with `result: "VALID"`.

## What does NULL mean?

One or more identity fields are missing or empty. The action **fails
closed**: it exits non-zero (`result: "NULL"`, `missing_fields: [...]`),
so a required check reports `failure` and merge is blocked.

## Why is this different from a normal CI check?

A normal CI check tests the *content* of a PR (does it build, do tests
pass). Merge Guard tests the *identity* of the PR itself — that the thing
being merged is fully and verifiably specified, with a hashed proof
artifact as evidence. It is a legitimacy check, not a build/test check,
and it fails closed (NULL → blocked) rather than failing open.

---

See [`VALIDATION.md`](./VALIDATION.md) for the external-consumer validation
record (Issue #1: VALID / NULL / proof artifacts / adoption friction).

See [`DEPENDENCY_ASSESSMENT.md`](./DEPENDENCY_ASSESSMENT.md) for the
dependency formation assessment (Issue #3: with/without comparison,
required-status-check recommendation, and the fail-closed
`BLOCKED_NULL` / `BLOCKED_UNKNOWN` / `BLOCKED_BREAK_GLASS_REQUIRED`
model).

See [`LOAD_BEARING_READINESS.md`](./LOAD_BEARING_READINESS.md) for the
load-bearing readiness assessment (Issue #5: canonical install path,
`@v0.1.0` version strategy, adoption-friction status, and documented
required-status-check configuration). Current classification:
`LOAD-BEARING_READY`.
