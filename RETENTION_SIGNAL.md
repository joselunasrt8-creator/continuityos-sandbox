# Dependency Retention Signal (Loop 10)

## Question

> Does the external operator still want the check after using it?

## Proof required

> Feedback says "keep this required" or identifies why not.

## Operator perspective

This document records `continuityos-sandbox`'s feedback as the external
operator that adopted Merge Guard (Loop 1 / Issue #1), made it a required
status check (Loop 3 / Issue #5, Issue #7), and exercised it in both
directions on real PRs (Loop 3 PR #8 = `VALID`, Loop 4 PR #9 = `NULL`).

## What the operator experienced

| Dimension | Experience |
|---|---|
| Install cost | One `uses:` line per workflow, no extra deps, no secrets (`VALIDATION.md` Section 1) |
| Ongoing cost per PR | One additional job (~seconds), runs automatically on every PR |
| False positives | None observed. Every real PR (`pull_request` event) supplies all five identity fields automatically; `VALID` is the expected and observed result for normal PRs (`VALIDATION.md` Section 2) |
| False negatives / silent failures | None observed. The only `NULL` results seen were deliberately constructed (`continuity-merge-guard-null-check.yml`, and PR #9's test branch) — `merge-guard` did not spuriously fail on legitimate work |
| Enforcement when it matters | Confirmed in both directions: `VALID → success → mergeable` (PR #8) and `NULL → failure → blocked` (PR #9) |
| Governance gaps | One identified (`BLOCKED_BREAK_GLASS_REQUIRED`), and now closed by `BREAK_GLASS.md` (Loop 7) |
| Comprehension cost for new readers | `COMPREHENSION.md` (Loop 5): all 6 comprehension questions answerable from README + one linked doc within 5 minutes |
| Versioning risk | `@v0.1.0` pinned (not floating `@main`); `@v0.1.0` → `@v0.1.1` upgrade path assessed as a provable no-op (`VERSION_UPGRADE.md`, Loop 8) |

## Cost/benefit, in the operator's own terms

**Cost:** ~seconds of CI time per PR, one workflow file, one pinned
`uses:` line, and (now) a documented break-glass procedure that only
activates if the check is ever actually wrong about a PR that must merge
anyway.

**Benefit:** every PR merged into `main` now carries a hashed,
reproducible proof (`MERGE_GUARD_PROOF.json`) that its identity
(`repo, pr_number, head_sha, base_sha, actor`) was complete at merge time
— and a PR whose identity object is *not* complete cannot merge silently.
Removing the check would not break any existing functionality, but it
would remove the only mechanism in this repo that makes "the PR's identity
was checked and complete" part of what "mergeable" means
(`DEPENDENCY_ASSESSMENT.md` Phase 3, "Effect of removing Merge Guard"
row).

## Feedback

> **Keep this required.**

Reasoning:

1. The check has cost effectively nothing in friction since activation —
   zero false positives across all PRs observed since `LOAD-BEARING_ACTIVE`
   (Section 6 of `LOAD_BEARING_READINESS.md`).
2. The one prior open governance gap (break-glass) has been closed with a
   documented, accountable override path (`BREAK_GLASS.md`) that does not
   require removing or weakening the required check.
3. Both halves of the enforcement contract are independently proven on
   real PRs (#8 `VALID`/mergeable, #9 `NULL`/blocked) — the check does
   what its documentation claims, in both directions.
4. The pinned-version model (`@v0.1.0`, with a provable no-op path to
   `@v0.1.1`) means future ContinuityOS changes cannot silently alter this
   repo's required-check behavior without an explicit, reviewable version
   bump in this repo.

## What would change this answer

This feedback is conditional, not unconditional. The operator would
revisit "keep required" if:

- A real (non-synthetic) PR ever produced an unexpected `NULL` — i.e. the
  five-field identity object was incomplete for a reason outside the
  operator's control (e.g. a GitHub API/event-payload regression).
- The `@v0.1.0` → future-version upgrade path stopped being a provable
  no-op (i.e. a future version changed `result`/`canonical_hash` semantics
  for unchanged PR objects without a major version signal).
- The break-glass path (`BREAK_GLASS.md`) were invoked and found to be
  impractical in a real emergency — that would be logged in
  `BREAK_GLASS_LOG.md` (created on first use) and would be the trigger to
  re-open this assessment.

None of these have occurred as of this writing.

## Classification

```text
RETAIN            <-- this assessment
RETAIN_WITH_CHANGES
REMOVE
```

**Decision: `RETAIN`.** `merge-guard` remains a required status check on
`main`. No change to branch protection, `check.mjs`, `action.yml`, or this
repo's workflows is requested or implied by this document.
