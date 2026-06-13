# WEDGE — First External Merge Guard Dependency Test

## Target user

A repository operator who wants pull-request merge eligibility to depend on a
ContinuityOS Merge Guard result without changing ContinuityOS core runtime or
expanding the repository's authority model.

For this first dependency-formation prep, the target operator is the maintainer
of `continuityos-sandbox`, acting as an external/pilot consumer rather than as
the ContinuityOS source runtime owner.

## Workflow

```text
External repo PR
  -> ContinuityOS Merge Guard required check runs
  -> Merge Guard emits VALID or NULL
  -> GitHub branch protection evaluates the required check
  -> merge eligibility is allowed or blocked
  -> proof/evidence artifact is captured for operator review
```

The workflow under test is the repository's existing pull-request workflow plus
Merge Guard as a required status check. This document does not install, enforce,
or modify that workflow; it only defines the dependency test wedge.

## Current alternative

Without this dependency wedge, the repository can rely on ordinary GitHub pull
request controls such as reviews, existing CI checks, and manual operator
judgment. Those controls may show whether code passed tests or received review,
but they do not by themselves establish a ContinuityOS Merge Guard VALID/NULL
identity result as a merge-eligibility dependency.

## Pain

The pain is not lack of CI. The pain is uncertainty about whether an external
repo can make Merge Guard load-bearing in a bounded, observable way:

- A passing PR should become mergeable only when the Merge Guard required check
  reaches `VALID` / success.
- A NULL PR should be blocked by the same existing required-check mechanism.
- Evidence should be captured without mutating ContinuityOS core runtime,
  canon, proof registry, or authority semantics.

## ContinuityOS dependency

The candidate dependency is ContinuityOS Merge Guard as consumed by this sandbox
repo through the repository's documented installation path. The dependency is
only a candidate until an operator actually runs the external test and records
proof.

This prep document makes no dependency claim. It defines what must be observed
before any later claim can be made.

## VALID path

Expected path for a normal pilot PR:

```text
PR against sandbox main
  -> Merge Guard required check runs using populated PR identity fields
  -> result = VALID
  -> required check conclusion = success
  -> GitHub reports the PR as mergeable, subject to any other existing repo gates
  -> MERGE_GUARD_PROOF.json or equivalent run evidence is captured
```

A VALID result must not bypass other existing workflow requirements. It only
satisfies the Merge Guard portion of merge eligibility.

## NULL path

Expected path for the negative pilot PR or controlled NULL exercise:

```text
PR or controlled test path
  -> Merge Guard required check runs with at least one required identity field
     intentionally absent/empty in the test branch or test configuration
  -> result = NULL
  -> required check conclusion = failure
  -> GitHub blocks merge because the required check did not succeed
  -> proof/evidence captures missing_fields and blocked merge state
```

The NULL path is a dependency-enforcement test, not a new authority model. A
NULL result does not create an override path and does not mutate the proof
registry.

## Dependency test

Run the first external Merge Guard dependency test in `continuityos-sandbox` as
the pilot external repo:

1. Confirm the Merge Guard installation path used by the sandbox repo.
2. Confirm branch protection requires the Merge Guard check by the exact check
   name used in this repo.
3. Open or identify a VALID pilot PR and capture:
   - check name
   - check conclusion
   - Merge Guard result
   - PR mergeability state
   - proof/evidence artifact location
4. Open or identify a NULL pilot PR/test path and capture:
   - check name
   - check conclusion
   - Merge Guard result
   - missing fields
   - PR blocked state
   - proof/evidence artifact location
5. Capture operator feedback on whether the required check is understandable,
   acceptable, and worth retaining.

## Success condition

Success is documentation-quality proof that the sandbox repo, as an external
pilot, can treat ContinuityOS Merge Guard as a required merge-eligibility check
using only the repo's existing workflow requirements:

```text
VALID PR -> required Merge Guard check succeeds -> PR becomes mergeable
NULL PR  -> required Merge Guard check fails    -> PR is blocked
```

The success condition is not satisfied by this scaffolding alone. It is only
satisfied after an operator runs the test and records evidence.

## Explicit non-goals

- No runtime behavior change.
- No ContinuityOS core repo mutation.
- No workflow enforcement change unless already required by sandbox scope.
- No authority model change.
- No proof registry mutation.
- No dependency claim until an operator actually runs the test.

## Classification

```text
DEPENDENCY_FORMATION_PREP
```
