# Dependency Formation Assessment (Issue #3)

This document assesses whether ContinuityOS Merge Guard is, or could
become, **load-bearing** for merge legitimacy in this sandbox repo, and
defines the dependency boundary and fail-closed model required to get
there.

It builds on [Issue #1 / VALIDATION.md](./VALIDATION.md), which proved
external consumption (VALID and NULL results, both proof-bound, from a
single `uses:` line referencing the upstream action).

---

## Phase 1 — Baseline

Current sandbox behavior, as installed in
[`.github/workflows/continuity-merge-guard.yml`](./.github/workflows/continuity-merge-guard.yml):

```text
PR opened / synchronized / reopened against main
  -> continuity-merge-guard / merge-guard job runs
  -> VALID or NULL result computed from {repo, pr_number, head_sha, base_sha, actor}
  -> MERGE_GUARD_PROOF.json artifact uploaded
  -> result has NO effect on merge eligibility
```

**Finding:** Merge Guard is present, produces a proof-bound VALID/NULL
result on every PR, but is **not yet load-bearing**. `main` has no branch
protection rule referencing this check, so a PR can merge regardless of
`result`. This matches the issue's expected Phase 1 finding.

---

## Phase 2 — Dependency Boundary

The dependency boundary is the point where the Merge Guard `result`
output starts to control merge eligibility instead of merely being
recorded.

```text
Merge Guard status check (continuity-merge-guard / merge-guard)
  -> added to branch protection on `main` as a required status check
  -> GitHub will not allow merge until the check reports `success`
  -> a NULL result exits the action with code 1 -> check reports `failure`
  -> failure on a required check -> merge blocked
```

This boundary requires **no change to `check.mjs` or `action.yml`** — the
action already exits non-zero on NULL (fail-closed by design, per
`check.mjs` line 201-204). The only change needed is a branch-protection
configuration on this repo (out of scope to apply per the issue's
non-goals; documented here as the recommendation).

Once crossed, this boundary converts the existing VALID/NULL
classification into a binding merge gate:

```text
VALID  -> required check = success -> merge eligible
NULL   -> required check = failure -> merge blocked
```

---

## Phase 3 — With / Without Comparison

| | Without ContinuityOS | With ContinuityOS (informational, today) | With ContinuityOS (required check, post-boundary) |
|---|---|---|---|
| PR identity canonicalization | None | `{repo, pr_number, head_sha, base_sha, actor}` canonicalized + sha256-hashed | Same |
| VALID/NULL legitimacy object | Does not exist | Computed and emitted every run | Computed and emitted every run |
| `MERGE_GUARD_PROOF.json` artifact | Does not exist | Generated and uploaded every run | Generated and uploaded every run |
| Effect of NULL result on merge | N/A | None — PR can still merge | Merge blocked (required check fails) |
| Effect of removing Merge Guard | N/A | No workflow change observed (purely additive) | `main` loses its only legitimacy gate; PRs with no/invalid identity object could merge silently |
| Merge depends on | Ordinary GitHub checks only | Ordinary GitHub checks only (Merge Guard result is advisory) | Ordinary GitHub checks **and** a VALID Merge Guard result |

---

## Phase 4 — Adoption Blockers

Carried forward from Issue #1 friction, reframed as blockers to crossing
the dependency boundary:

1. **Trailing-dash source repo name** (`ContinuityOS-`) is easy to
   mistype in a `uses:` reference. A typo silently breaks the
   `continuity-merge-guard` job (job fails to resolve the action), which
   — once required — would block *all* merges with an unhelpful "action
   not found" error rather than a Merge Guard result.
2. **Stale `mindshift-demo` path** in the action's own README install
   example does not match the real source repo
   (`joselunasrt8-creator/ContinuityOS-`), confusing first-time adopters
   configuring the dependency.
3. **No version tag** exists for the action, so a required check would
   pin to `@main` — a moving target. A breaking change to `check.mjs` on
   `main` would silently change the behavior of a *required* gate on this
   repo with no version boundary to roll back to.
4. **NULL requires a dedicated workflow path** (`continuity-merge-guard-null-check.yml`)
   because real PR context always supplies all five identity fields. This
   is fine for validation, but means the **NULL branch of the required
   check has never been exercised by a real PR** — only by the synthetic
   workflow.
5. **Merge Guard is not yet a required status check** — this is the
   dependency boundary itself (Phase 2), not yet crossed.

---

## Fail-Closed Dependency Model (BLOCKED_* states)

Per the issue's determination comment, LOAD-BEARING_ACTIVE must fail
closed. This extends the VALID/NULL classification with the merge-result
side of the boundary:

```text
normal path:
  PR -> Merge Guard -> VALID -> required check = success -> merge eligible

invalid path:
  PR -> Merge Guard -> NULL  -> required check = failure -> BLOCKED_NULL
  (Merge Guard ran; the legitimacy object was rejected)

failure path:
  PR -> no valid Merge Guard result (job errored, offline, no artifact,
        action unresolved, etc.) -> required check != success -> BLOCKED_UNKNOWN
  (legitimacy is unknown, not rejected — GitHub's required-status-check
   semantics already block merge on non-success for any reason, so
   BLOCKED_UNKNOWN is covered by the same gate without new logic)

break-glass path:
  separate, explicitly-governed authority object
    -> emergency scope, human owner, reason, expiry, proof,
       post-event reconciliation
    -> BLOCKED_BREAK_GLASS_REQUIRED until that object exists
  (NOT a branch-protection "include administrators" bypass — that would
   collapse break-glass into an ungoverned override)
```

**Core rule:**

```text
No VALID Merge Guard result -> no merge eligibility
```

`BLOCKED_NULL`, `BLOCKED_UNKNOWN`, and `BLOCKED_BREAK_GLASS_REQUIRED` are
all merge-blocked outcomes. The distinction is classification (rejected
vs. unknown vs. requires-separate-governance), not permission — none of
the three confer merge eligibility.

### Mapping to current implementation

- `check.mjs` already fails closed for the `BLOCKED_NULL` case: NULL
  exits 1, so a required check reports `failure`.
- `BLOCKED_UNKNOWN` does not require new validator logic: GitHub's
  required-status-check enforcement already blocks merge when a required
  check is missing, pending, or errors before producing a result —
  exactly the "no valid Merge Guard result" condition. This repo gets
  `BLOCKED_UNKNOWN` coverage for free once the boundary in Phase 2 is
  crossed.
- `BLOCKED_BREAK_GLASS_REQUIRED` has **no implementation in this repo or
  in ContinuityOS today**. Per the issue's non-goals (no new validator
  logic, no expanded canon), this assessment does not design that
  mechanism — it is recorded as the blocker that must be resolved before
  any break-glass override is introduced. Until it exists, the only
  correct behavior is: blocked stays blocked.

