# Dependency-Formation Ceiling — Primitive Gate HALT

## Classification

```text
GATE_RESULT: HALT
CLASSIFICATION: NO_NON_TRIVIALLY_SUBSTITUTABLE_PRIMITIVE_IDENTIFIED
CEILING: DEMONSTRATION_AND_CROSS_REPO_ONLY
INDEPENDENT_CONTINUITY_DEPENDENCY: NOT_ACHIEVABLE_WITH_CURRENT_SURFACE
FILES_CREATED_FOR_PROOF: none
WORKFLOWS_CREATED: none
DEPENDENCY_CLAIMED: none
```

## Intent

Record, honestly, the ceiling on dependency formation discovered while
designing the smallest independent-consumer fixture for ContinuityOS. The goal
of that fixture was an **Independent Dependency Proof**: evidence that a real
workflow becomes observably worse (PASS → FAIL) when a ContinuityOS *continuity*
primitive is removed, under a clean counterfactual, where no trivial local
substitute can preserve the same evidence.

This document exists because that fixture was **not built**. A mandatory,
gate-first check (the Primitive Gate) halted the work before any fixture,
workflow, or proof was created. Recording the halt is the deliverable.

## Scope

In scope:

- The one independently-consumable ContinuityOS primitive: **Continuity Merge
  Guard** (`actions/continuity-merge-guard@v0.1.0`), as consumed by this repo.
- The Primitive Gate evaluation and its result.
- The classification ceiling that follows.

Out of scope (and deliberately not done):

- Building a fixture, workflow, required check, toggle, or substitution probe.
- Any new primitive, service layer, state store, or core-runtime change.
- Any dependency claim. None is made here.

## The Primitive Gate (the test that halted)

> **Primitive Gate.** Select a primitive whose *continuity* guarantee — state
> preserved across a boundary (commit N → N+1, or job A → job B) — cannot be
> reproduced by:
> - a flat file
> - an environment variable
> - a cache
> - a simple local state store
>
> If no such primitive exists: **HALT.** Do not create files. Do not create
> workflows. Do not push commits. Report the halt as the result. Do not claim
> dependency evidence.

The gate runs **first**, before any build, precisely to prevent a false-positive
fixture in which a trivial substitute silently equals the primitive (the
`file write → file write` trap).

## What was evaluated

The only packaged, independently-consumable primitive ContinuityOS ships.
Verified by code search: the source repo (`joselunasrt8-creator/ContinuityOS-`)
contains exactly one `action.yml`, at `actions/continuity-merge-guard/`.
Everything else is core runtime / conformance / proof documentation, which is
not consumable as a single external call site and would pull in additional
architecture.

Source inspected (not documentation — the actual logic):

- `actions/continuity-merge-guard/check.mjs` — decision logic.
- `actions/continuity-merge-guard/canonical.mjs` — canonicalize + SHA-256.
- `actions/continuity-merge-guard/attribution.mjs` — attribution classifier.
- `actions/continuity-merge-guard/action.yml` — inputs/outputs/runs.
- `.github/workflows/continuity-merge-guard.yml` (this repo) — the call site.

What Merge Guard does:

- Takes `{repo, pr_number, head_sha, base_sha, actor}` plus optional
  policy/attribution inputs.
- Checks **completeness** of the five required fields → `NULL` (fail-closed,
  exit 1) if any are missing/empty.
- **Canonicalizes + SHA-256 hashes** the payload → `canonical_hash`, `proof_id`.
- Optional attribution layer: conflicting *authoritative* authorship signals →
  `identity_ambiguous` → `NULL`.
- Emits a `MERGE_GUARD_PROOF.json` artifact.

## Decisive finding: the primitive is stateless

The gate requires a *continuity* guarantee — state carried across a boundary.
Merge Guard carries none:

- Every decision is a **pure function of inputs available within a single
  workflow run**.
- It **never reads any prior run's output.** The proof artifact is uploaded but
  is never consumed as an input to a later decision. There is no chain, no
  carryover, and no boundary-A → boundary-B verification.
- `canonical_hash = sha256(canonicalize(current inputs))`. No external state
  enters the computation.

Because nothing is preserved across a boundary, there is **no continuity
guarantee for the gate to qualify.** Merge Guard is a legitimacy/completeness
gate, and legitimacy ≠ continuity.

## Substitution probe

Each guarantee Merge Guard provides, tested against the forbidden substitution
classes:

| Guarantee | Reproducible by trivial local state? | How |
|---|---|---|
| Field completeness / fail-closed | **Yes** | `[ -z "$actor" ] && exit 1` over **env vars** |
| Deterministic canonical hash | **Yes** | sorted **flat file** → `sha256sum` (portable across runners) |
| Required-check enforcement | **Yes** (a branch-protection property, not a primitive property) | any non-zero exit + required status check |
| Attribution ambiguity → NULL | **Yes** | pure function of the current PR's metadata; a local script reproduces it |

Every guarantee is **local and stateless**, so each is reproducible by a flat
file / env var / `sha256sum` / a few conditionals. In counterfactual terms, the
**substitute wins**. And because there is no cross-boundary state at all, this is
not even *partial* evidence for a continuity dependency — there is no continuity
dependency to degrade.

## Why HALT rather than build

Forcing Merge Guard into a continuity-proof fixture would manufacture a
`file → file` comparison that the substitute trivially equals. The fixture would
"pass," and the experiment would **overstate** its result. The Primitive Gate was
written to catch exactly this. Halting is the correct, honest outcome:

```text
"No non-trivially-substitutable primitive identified."
```

No fixture, workflow, required check, toggle, or substitution probe was created.
No dependency was claimed.

## Dependency-formation ceiling

With the current ContinuityOS surface, the reachable evidence classes are:

| Class | Reachable? | Basis |
|---|---|---|
| **A. Demonstration Proof** | **Yes** | Merge Guard runs, emits VALID/NULL + proof artifact. |
| **B. Cross-Repo Dependency** | **Yes** | This repo consumes the primitive from a separate source repo as a required check (existing VALID/NULL enforcement records). |
| **C. Independent *Continuity* Dependency** | **No** | The one consumable primitive is stateless; its guarantees are reproducible by trivial local state. |
| **D. Partial Evidence Only** | n/a | Not applicable — there is no continuity workflow to partially degrade. |

The ceiling is **B**. **C is not achievable** without changing the primitive
itself.

## What would lift the ceiling (not done here; requires explicit authorization)

An Independent Continuity Dependency Proof would require a primitive that
**carries verified state across a boundary** such that a flat file / env var /
cache cannot reproduce the guarantee — for example, a chain-verifying check where
run N reads run N−1's proof and fails closed on a broken chain. That is a **new
primitive in the source repo**, i.e. architecture work, and sits outside the
"smallest fixture" boundary. It is recorded here only as the condition that would
change the result — not as a proposal to build.

## Finality

This document records a halt and a ceiling. It does not assert that any fixture
was built, any workflow ran, any check passed or failed, or any dependency was
formed. No proof was fabricated. The current ContinuityOS consumable surface
supports Demonstration and Cross-Repo evidence; it does not support Independent
Continuity Dependency Proof.
