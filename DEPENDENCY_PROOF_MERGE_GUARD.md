# Dependency Proof — Merge Guard

## Classification

```text
MERGE_GUARD_DEPENDENCY_PROOF_COMPLETE
SAME_OWNER_CONTROLLED_REFERENCE
OUTSIDE_OWNER_PROOF_OPEN
```

## Purpose

This file satisfies the bounded artifact requested by issue #16: a single operator-facing Merge Guard dependency proof artifact with mechanism evidence, VALID evidence, NULL evidence, operator feedback, and final classification.

## Mechanism evidence

`continuityos-sandbox` consumes ContinuityOS Merge Guard as an external repository workflow dependency. The protected-branch path is:

```text
PR against main
  -> `merge-guard` required check runs
  -> ContinuityOS action emits VALID or NULL
  -> GitHub branch protection evaluates the required check
  -> merge eligibility is allowed or blocked
```

Evidence already captured in the repo:

- `EXTERNAL_DEPENDENCY_PROOF.md` records the required-check dependency and final confirmation.
- `NULL_ENFORCEMENT_PROOF.md` records the fail-closed blocked path.
- `RETENTION_SIGNAL.md` records operator retention feedback.
- `BREAK_GLASS.md` records the governed override path.

## VALID path evidence

A normal PR with populated identity fields reaches `result: VALID`, the `merge-guard` required check succeeds, and GitHub permits merge subject to the repo's other gates.

Canonical evidence: `EXTERNAL_DEPENDENCY_PROOF.md`.

## NULL path evidence

A controlled NULL path makes the required `merge-guard` check fail closed, and GitHub blocks merge eligibility because the required check did not succeed.

Canonical evidence: `NULL_ENFORCEMENT_PROOF.md`.

## Operator impact assessment

The operator retention signal is recorded in `RETENTION_SIGNAL.md`.

The dependency conclusion is that removing the required Merge Guard check would materially worsen this sandbox's protected-branch merge workflow because the repo would lose the native required-check mechanism that makes PR identity completeness part of mergeability.

## Final determination

```text
Merge Guard dependency mechanism .... COMPLETE
VALID path evidence .................. COMPLETE
NULL blocked-path evidence ........... COMPLETE
Operator retention signal ............ RETAIN
Same-owner dependency proof .......... COMPLETE
Independent outside-owner proof ...... OPEN
```

The proof is complete for the controlled same-owner sandbox reference. It does not claim independent outside-owner dependency, because this repository and the ContinuityOS source repository share the same owner. The open dependency frontier remains one unaffiliated maintainer retaining the gate because removal would make their workflow worse.

## Related tracking

- Main dependency board: `joselunasrt8-creator/ContinuityOS-` issue #2173
- Sandbox validation board: issue #26
- Original artifact request: issue #16
