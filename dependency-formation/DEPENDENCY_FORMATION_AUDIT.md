# ContinuityOS Ecosystem Dependency Formation Audit

## Classification

```text
DEPENDENCY_FORMATION_RESEARCH
```

## Intent

Audit the `ContinuityOS-` / `continuityos-sandbox` ecosystem as it stands today
and answer one question precisely: has a **real cross-repo dependency** formed,
or only a **same-owner demonstration** of one? This complements
[`FROM_TOOL_TO_DEPENDENCY.md`](./FROM_TOOL_TO_DEPENDENCY.md), which models *why*
infrastructure becomes load-bearing in general. This audit maps the *current*
state of every primitive in the ecosystem against that model and ranks them by
present counterfactual strength, so the next action is chosen by evidence
rather than by which surface is most recently built.

## Dependency verdict

A real **repo-to-repo technical dependency** has formed around **Merge Guard**,
but a real **independent external dependency** has not yet formed. The
technical dependency is real because `continuityos-sandbox` installs
`joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.1.0`,
makes `merge-guard` a required status check on `main`, and records both sides
of the counterfactual: one PR passed `merge-guard` and became mergeable, while
another forced `result: NULL`, produced `conclusion: failure`, and GitHub
reported the PR as blocked. GitHub's own required-status-checks model means
that check must pass before merging into a protected branch. If ContinuityOS
disappeared, the sandbox's protected-branch merge workflow would lose the
check that currently participates in its definition of "mergeable."

The reason this is **not yet a true external dependency in the stronger market
sense** is that the producer repo and the consumer repo are both owned by
`joselunasrt8-creator`, and the activation, enforcement, and retention evidence
all come from that same operator boundary. The system has crossed an
installation boundary and an enforcement boundary, but not yet an
**independent trust boundary**. The workflow is load-bearing across
repositories; it is not yet load-bearing across maintainers.

That distinction matters because the core question is not "can ContinuityOS
do something?" but "what becomes worse when it is removed?" Today, the
strongest answer is narrow and concrete: **protected-branch merge legitimacy
in the sandbox gets worse**. The broader answer — "an external maintainer's
workflow gets worse" — is still not proven outside the same owner boundary.

## Producer / consumer map

The load-bearing chain is clearest when the system is treated as producer and
consumer together rather than as two unrelated repos.

