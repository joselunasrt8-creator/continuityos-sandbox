# External Stranger Comprehension (Loop 5)

## Question

> Can a new user understand Merge Guard's value in 5 minutes?

## Method

Six comprehension questions, answerable solely from
[`README.md`](./README.md) and the documents it links
(`VALIDATION.md`, `DEPENDENCY_ASSESSMENT.md`,
`LOAD_BEARING_READINESS.md`, `NULL_ENFORCEMENT_PROOF.md`). A reader with
no prior context should be able to read the README top-to-bottom, follow
at most one link per question, and answer all six within 5 minutes.

## The 6 questions

1. **What does Merge Guard check — the PR's content, or the PR's
   identity?**
2. **What are the five fields that make up a PR's "identity object"?**
3. **What does it mean if Merge Guard reports `result: VALID`? What does
   it mean if it reports `result: NULL`?**
4. **What happens to a PR if the required `merge-guard` check reports
   `NULL`/`failure` — does it merge anyway?**
5. **Where does the proof of a Merge Guard run live, and what's in it?**
6. **Right now, in *this* repo, is Merge Guard just informational, or does
   it actually block merges? How do you know?**

## Answer key (with source)

1. **Identity, not content.** "A normal CI check tests the *content* of a
   PR (does it build, do tests pass). Merge Guard tests the *identity* of
   the PR itself" — `README.md`, "Why is this different from a normal CI
   check?".
2. **`repo`, `pr_number`, `head_sha`, `base_sha`, `actor`** — `README.md`,
   "What is Merge Guard?".
3. **`VALID`** = all five fields present and non-empty; action exits `0`
   and uploads `MERGE_GUARD_PROOF.json` with `result: "VALID"`. **`NULL`**
   = one or more fields missing/empty; action **fails closed** (exits
   non-zero, `result: "NULL"`, lists `missing_fields`) — `README.md`,
   "What does VALID mean?" / "What does NULL mean?".
4. **No — it's blocked.** `merge-guard` is a *required* status check on
   `main`. A `NULL` result makes the check report `failure`, and GitHub
   reports the PR's `mergeable_state` as `"blocked"` —
   `NULL_ENFORCEMENT_PROOF.md`, classification `BLOCKED_NULL_CONFIRMED`
   (PR #9).
5. **`MERGE_GUARD_PROOF.json`**, uploaded as a workflow artifact named
   `MERGE_GUARD_PROOF` on every run, and also written to the job's step
   summary. It contains `proof_id`, `canonical_payload`, `canonical_hash`,
   `result`, and `missing_fields` — `README.md`, "What is Merge Guard?"
   and `VALIDATION.md` Section 2/3 (real examples from PR #2 and the
   NULL-check workflow).
6. **It actively blocks merges (`LOAD-BEARING_ACTIVE`).** Branch
   protection on `main` requires the `merge-guard` check.
   `LOAD_BEARING_READINESS.md` Section 6 shows a `VALID` PR (#8) reported
   as mergeable with `merge-guard` listed as a required, passed check, and
   `NULL_ENFORCEMENT_PROOF.md` shows a `NULL` PR (#9) reported as
   `"blocked"`. Together: "this is part of what 'mergeable' means for the
   repo — not just an informational badge" (`README.md`, "Why require it
   before merge?").

## Result

All six questions are answerable from the README plus at most one linked
document, with a direct quote or citation as the source for each answer.
No question requires reading source code (`check.mjs`, `action.yml`) or
external context.

**Classification: `COMPREHENSIBLE_IN_5_MINUTES`** — a stranger arriving at
this repo with no prior ContinuityOS context can read `README.md`, follow
one link per question, and correctly explain what Merge Guard is, what
VALID/NULL mean, what happens on failure, where the proof lives, and
whether it's currently load-bearing in this repo — within the 5-minute
budget.
