# External Operator Simulation Report

## Mandate

Act as an external operator with **no prior ContinuityOS knowledge**, on a
fresh container with a clean clone of `ContinuityOS-` and
`continuityos-sandbox`, using **only what the README and linked docs say**.
Attempt:

- the **governed demo** (VALID / REPLAY_NULL / POLICY_NULL), as advertised in
  `ContinuityOS-/README.md`
- installing **Merge Guard** into a fresh repository (VALID / NULL / replay)

Record commands, timings, confusion points, undocumented assumptions,
documentation defects, missing dependencies, and adoption blockers.

This is an **agent dependency proof** (does it install and run as
documented?), not a human dependency proof (would someone choose to depend on
it?) — see the framing this report was commissioned under.

## Environment

- Fresh container, `joselunasrt8-creator/ContinuityOS-` cloned, no
  `node_modules`.
- `node v22.22.2`, `npm 10.9.7`. Both pre-installed; README does not state a
  minimum Node version for the root project (it does for the conformance pack:
  "Node.js >= 18, shell").

---

## Path A — Governed Demo (`npm install && npm run demo`)

Commands run exactly as in `README.md`:

```bash
npm install     # 9.975s real
npm run demo    # 0.567s real
```

**Result: works exactly as documented, first try, no flags, no config, no
credentials.**

`npm install` added 68 packages, 0 vulnerabilities. `npm run demo` produced
JSON containing all three states the README promises:

| State | README claim | Observed |
|---|---|---|
| VALID | `validated_object_hash == executed_object_hash`, proof + lineage persisted | ✅ exact match |
| Replay NULL | `no_new_proof`, `no_new_lineage`, `reason_class: REPLAY_NULL`, `denial_reason: REPLAY_NONCE_CONSUMED` | ✅ exact match |
| Policy NULL | same shape, `reason_class: POLICY_NULL`, `denial_reason: PATH_NOT_ALLOWED` | ✅ exact match |

The only differences from the recorded transcript in
`demo/portability/RECORDED_DEMO.md` are non-deterministic fields (hashes that
depend on a wall-clock-derived `seed.md`, `correlation_id`, timestamps) — the
*shape* and *invariants* are identical. `git status` after the run is clean;
the demo is a pure in-memory adapter with no filesystem side effects.

**Time to success: ~10.5 seconds, zero confusion.** This is the strongest
part of the funnel — there is nothing to get wrong.

Also ran (not required by the "fastest way" path, but referenced lower in the
README as additional evidence):

```bash
node actions/continuity-merge-guard/test.mjs   # 0.065s — 4/4 PASS
node conformance/pack-v1/harness.mjs           # 0.069s — 15/15 PASS
```

Both pass cleanly and match the README's claimed output blocks verbatim
(`MERGE_GUARD_CONFORMANCE_COMPLETE`, `PACK_V1_CONFORMANCE_COMPLETE`, etc.).

⚠️ Side effect noted and reverted: `conformance/pack-v1/harness.mjs` rewrites
`conformance/pack-v1/conformance-pack-v1-evidence.json` on every run (new
`generated_at` timestamp), which shows up as a dirty working tree (`git
status`) immediately after following the README's own instructions. Not
harmful, but an operator who runs this before committing other work will see
an unexplained modified file. Minor documentation gap: nothing in the README
or `conformance/pack-v1/README.md` mentions this file is regenerated on every
run.

---

## Path B — Install Merge Guard into a fresh repository

### Step 1: portability claim

`actions/continuity-merge-guard/README.md` claims:

> This directory is self-contained ... no external npm dependencies. It can
> be copied into any repository unmodified.

Test: created a brand-new empty git repo (`/tmp/fresh-operator-repo`,
`git init`, nothing else), copied **only** `actions/continuity-merge-guard/`
into it, ran:

```bash
node actions/continuity-merge-guard/test.mjs
```

**Result: 4/4 PASS, identical output to the in-place run.** The portability
claim holds — confirmed with zero `npm install`, zero shared state with the
rest of `ContinuityOS-`.

### Step 2: VALID / NULL / "replay" via the action's actual entrypoint

`action.yml` is a composite action whose only step is:

```bash
node "${{ github.action_path }}/check.mjs"
```

with five `MERGE_GUARD_*` env vars set from the action's `inputs:`. There is
no GitHub Actions runner available in this environment, so the composite
action itself cannot be executed end-to-end — but its exact invocation
contract (env vars → `check.mjs` → stdout + `MERGE_GUARD_PROOF.json` +
`GITHUB_OUTPUT`/`GITHUB_STEP_SUMMARY` appends + exit code) is fully
reproducible locally and was exercised directly:

