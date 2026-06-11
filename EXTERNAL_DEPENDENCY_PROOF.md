# External Dependency Proof (Loop 6)

## Question

> Will another repo/operator install and keep Merge Guard required?

## Proof

> External repo pins `@v0.1.0`, requires `merge-guard`, and uses it for
> merge eligibility.

This document is the formal closure record for that proof. It does not
introduce new evidence — it consolidates evidence already produced by
Loops 1-4 (Issues #1, #3, #5, #7, #9) into a single load-bearing-dependency
claim, because all three required conditions are now independently
satisfied and demonstrated.

## The "external repo/operator" is `continuityos-sandbox`

This repo is, by design, "an external consumer / adoption-proof
environment for ContinuityOS" (`README.md`), not the repo where Merge
Guard was built (`joselunasrt8-creator/ContinuityOS-`). It is operated as
a normal consumer repo: its own `main` branch, its own branch protection,
its own PRs.

## Condition 1 — Pins `@v0.1.0`

`.github/workflows/continuity-merge-guard.yml` and
`.github/workflows/continuity-merge-guard-null-check.yml` both reference:

```yaml
uses: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0
```

`v0.1.0` is a published tag on `joselunasrt8-creator/ContinuityOS-`
(`83aac5a11104c6b2ef9c5ebdc23636f3f5b71d89`). This is a pinned reference,
not a floating `@main` reference — see `LOAD_BEARING_READINESS.md`
Section 2 for the migration record.

## Condition 2 — Requires `merge-guard`

Branch protection on this repo's `main` requires the `merge-guard` status
check to pass before a PR can merge. `LOAD_BEARING_READINESS.md` Section 5
documents the exact configuration (check name `merge-guard`, not the
slash-qualified `continuity-merge-guard / merge-guard`), and Section 6
records that this configuration is live (applied by the repo owner and
verified against PR #8).

## Condition 3 — Uses it for merge eligibility

This is demonstrated in both directions, by two real PRs against `main`:

| PR | `merge-guard` result | Check conclusion | `mergeable_state` | Reference |
|---|---|---|---|---|
| [#8](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/8) | `VALID` | `success` | mergeable (required check passed) | `LOAD_BEARING_READINESS.md` Section 6 |
| [#9](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/9) | `NULL` | `failure` | `"blocked"` | `NULL_ENFORCEMENT_PROOF.md`, `BLOCKED_NULL_CONFIRMED` |

`VALID → success → mergeable` and `NULL → failure → blocked` are both
observed on real PRs against the protected `main` branch, with
`merge-guard` as the deciding required check in each case. Merge
eligibility for this repo is, in fact, conditioned on a `VALID` Merge
Guard result.

## Classification

```text
INSTALLED            -- @v0.1.0 referenced in workflows (Condition 1)
REQUIRED             -- merge-guard is a required status check (Condition 2)
LOAD_BEARING         -- VALID/NULL results observed to gate real PRs (Condition 3)
```

**Decision: `EXTERNAL_DEPENDENCY_CONFIRMED`**

`continuityos-sandbox` is a repository other than
`joselunasrt8-creator/ContinuityOS-` that has installed Merge Guard at a
pinned version, configured it as a required status check, and had real
pull requests pass and fail merge eligibility based on its result. This
satisfies Loop 6 in full: an external operator both *adopted* and
*depends on* Merge Guard for its definition of "mergeable".

## Continuity / retention

Whether this dependency is *retained* going forward — i.e. whether the
operator still wants the check after living with it — is assessed
separately in [`RETENTION_SIGNAL.md`](./RETENTION_SIGNAL.md) (Loop 10).
