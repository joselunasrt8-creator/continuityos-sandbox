# Break-Glass Governance (Loop 7)

## Question

> What happens when Merge Guard blocks but emergency override is needed?

## Proof required

> Documented override path that preserves authority, evidence, and
> accountability.

## Background

`DEPENDENCY_ASSESSMENT.md` defined three merge-blocked outcomes once
`merge-guard` is a required check:

```text
BLOCKED_NULL              -- Merge Guard ran, identity object rejected
BLOCKED_UNKNOWN           -- no valid Merge Guard result (errored/missing)
BLOCKED_BREAK_GLASS_REQUIRED -- separate, explicitly-governed override path
```

`BLOCKED_NULL` and `BLOCKED_UNKNOWN` are both proven, working, fail-closed
states (`NULL_ENFORCEMENT_PROOF.md`, `LOAD_BEARING_READINESS.md`).
`BLOCKED_BREAK_GLASS_REQUIRED` was explicitly carried forward as an open
item: "No governed `BLOCKED_BREAK_GLASS_REQUIRED` mechanism exists in
`ContinuityOS-` or this repo" (`LOAD_BEARING_READINESS.md` Section 4,
item 6).

This document defines that mechanism. It is **documentation only** — it
does not change branch protection, `check.mjs`, or `action.yml`. It
defines the *governed* path a human operator follows when `merge-guard`
is genuinely blocking a merge that cannot wait, **without** silently
reintroducing the bypass that `LOAD_BEARING_READINESS.md` explicitly
excluded ("Do **not** enable 'Include administrators' as a substitute for
break-glass").

## Why "Include administrators: off" is not enough on its own

With "Include administrators" left **disabled** (the current, correct
configuration), a repo admin already has a technical capability: GitHub
lets an admin merge a PR despite a failing/missing required check
("Merge without waiting for requirements to be met"). That capability
exists today, with **zero** governance around it — no record of why, no
link back to the `merge-guard` proof that was overridden, no
accountability trail beyond GitHub's generic audit log entry.

Break-glass governance does not add or remove that technical capability.
It wraps the existing capability in a **mandatory paper trail**, so that
using it produces a `BLOCKED_BREAK_GLASS_REQUIRED → OVERRIDE_RECORDED`
transition instead of a silent, ungoverned merge.

## The override path

```text
1. TRIGGER
   merge-guard reports NULL (failure) or is UNKNOWN (errored/missing) on
   a PR that a repo admin determines must merge before that can be
   resolved through the normal path (e.g. fixing the PR object, or
   waiting for ContinuityOS- to recover).

2. DECLARE  (preserves accountability)
   Before merging, the admin opens a "Break-Glass Override" issue in this
   repo using the template below. This is the accountability record: who,
   why, what scope, what expiry for follow-up.

3. CAPTURE  (preserves evidence)
   The admin records the current merge-guard state for the PR being
   overridden:
     - the check run conclusion (failure / missing / errored)
     - the MERGE_GUARD_PROOF.json artifact if one exists (NULL case has
       one; UNKNOWN/errored case may not)
     - the PR number, head_sha, base_sha
   This is pasted into the Break-Glass Override issue, so the *blocked*
   state is preserved even after the override changes the PR's
   mergeable_state.

4. OVERRIDE  (preserves authority)
   The admin uses GitHub's existing "Merge without waiting for
   requirements to be met" control to merge the PR. This requires admin
   permissions — the same authority boundary that already exists today.
   No new permission is granted; the existing one is exercised
   *traceably*.

5. RECONCILE  (closes the loop)
   Within 7 days, the admin appends an entry to
   `BREAK_GLASS_LOG.md` (created on first use) linking:
     - the Break-Glass Override issue
     - the merged PR and its merge_commit_sha
     - the captured merge-guard state (Step 3)
     - the resolution: was the underlying NULL/UNKNOWN condition fixed
       afterward? (e.g. a follow-up PR that produces a VALID result for
       the same change)
   The Break-Glass Override issue is then closed, referencing the log
   entry.
```

## Break-Glass Override issue template

```markdown
## Break-Glass Override Request

- **Requested by:** <github-username, must have admin/merge-override rights>
- **PR being overridden:** #<pr-number>
- **merge-guard state at time of override:**
  - check conclusion: <failure | missing | errored>
  - result (if available): <VALID | NULL | none>
  - missing_fields (if NULL): <[...] | n/a>
  - proof artifact (if available): <link or "none produced">
  - head_sha / base_sha: <...> / <...>
- **Reason override is needed now (not after normal resolution):**
  <free text — must describe the operational consequence of waiting>
- **Scope:** <this PR only | also covers: ...>
- **Planned reconciliation date (<= 7 days):** <date>
```

## Resulting state machine

```text
merge-guard: NULL or UNKNOWN
  -> BLOCKED_NULL / BLOCKED_UNKNOWN  (default; matches existing proofs)

  -> if admin invokes override AND files Break-Glass Override issue
     (Steps 2-3 BEFORE Step 4)
       -> BLOCKED_BREAK_GLASS_REQUIRED -> OVERRIDE_RECORDED -> merged
       -> reconciled within 7 days (Step 5) -> CLOSED

  -> if admin merges WITHOUT filing the issue first
       -> UNGOVERNED_OVERRIDE (out of policy; not a defined state in this
          model — flagged for repo-owner review via GitHub's audit log,
          which independently records admin merge-overrides)
```

`UNGOVERNED_OVERRIDE` is named explicitly so it is distinguishable from
`OVERRIDE_RECORDED` in any future audit: the *capability* to merge without
the issue exists (it's a GitHub admin permission, unchanged by this
document), but using it without Steps 2-3 is **out of policy** under this
governance, not a sanctioned break-glass.

## What this document does not do

- Does not change branch protection (`Include administrators` remains
  **off**, as recommended in `LOAD_BEARING_READINESS.md` Section 5, item 4).
- Does not change `check.mjs` / `action.yml` — `merge-guard` continues to
  fail closed exactly as proven in `NULL_ENFORCEMENT_PROOF.md`.
- Does not grant any new permission. Admin merge-override is a pre-existing
  GitHub capability for repo admins; this document governs its use, it
  does not create it.
- Does not create `BREAK_GLASS_LOG.md` preemptively — that file is created
  on first real use (Step 5), so it remains a record of actual events, not
  a placeholder.

## Classification

```text
BLOCKED_BREAK_GLASS_REQUIRED  -- prior state: no governed path existed
GOVERNED_OVERRIDE_DEFINED     <-- this document
```

**Decision:** `GOVERNED_OVERRIDE_DEFINED`. The override path preserves
**authority** (admin-only, no new permission), **evidence** (Step 3
captures the blocked merge-guard state before it's overridden), and
**accountability** (Steps 2 and 5 produce a durable, linked record of who,
why, and what happened next). `BLOCKED_BREAK_GLASS_REQUIRED` is no longer
an undefined dead end — it now resolves to a documented, auditable
sequence that a repo admin can actually follow.