**VALID** (all five fields populated, as a real `pull_request` event would):

```
ContinuityOS Merge Guard — result=VALID
proof_id=MERGE_GUARD-42-abc123de
canonical_hash=3c5c8a9b...
exit=0
```

→ `MERGE_GUARD_PROOF.json` written, `result: "VALID"`, `missing_fields: []`.
Exact match to the README's documented output shape.

**NULL** (empty `actor`, simulating
`github.event.pull_request.user.login` being unavailable):

```
ContinuityOS Merge Guard — result=NULL
proof_id=MERGE_GUARD-43-cccccccc
canonical_hash=40bf4c84...
missing_fields=actor
NULL — missing required field(s): actor
exit=1
```

→ `MERGE_GUARD_PROOF.json` written with `result: "NULL"`, `missing_fields:
["actor"]`, **and the process exits 1** — which is what makes the GitHub
required-status-check report `failure`. Exact match to the documented
fail-closed behavior, and consistent with `continuityos-sandbox`'s own
`NULL_ENFORCEMENT_PROOF.md` (PR #9, `BLOCKED_NULL_CONFIRMED`).

**"replay"** — resubmitted the *exact same* identity object
(`repo`/`pr_number`/`head_sha`/`base_sha`/`actor` all identical to the VALID
run above) a second time:

```
ContinuityOS Merge Guard — result=VALID
proof_id=MERGE_GUARD-42-abc123de
canonical_hash=3c5c8a9b...   (identical)
exit=0
```

`diff` against the first run shows **only `generated_at` differs**.

### Finding: Merge Guard v1 has no replay concept

The README's top-level "Try the Governed Demo" section establishes a
three-state vocabulary — **VALID / Replay NULL / Policy NULL** — and the task
mandate (reasonably) expected the same three states from Merge Guard. They
are **not the same contract**:

- The Stage-1 governed-execution gateway (`npm run demo`) tracks a
  `replay_nonce` per `decision_id` and returns `REPLAY_NULL` /
  `REPLAY_NONCE_CONSUMED` on reuse.
- Merge Guard v1 (`check.mjs`) is **stateless and idempotent by design**: it
  is a pure function of the five identity fields, with no nonce, no
  persistence between runs, and no notion of "already used". Resubmitting an
  identical PR-identity object is — correctly, by its own spec — `VALID`
  again, every time.

This is *consistent* with Merge Guard's documented scope ("v1 ... is a
legitimacy check on object identity, not a review system ... policy engine"),
and "replay binding" is not even listed under Merge Guard's "v2 (not yet
built)" backlog. But neither `actions/continuity-merge-guard/README.md` nor
the top-level README **says explicitly that Merge Guard has no replay
protection**, even though the top-level README puts "replay resistance" in
its list of core runtime principles immediately above the Merge Guard
section. An operator who read the governed-demo section first (as the README
recommends — "the fastest way to see the runtime in action") and then jumped
to Merge Guard could reasonably assume the same replay semantics apply.
**Recommendation: add one sentence to `actions/continuity-merge-guard/README.md`'s
"What this proves (v1)" / "does NOT validate" list stating that repeated
submission of an identical identity object is expected to re-evaluate to the
same result (idempotent, not replay-tracked).**

### Step 3: workflow YAML — install path correctness check

Copied the exact `uses:` snippet from the **current `main` branch** of
`ContinuityOS-/README.md` into `/tmp/fresh-operator-repo/.github/workflows/continuity-merge-guard.yml`:

```yaml
- uses: joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0
  with:
    repo: ${{ github.repository }}
    pr-number: ${{ github.event.pull_request.number }}
    head-sha: ${{ github.event.pull_request.head.sha }}
    base-sha: ${{ github.event.pull_request.base.sha }}
    actor: ${{ github.event.pull_request.user.login }}
```

This snippet is **correct** (repo path `joselunasrt8-creator/ContinuityOS-`,
action path `actions/continuity-merge-guard`, ref `@v0.1.0`) and matches what
`continuityos-sandbox`'s own `.github/workflows/continuity-merge-guard.yml`
and `continuity-merge-guard-null-check.yml` actually use.

### 🔴 Documentation defect found, still live: `v0.1.0` tag's own README is stale

`LOAD_BEARING_READINESS.md` (Section 2) already documents that `v0.1.0` was
cut **one commit before** the README fix that replaced the stale
`mindshift-demo` reference, and flags this as "carried forward ... for a
future `v0.1.1` tag". This simulation independently re-derived and confirmed
that gap is **still open**:

