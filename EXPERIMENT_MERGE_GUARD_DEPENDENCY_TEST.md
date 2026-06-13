# Experiment — Run First External Merge Guard Dependency Test

## Classification

```text
DEPENDENCY_FORMATION_PREP
```

## Intent

Prepare the first external dependency test for ContinuityOS Merge Guard using
`continuityos-sandbox` as the pilot environment.

The experiment asks whether an external repo PR can depend on the ContinuityOS
Merge Guard required check for merge eligibility while preserving existing repo
workflow boundaries.

## Scope

In scope:

- Documentation/experiment scaffolding in `continuityos-sandbox`.
- The sandbox repo as the external pilot environment.
- Existing Merge Guard installation and required-check workflow requirements as
  the intended test surface.
- Evidence capture requirements for VALID and NULL paths.
- Operator feedback capture.

Out of scope:

- ContinuityOS core runtime changes.
- Canon expansion.
- New governance semantics.
- New authority beyond existing sandbox workflow requirements.
- Proof registry mutation.
- Any dependency claim before an operator runs the test.

## Pilot selection

Selected pilot repo:

```text
continuityos-sandbox
```

Rationale:

- It is separate from the ContinuityOS source/runtime repo.
- It is already the sandbox/adoption surface for Merge Guard.
- It can exercise PR-level merge eligibility without mutating ContinuityOS core
  runtime.

## Merge Guard installation path to document

The operator running the test must record the exact installation path used by
the sandbox workflow, including:

- workflow file path
- `uses:` reference
- pinned version or source ref
- check/job name that appears in branch protection

Evidence placeholder:

```text
workflow file: <fill after operator inspection>
uses reference: <fill after operator inspection>
required check name: <fill after operator inspection>
source run URL: <fill after operator run>
```

## Branch protection requirement to document

The operator running the test must record the branch-protection requirement that
makes Merge Guard load-bearing for the pilot PR:

```text
protected branch: <fill after operator inspection>
required status check name: <fill after operator inspection>
other required gates still applying: <fill after operator inspection>
configuration evidence URL or screenshot/artifact: <fill after operator run>
```

The test must not add enforcement beyond what the sandbox scope already allows.
If branch protection is not already configured for the required check, this
experiment remains prep/planning until an authorized operator configures it.

## Acceptance criteria

- [ ] Sandbox repo selected as the pilot external environment.
- [ ] Merge Guard installation path documented.
- [ ] Branch protection requirement documented.
- [ ] VALID PR expected to become mergeable when the required Merge Guard check
      succeeds, subject to other existing repo gates.
- [ ] NULL PR expected to be blocked when the required Merge Guard check fails.
- [ ] Proof/evidence artifact captured for both VALID and NULL paths.
- [ ] Operator feedback capture section completed.

## VALID test path

Expected observation:

```text
PR -> Merge Guard -> VALID -> required check success -> mergeable
```

Evidence to capture:

| Field | Value |
|---|---|
| PR URL | `<fill after operator run>` |
| Head SHA | `<fill after operator run>` |
| Base SHA | `<fill after operator run>` |
| Actor | `<fill after operator run>` |
| Merge Guard run URL | `<fill after operator run>` |
| Merge Guard result | `VALID` |
| Check conclusion | `success` |
| PR mergeability state | `<fill after operator run>` |
| Proof artifact URL/name | `<fill after operator run>` |
| Other required gates | `<fill after operator run>` |

## NULL test path

Expected observation:

```text
PR or controlled NULL exercise -> Merge Guard -> NULL -> required check failure -> blocked
```

Evidence to capture:

| Field | Value |
|---|---|
| PR/test URL | `<fill after operator run>` |
| Head SHA | `<fill after operator run>` |
| Base SHA | `<fill after operator run>` |
| Missing field(s) | `<fill after operator run>` |
| Merge Guard run URL | `<fill after operator run>` |
| Merge Guard result | `NULL` |
| Check conclusion | `failure` |
| PR mergeability/blocked state | `<fill after operator run>` |
| Proof artifact URL/name | `<fill after operator run>` |
| Notes on how NULL was induced | `<fill after operator run>` |

## Proof/evidence artifact requirements

The operator should capture enough evidence for replay-safe review:

- PR URL and commit identity.
- Merge Guard workflow run URL.
- Check name and conclusion.
- `VALID` or `NULL` result.
- `missing_fields` for NULL.
- Proof artifact name/URL or archived artifact reference.
- PR mergeability or blocked state at the time evidence was captured.
- Any other pre-existing required gates that affected mergeability.

## Operator feedback capture

After running the test, capture operator feedback:

```text
operator: <fill after operator run>
date: <fill after operator run>
was installation understandable?: <yes/no + notes>
was required-check behavior understandable?: <yes/no + notes>
did VALID behavior match expectation?: <yes/no + notes>
did NULL behavior match expectation?: <yes/no + notes>
was evidence sufficient for later review?: <yes/no + notes>
would operator retain this required check?: <yes/no/undecided + notes>
friction or ambiguity observed: <fill after operator run>
```

## Non-goals preserved

- No runtime behavior change.
- No core repo mutation.
- No workflow enforcement change unless already required by sandbox scope.
- No authority model change.
- No proof registry mutation.
- No dependency claim until an operator actually runs it.

## Completion rule

This experiment remains `DEPENDENCY_FORMATION_PREP` until an authorized operator
runs the VALID and NULL paths and records the required evidence. Only then can a
separate document or issue update classify the result as a dependency claim.
