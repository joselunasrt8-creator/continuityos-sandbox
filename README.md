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