| Primitive | Producer | Consumer | Dependency direction | Enforcement mechanism | Proof mechanism | Counterfactual | Current proof class |
|---|---|---|---|---|---|---|---|
| **Merge Guard** | ContinuityOS action `actions/continuity-merge-guard` emits `VALID`/`NULL`, proof hash, and proof artifact. | `continuityos-sandbox` workflow `continuity-merge-guard.yml` calls the action at `@v0.1.0`. | Producer → consumer | GitHub protected-branch required status check named `merge-guard`. | `MERGE_GUARD_PROOF.json`, a passing PR, a failing PR, docs in `EXTERNAL_DEPENDENCY_PROOF.md`. | Remove ContinuityOS and the sandbox loses the only mechanism that currently makes "PR identity was checked and complete" part of mergeability. | **Dependency proof** |
| **Agent Identity / Attribution Classification** | ContinuityOS `attribution.mjs`, `AGENT_ATTRIBUTION_SPEC.json`, and `AGENT_ATTRIBUTION_POLICY.json` classify PRs from authoritative, supporting, and heuristic signals; conflicting authoritative signals fail closed to ambiguity. | Sandbox workflow `continuity-agent-attribution-gate.yml` consumes `attribution_classification` from `continuity-merge-guard@v0.3.0`. | Producer → consumer | Enforcement is in the consumer workflow, not the producer: agent-lane branches fail unless classification is `AGENT_AUTHORED`; non-agent branches pass neutrally. | `ATTRIBUTION_DEPENDENCY_PROOF.md` now records both the pass path and the fail path with live evidence; required-check activation on branch protection remains the one pending step. | Remove ContinuityOS attribution output and the sandbox gate can no longer distinguish `AGENT_AUTHORED` from `UNKNOWN`; agent-lane enforcement degrades immediately. | **Architecture proof, demonstrated end-to-end; required-check activation pending** |
| **Merge Proof and Proof Registry** | ContinuityOS `merge-proof.yml` generates merge proofs, fetches the merge-guard attribution sidecar, and appends proof entries through PR-mediated registry persistence. | Consumer is primarily ContinuityOS itself: later audit, registry persistence, and standing-authority budget accounting. Sandbox does not consume this registry. | Producer → internal consumer | Append-only registry PR path; direct main persistence is explicitly forbidden. | `MERGE_PROOF_SPEC.json`, `merge_proof_registry.jsonl`, proof-registry PRs. | Remove it and ContinuityOS internal audit lineage worsens, but sandbox mergeability does not materially change. | **Demonstration / internal proof, not external dependency** |
| **Governance Mutation Authorization** | ContinuityOS `GOVERNANCE_MUTATION_AUTHORIZATION_SPEC.json`, singleton GMA file, and append-only GMA registry define authorization for governance/workflow mutations. | Consumer is ContinuityOS merge-governance machinery, not sandbox. | Producer → internal consumer | Fail-closed authorization tiers: explicit GMA registry, singleton fallback, or Standing Authority derivation. | Current valid GMA objects in the repo. | Remove it and ContinuityOS self-governance breaks; sandbox consumer workflows are unchanged. | **Internal dependency only** |
| **Standing Authority** | ContinuityOS `STANDING_AUTHORITY_SPEC.json` and `standing_authority_registry.jsonl` define bounded reusable authority for governance/workflow mutations. | Consumer is ContinuityOS Tier-3 admission and merge-proof attribution, not sandbox. | Producer → internal consumer | Tier-3 derivation in merge governance; budget consumption recorded through the merge proof registry. | Active standing authority records exist in the registry. | Remove it and internal governed throughput falls back to manual GMA issuance; sandbox remains unaffected. | **Internal dependency only** |
| **Sandbox Attribution Gate** | Consumer-defined workflow in the sandbox, but its signal source is produced by ContinuityOS. | Agent-lane PR mergeability in the sandbox. | Composite dependency: ContinuityOS signal → sandbox gate → GitHub check | Workflow-level fail-closed logic; required-check activation in branch protection is the one remaining administrative step. | `ATTRIBUTION_DEPENDENCY_PROOF.md`, now with both PASS and FAIL evidence rows populated. | Remove the upstream signal and the gate becomes blind; leave it non-required and it remains non-indispensable until that toggle is flipped. | **Moderate counterfactual, one step from dependency** |

The map makes the central result obvious: **Merge Guard is the only primitive
that is both externally installed and already enforced by a consumer workflow
in a way that changes merge eligibility today.** Attribution is the next
closest wedge — its demonstration evidence (both the pass path and the fail
path) is now complete; it is one administrative branch-protection toggle short
of the same status. The proof registry, GMA, and Standing Authority are
meaningful, but they are meaningful **inside ContinuityOS**, not yet as
external dependency surfaces for the sandbox.

## Ranked candidates and wedges

Ranked by present counterfactual strength across the combined system:

