# continuityos-sandbox

External consumer / adoption-proof environment for
[ContinuityOS](https://github.com/joselunasrt8-creator/ContinuityOS-).

This repo is intentionally separate from the ContinuityOS source repo. Its
purpose is to validate that ContinuityOS components — starting with the
Continuity Merge Guard GitHub Action — can be installed and used by a repo
that is not where ContinuityOS was built.

See [`VALIDATION.md`](./VALIDATION.md) for the external-consumer validation
record (Issue #1: VALID / NULL / proof artifacts / adoption friction).

See [`DEPENDENCY_ASSESSMENT.md`](./DEPENDENCY_ASSESSMENT.md) for the
dependency formation assessment (Issue #3: with/without comparison,
required-status-check recommendation, and the fail-closed
`BLOCKED_NULL` / `BLOCKED_UNKNOWN` / `BLOCKED_BREAK_GLASS_REQUIRED`
model).
