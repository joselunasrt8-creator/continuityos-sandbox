# Version Upgrade Proof: `@v0.1.0` → `@v0.1.1` (Loop 8)

## Question

> Can a consumer safely move from `@v0.1.0` to `@v0.1.1` without losing
> legitimacy?

## Proof required

> Version upgrade PR passes VALID/NULL expectations and documents
> validator continuity.

## Background

`LOAD_BEARING_READINESS.md` Section 2 carried forward an open item:

> `v0.1.0` was cut one commit before the `mindshift-demo` README fix
> (`c3d243f` on `ContinuityOS-`), so the action's *own* README at the
> `v0.1.0` ref still shows the stale install path. ... It is carried
> forward as a documentation-only item for a future `v0.1.1` tag on
> `ContinuityOS-`.

This document assesses readiness to cut and consume that `v0.1.1` tag.

## Validator continuity check

`v0.1.0` points at `83aac5a` on `joselunasrt8-creator/ContinuityOS-`.
`c3d243f` (current `main`, the proposed `v0.1.1` commit) is one commit
later — exactly the `docs: fix stale mindshift-demo reference in Merge
Guard install snippet (#1974)` commit.

A diff of the action's executable surface between the two commits:

```
$ git diff --stat v0.1.0 c3d243f -- actions/continuity-merge-guard/
 actions/continuity-merge-guard/README.md | 12 +++++++++++-
 1 file changed, 11 insertions(+), 1 deletion(-)
```

`action.yml`, `check.mjs`, `test.mjs`, and `fixtures/` are **byte-identical**
between `v0.1.0` and `c3d243f`. Only `README.md` (documentation) changes.

## VALID / NULL expectations

Because the canonicalization function, hashing algorithm
(`canonicalize` + `sha256Hex`), and exit-code behavior in `check.mjs` are
unchanged:

- The `v0.1.0` `VALID` example from `VALIDATION.md` (Issue #1, PR #2,
  `canonical_payload` → `canonical_hash`
  `703ab58e6ca63166cefea404c0d6ef74819325e570b1a50d1fff6f7b4e9755c0`) would
  produce the **identical** `canonical_hash` for the same payload under
  `@v0.1.1` — the hash function did not change.
- The `v0.1.0` `NULL` example (`canonical_hash`
  `3fed7e9a572d49437da341abb3842ec240ac956c130514c61e62a8438e2d2242`,
  `missing_fields: ["actor"]`) would likewise be reproduced exactly under
  `@v0.1.1`.
- The required-check semantics proven in `NULL_ENFORCEMENT_PROOF.md`
  (`NULL → failure → BLOCKED_NULL`) and `LOAD_BEARING_READINESS.md`
  Section 6 (`VALID → success → mergeable`) depend only on `check.mjs`'s
  exit-code behavior, which is unchanged.

**Conclusion: validator continuity is provable by code-identity, not just
by re-running the checks.** A `v0.1.0` → `v0.1.1` consumer would see
*zero* change in `result`, `proof_id` format, `canonical_hash` for a given
payload, or pass/fail behavior — only the action's own README (an
informational document, not part of the proof object) differs.

## Upgrade status

| Item | Status |
|---|---|
| Validator continuity (code diff) | **Done** — see above, byte-identical executable surface |
| `v0.1.1` tag published on `ContinuityOS-` | **Not done** — tag creation is a repository-admin action outside this session's push scope (the same class of action as the branch-protection change in `LOAD_BEARING_READINESS.md` Section 6, performed by the repo owner outside session tooling) |
| Sandbox workflows updated to `@v0.1.1` | **Not done** — deferred until the tag exists, to avoid pinning to a non-existent ref. This repo's workflows remain correctly pinned to `@v0.1.0`, which is fully functional and unaffected by this assessment |
| `ContinuityOS-` source docs reference `@v0.1.1` | See `Loop 9` alignment in `joselunasrt8-creator/ContinuityOS-` — source docs continue to recommend `@v0.1.0` as the published, pinnable version until `v0.1.1` exists |

## Migration path (once `v0.1.1` is tagged)

1. Maintainer tags `v0.1.1` at `c3d243f` on `ContinuityOS-`.
2. This repo's `.github/workflows/continuity-merge-guard.yml` and
   `continuity-merge-guard-null-check.yml`: change
   `@v0.1.0` → `@v0.1.1` (one-line `uses:` edit, both files).
3. Open a PR with that change. Expectation, per the continuity analysis
   above:
   - `merge-guard` reports `result: VALID`, `conclusion: success` (same
     as PR #8 under `@v0.1.0`), because the PR's identity object is
     unaffected by the version bump and the validator code is identical.
   - The required check passes; the PR is mergeable on that basis (subject
     to the same independent review-requirement gate noted in
     `LOAD_BEARING_READINESS.md` Section 6).
4. Record the PR number, run id, and `proof_id`/`canonical_hash` here as
   the closing evidence for this upgrade.

## Classification

```text
VALIDATOR_CONTINUITY_PROVEN   <-- this assessment (code-identity diff)
TAG_PUBLISHED                 -- pending maintainer action
UPGRADE_EXECUTED_AND_VERIFIED -- pending TAG_PUBLISHED
```

**Decision:** `VALIDATOR_CONTINUITY_PROVEN`. The `@v0.1.0` → `@v0.1.1`
move is provably a no-op for `result`, `canonical_hash`, and pass/fail
behavior, because the only change between the two refs is to a
documentation file outside the action's executable surface. A consumer
can move from `@v0.1.0` to `@v0.1.1` **without losing legitimacy** the
moment the tag exists — legitimacy here meaning "the same PR identity
object produces the same VALID/NULL verdict and the same `canonical_hash`
under both refs", which is guaranteed by code-identity rather than by
re-running the check. Publishing the tag and bumping this repo's
`uses:` lines is the remaining, low-risk, mechanical step (Migration path
above), tracked here for whenever `v0.1.1` is published.