```bash
git show v0.1.0:actions/continuity-merge-guard/README.md | grep -A6 'Install (2 minutes)'
git show v0.1.0:README.md | grep -A6 'uses: joselunasrt8-creator'
```

Both occurrences at the `v0.1.0` tag still read:

```yaml
- uses: joselunasrt8-creator/mindshift-demo/actions/continuity-merge-guard@main
```

— wrong repo (`mindshift-demo` instead of `ContinuityOS-`) **and** a floating
`@main` ref, inside a tag whose entire purpose is to be the pinned, stable
reference.

**Why this matters for an external operator specifically:** the current
`main` README's Merge Guard section explicitly recommends `@v0.1.0` as "the
published, pinnable version ... recommended for any consumer that treats the
Merge Guard result as load-bearing". A reasonable operator following that
recommendation might browse the repository **at the `v0.1.0` tag** (e.g. via
GitHub's tag/ref selector, or `git checkout v0.1.0`) to read "the README for
the version I'm pinning to" — and would copy a snippet that points at a
different repository entirely. `v0.1.0` has no `mindshift-demo` reference
that still works as Merge Guard, so this snippet would simply fail
(`uses:` resolves to a nonexistent action path) for anyone who copies it
verbatim instead of the `main`-branch README.

`v0.1.1` (documentation-only, byte-identical `action.yml`/`check.mjs`) is
already planned and described in the action's README as a no-op for
`canonical_hash`/`result`/`proof_id`. This simulation did not cut the tag
(tag creation is a release action outside an agent's default authority), but
confirms the fix is purely the README text already present on `main` —
cutting `v0.1.1` at current `main` HEAD would close this gap with zero
behavioral risk.

---

## Time-to-success summary

| Task | Time | Friction |
|---|---|---|
| Clone → `npm install` → `npm run demo` (VALID/REPLAY_NULL/POLICY_NULL) | ~10.5s | None |
| Merge Guard conformance (`test.mjs`, in place) | <0.1s | None |
| Copy `actions/continuity-merge-guard/` to a fresh repo and re-run `test.mjs` | <0.1s | None — portability claim verified |
| Reproduce VALID / NULL / "replay" via `check.mjs` directly | <0.1s | "replay" path doesn't exist for Merge Guard (see finding above) — not a bug, but undocumented relative to the demo's vocabulary |
| Copy install YAML from `main` README | instant | Correct |
| Copy install YAML from `v0.1.0` tag README (the version the `main` README tells load-bearing consumers to pin) | instant, but **broken** | 🔴 stale `mindshift-demo@main` reference, still live |

---

## What this simulation could *not* test

- Actually registering `merge-guard` as a required status check on a real
  GitHub repo's branch protection, and watching a real PR go
  `VALID`/`success` → mergeable or `NULL`/`failure` → `blocked`. This requires
  a live GitHub repository with Actions enabled and branch-protection admin
  access — `continuityos-sandbox`'s `LOAD_BEARING_READINESS.md` and
  `NULL_ENFORCEMENT_PROOF.md` already provide this evidence for that specific
  repo (PR #8 / PR #9). Reproducing it for a *new, independent* fresh
  repository is the human-dependency-decision boundary described in this
  task's framing: an agent can prove `WORKS`, not `REQUIRED`.

---

## Conclusion

The agent-discoverable funnel from "fresh clone" to "governed demo showing
VALID/REPLAY_NULL/POLICY_NULL" is **frictionless** (~10 seconds, exact match
to documented output). The Merge Guard wedge is genuinely portable
(self-contained, copies cleanly, VALID/NULL behave exactly as documented and
match `continuityos-sandbox`'s real enforcement evidence).

One concrete, previously-flagged-but-still-open documentation defect was
re-confirmed and pinpointed precisely: **the `v0.1.0` tag's README (both
`README.md` and `actions/continuity-merge-guard/README.md`) still contains the
stale `joselunasrt8-creator/mindshift-demo/...@main` install snippet**, which
is exactly the snippet a load-bearing-minded operator following the `main`
README's own advice ("pin to `@v0.1.0`") would be most likely to copy from.
Cutting the already-planned, documentation-only `v0.1.1` tag at current `main`
HEAD resolves this with no behavioral change (`action.yml`/`check.mjs`
byte-identical).

A second, smaller gap was identified: Merge Guard v1's idempotent
(non-replay-tracked) behavior is not stated explicitly, and could surprise an
operator coming from the governed demo's three-state (VALID/Replay
NULL/Policy NULL) vocabulary.

No missing dependencies, no broken `npm install`, no undocumented setup
steps, no credentials required for either path tested.