| Rank | Candidate | User and workflow | Pain if removed | Current state | Counterfactual strength |
|---|---|---|---|---|---|
| 1 | **Merge Guard required check** | Sandbox maintainer deciding whether a PR is mergeable on `main`. | Protected branch loses the only legitimacy gate tied to PR identity completeness. | Installed, pinned, required, and exercised in both the passing and failing direction. | **Dependency** |
| 2 | **Agent-attribution gate on agent lanes** | Maintainer deciding whether an agent-lane PR is truthfully agent-authored before merge. | Loses the ability to distinguish `AGENT_AUTHORED` from `UNKNOWN` on agent branches. | Implemented, pinned to `@v0.3.0`, with both pass-path and fail-path evidence now recorded; required-check activation is the one remaining step. | **Strong, nearly at dependency** |
| 3 | **Self-serve adoption package for the attribution gate** | An independent maintainer who wants the gate without reverse-engineering the sandbox workflow. | No copy-pasteable install path means adoption cost stays higher than it needs to be, which directly suppresses the probability of an outside-owner install. | Shipped in `ContinuityOS-`: `actions/continuity-merge-guard/examples/continuity-agent-attribution-gate.yml`, `ADOPT_AGENT_ATTRIBUTION_GATE.md`, and a README install section. | **Adoption-cost-reducing enabler, not itself dependency** |
| 4 | **Pinned validator versions** | Consumer maintainer relying on stable required-check semantics. | Floating `@main` would silently change gate behavior. | Sandbox already pins `@v0.1.0`; the attribution gate pins `@v0.3.0`. | **Moderate enabler, not itself dependency** |
| 5 | **Break-glass governance** | Maintainer facing a rare emergency when required checks are wrong or unavailable. | Removal would not destroy daily operation, but would weaken retention confidence and emergency accountability. | Documented, governance-only wrapper around existing admin capability. | **Moderate support wedge** |
| 6 | **MERGE_GUARD_PROOF artifact** | Auditor wanting PR-specific proof. | Fewer artifacts, but merge blocking still comes from job conclusion + branch protection. | Produced every run. | **Moderate evidence, weak dependency** |
| 7 | **Attribution sidecar in merge proof** | Internal audit wanting attribution lineage after merge. | Lineage quality drops, but the sandbox consumer workflow does not degrade. | Implemented inside ContinuityOS. | **Weak external dependency** |
| 8 | **Proof registry** | Internal audit and append-only history. | Internal audit gets worse; sandbox user notices little or nothing. | Active internal persistence path. | **Weak external wedge** |
| 9 | **Governance Mutation Authorization** | ContinuityOS maintainer changing governance/workflow surfaces. | Internal self-governance degrades; external consumer unchanged. | Active internal gating. | **Internal only** |
| 10 | **Standing Authority** | ContinuityOS maintainer seeking throughput for bounded governance mutations. | Falls back to manual GMA; no sandbox workflow degradation. | Active internal registry and derivation design. | **Internal only** |
| 11 | **Governed runtime demos** | Evaluator running filesystem or portability demos. | Demonstrations disappear, but no consumer workflow breaks. | Demo proof exists, not dependency proof. | **Demonstration only** |

This ranking also exposes the **false bottlenecks**. More governance
artifacts, more topology documentation, more proof infrastructure, and more
policy refinement are not the short path to external dependency, because
several of the key artifacts are explicitly non-operative internally and the
sandbox currently depends on only a tiny subset of producer surfaces: the
Merge Guard action and, now end-to-end demonstrated, the attribution outputs.
The external dependency problem was never "missing architecture." It is
"missing independent maintainer adoption" — which is itself downstream of
**adoption cost**.

The **strongest present wedge** is still **Merge Guard as a required
protected-branch check**, because it is already live and changes the merge
button. The **highest-upside next wedge** is **agent-attribution
enforcement**, because it addresses a sharper pain: maintainers increasingly
need a low-friction way to require explicit agent-authorship signals on
AI-generated PRs without touching human PRs — and that wedge now has a
self-serve install package in the producer repo, which removes the single
biggest friction point an outside adopter would have faced. The **weakest
external wedge** is the proof registry: valuable, but its current consumers
are internal, not external.

## External user and execution path

The most likely first independent dependent user is an **open-source or
startup maintainer who already receives AI-generated PRs on protected
branches and does not want to run a GitHub App, manage secrets, or redesign
review flow**. That user's existing workflow is "review PR, trust GitHub
checks, merge to `main`." Their specific pain is that AI-generated branches
and PRs can look operationally normal while lacking a bounded, explicit,
auditable declaration of identity or authorship. ContinuityOS is attractive
because the install surface is tiny, requires no secrets, and already maps
onto GitHub's native required-status-check model.

That user would adopt **Merge Guard** if they want a general legitimacy gate
for every PR, or **agent-attribution-gate** if the real pain is concentrated
in Claude/Codex/Cursor-style agent lanes. They would stay only if three things
remain true: false positives stay near zero, version drift stays explicit
through pinned tags, and emergency override is documentable without silently
weakening the gate. The sandbox evidence supports the first two conditions for
Merge Guard, and a break-glass path documents the third.

The highest-leverage dependency path is therefore not "expand ContinuityOS
architecture." It is **crossing the trust boundary with one independently
owned maintainer repo whose mergeability actually depends on ContinuityOS**.
The fastest path is to use the already-proven protected-branch check model,
sharpened to **maintainer trust for agent-authored PRs** — a pain maintainers
already feel, with an install footprint of one required check on one
protected branch.