---

## Required-Status-Check Recommendation

Add `continuity-merge-guard / merge-guard` as a required status check on
`main`'s branch protection rule, **after** resolving blockers 1–3 above
(fix the `uses:` typo risk via a pinned tag once one exists, fix the
README path, publish `v0.1.0`). Do not enable "include administrators" as
a substitute for break-glass — that would silently reintroduce an
ungoverned bypass.

This is a recommendation only; per the issue's non-goals, branch
protection is **not enabled** as part of this assessment.

---

## Classification Decision

```text
Current state:        INFORMATIONAL
Next state:           LOAD-BEARING_READY
  reached when:       blockers 1-3 are resolved (pinned version tag,
                       corrected README, typo-safe `uses:` reference) and
                       branch protection is configured to require
                       `continuity-merge-guard / merge-guard`
Following state:      LOAD-BEARING_ACTIVE
  reached when:       LOAD-BEARING_READY config is applied to `main` and
                       a real PR has exercised both the VALID->merge-eligible
                       and NULL->BLOCKED_NULL paths under the required check
Fail-closed coverage: BLOCKED_NULL and BLOCKED_UNKNOWN are both already
                       satisfied by existing fail-closed exit behavior +
                       GitHub required-status-check semantics once
                       LOAD-BEARING_ACTIVE is reached.
                       BLOCKED_BREAK_GLASS_REQUIRED has no governed
                       implementation and remains an open blocker.
```

**Decision:** The sandbox should move from INFORMATIONAL toward
LOAD-BEARING_READY by resolving adoption blockers 1–3 (version pin,
README fix, typo-safe reference) as a prerequisite. Enabling branch
protection (the LOAD-BEARING_READY -> LOAD-BEARING_ACTIVE transition)
remains a deliberate, separately-approved step, not part of this
assessment.

---

## Exit-Condition Assessment

A concrete dependency boundary has been identified: adding
`continuity-merge-guard / merge-guard` to `main`'s required status checks
converts the existing VALID/NULL proof object into a binding merge gate,
with no changes required to ContinuityOS runtime semantics. The
fail-closed model (`BLOCKED_NULL`, `BLOCKED_UNKNOWN`,
`BLOCKED_BREAK_GLASS_REQUIRED`) maps onto this boundary using GitHub's
existing required-status-check enforcement plus the action's existing
exit-code behavior — the only unresolved piece is
`BLOCKED_BREAK_GLASS_REQUIRED`, which has no governed implementation
anywhere yet and is recorded as the next dependency to design.

ContinuityOS is no longer only externally consumable: it has a defined,
documented path to becoming required for merge legitimacy in this repo.