## Final decisions

1. **Current bottleneck:** trust-boundary dependency formation. Installation,
   enforcement, and pinned-version operation already work; what is missing is
   an independently owned maintainer who relies on the system outside the
   `joselunasrt8-creator` operator boundary.

2. **Highest-leverage dependency path:** use **maintainer trust** as the entry
   point, focused on the narrower and sharper pain of **agent-lane
   attribution enforcement** — `continuity-agent-attribution-gate` in a
   non-owned repo, pinned to `continuity-merge-guard@v0.3.0`, made a required
   status check on protected branches that receive `claude/*`, `codex/*`, or
   similar PRs.

3. **Smallest workflow that can become dependent:** one protected branch's
   **mergeability decision**. The minimal unit is "can this PR merge into
   `main`?" For general legitimacy, that is `merge-guard`; for AI-heavy
   repos, the sharper unit is "can this agent-lane PR merge unless it is
   explicitly attributed `AGENT_AUTHORED`?"

4. **Most likely external dependent user:** a maintainer of a repo that
   already accepts AI-generated PRs, wants a native GitHub required check
   rather than a new platform, and cares about auditable merge legitimacy
   with near-zero integration friction.

5. **What was built next, and why:** not more core architecture, but the
   **smallest self-serve external adoption package** for maintainer trust —
   shipped in `ContinuityOS-` as
   `actions/continuity-merge-guard/examples/continuity-agent-attribution-gate.yml`,
   `ADOPT_AGENT_ATTRIBUTION_GATE.md`, and a discoverable README section. This
   directly answers the audit's own finding that the bottleneck is
   installability, not architecture. The next "build" after that is not
   another artifact — it is one real independent installation.

6. **What should not be built next:** more non-operative governance specs,
   more topology expansion, more proof-registry elaboration, or more Standing
   Authority refinement aimed only at ContinuityOS self-governance. Those
   surfaces are meaningful internally, but they do not presently create
   external dependency in the sandbox consumer.

7. **Strongest wedge:** **Merge Guard** as a required status check on a
   protected branch. It is the only wedge already proven to change merge
   eligibility in both directions in the consumer repo.

8. **Weakest wedge:** **Proof Registry** as an external wedge. It matters for
   internal audit lineage and standing-authority budget accounting, but the
   sandbox consumer does not materially depend on it today.

9. **Probability ContinuityOS achieves external dependency through maintainer
   trust:** roughly **45%**, as an inference rather than a directly observed
   metric. Materially above zero because the install surface is tiny, the
   enforcement semantics are already proven, the tags exist, GitHub branch
   protection makes the wedge natively load-bearing, and the adoption-cost
   barrier has now been substantially lowered with a self-serve package. Still
   below 50% because all current evidence is same-owner self-consumption and
   there is not yet an independently owned maintainer repo depending on it.

10. **Single highest-leverage action for the next 30 days:** get **one
    independently owned repo** to install a ContinuityOS-powered required
    check and prove the full counterfactual with real work. If the target
    repo has general merge-legitimacy pain, use `merge-guard`; if it has
    AI-PR pain, use `agent-attribution-gate`. Capture one passing PR, one
    blocked PR, and one maintainer retention statement. That single act would
    convert this system from self-attested cross-repo dependency into genuine
    external dependency formation.

## Constraints / non-goals

- No runtime behavior change, core-repo mutation, or canon expansion.
- No proof-registry mutation.
- No dependency claim until an outside-owner operator actually runs the test.

---

See also: `ContinuityOS-` core repo →
[`actions/continuity-merge-guard/ADOPT_AGENT_ATTRIBUTION_GATE.md`](https://github.com/joselunasrt8-creator/ContinuityOS-/blob/main/actions/continuity-merge-guard/ADOPT_AGENT_ATTRIBUTION_GATE.md)
for the self-serve adoption package this audit's decision #5 produced, and
[`FROM_TOOL_TO_DEPENDENCY.md`](./FROM_TOOL_TO_DEPENDENCY.md) for the underlying
dependency-probability model.
