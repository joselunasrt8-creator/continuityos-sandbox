# From Tool to Dependency: The Formation of Load-Bearing Infrastructure

## Classification

```text
DEPENDENCY_FORMATION_RESEARCH
```

## Intent

Identify the first credible **dependency event** for ContinuityOS — the
specific moment a repository owner decides Merge Guard *must* be present for a
pull request to remain mergeable — and define a reusable model for why
external control-plane infrastructure transitions from "useful" to "required."

The thesis: the first credible dependency event is **ContinuityOS Merge Guard
as a required status check on protected GitHub branches for agent-authored or
high-autonomy pull requests.** This is not another paper on legitimacy theory,
topology, or orchestration. The remaining bottleneck is dependency conversion,
not architecture.

## Scope

In scope:

- The dependency-probability model and its application to ContinuityOS install
  surfaces.
- Benchmarking against comparable infrastructure that became load-bearing.
- Assessment of current evidence in this repo (same-owner) vs. the
  outside-owner frontier.
- Falsification conditions and the next experiment.

Out of scope:

- ContinuityOS core runtime changes or new governance semantics.
- New canon, authority model, or proof-registry mutation.
- Any dependency claim before an outside-owner operator runs the test.

## Dependency-probability model

Dependency probability is modeled as a function of five forces. The model is an
inference from the comparison cases below, not a vendor benchmark — but it fits
the observed pattern: infrastructure becomes required when it sits on an
existing gate, solves a recurring failure mode, and makes removal obviously
harmful.

| Force | Question it answers |
|---|---|
| **Gate adjacency** | Does the tool sit on a decision the platform already enforces (merge, deploy, admission)? |
| **Installation friction** | How few steps from "interested" to "running in the real flow"? |
| **Evidence visibility** | Is the result visible exactly where the decision is made (the merge box, the PR check run)? |
| **Failure cost without the tool** | What recurring bad outcome happens when the tool is absent? |
| **Removal pain after adoption** | Once a team organizes its workflow around it, how much worse is the workflow if removed? |

### Applied to ContinuityOS surfaces (ranked)

| Surface | Gate adjacency | Install friction | Evidence visibility | Dependency rank |
|---|---|---|---|---|
| **Merge Guard on protected branches** | High — sits directly on GitHub's required-status-check merge gate | Low — one workflow file + one branch-protection toggle | High — result appears in the PR merge box / check run | **1 — strongest near-term dependency** |
| GitHub issue-comment adapter | Medium — same host and buyer, but not a merge gate | Low | Medium | 2 — best second portability proof |
| Governed filesystem demo (`npm run demo`) | Low — terminal-local, no shared gate | Low | High for explanation, low for dependency | 3 — strongest for explanation, weakest for dependency |
| Broader distributed-legitimacy stack | Low — abstract, documentation-heavy | High | Low | 4 — strategically interesting, not the first thing made required |

The governed filesystem demo is the best *explanation* surface and the worst
*dependency* surface. Merge Guard is the inverse: it has the highest gate
adjacency and the lowest path to becoming load-bearing.

## Comparable load-bearing infrastructure

Each case became non-optional through one or more of the five forces. None
became required by selling a theory first; all attached to an existing problem,
then deepened.

| System | How it became load-bearing |
|---|---|
| **GitHub branch protection / CODEOWNERS** | Owns the merge gate itself — required status checks and required reviews block merge. ContinuityOS does not need to invent this gate, only occupy it. |
| **Snyk** | Scans PRs before merge, adds CI/CD guardrails, opens remediation PRs — lands in the exact PR decision point. |
| **Semgrep** | PR checks across GitHub/GitLab/Bitbucket/Azure, remediation directly in PR comments — sticky because it appears where code is allowed to move forward. |
| **Mergify** | Merge queue tests every PR against latest main before merge; value framed around broken main, throughput, CI cost — removal pain. |
| **Graphite** | Stacked PRs, AI review, stack-aware merge queue — workflow reorganizes around it; removal degrades the flow. |
| **OPA / Gatekeeper** | Policy decision offloaded from services; Gatekeeper as an admission webhook rejects non-compliant changes before bad state lands — proof tied to a consequential gate. |
| **Sigstore** | Identity-bound signing + append-only transparency log — supply-chain identity that is valuable because it is verified at a gate. |
| **Tailscale** | "Install in minutes," then org policy at scale — fast attach first, policy depth after adoption. |
| **Auth0** | Enterprise auth for apps, agents, MCP servers "in minutes" — easy attachment, then depth. |
| **LaunchDarkly** | Runtime control for AI-built code and agents: progressive rollout, approvals, RBAC — control plane that becomes the way changes ship. |

ContinuityOS most closely resembles the **policy-and-proof cohort**
(OPA/Gatekeeper/Sigstore): it is valuable only when its proof is tied to an
already-consequential gate. Its install motion should resemble the
**control-plane cohort** (Tailscale/Auth0/LaunchDarkly): fast attach first,
policy depth second.

## Timing

The surrounding behavior is already real, which makes "one more required check
before merge" legible:

- Large-scale studies report hundreds of thousands of agent-authored pull
  requests across tens of thousands of repositories, and coding-agent adoption
  across a meaningful fraction of active GitHub projects.
- Developer surveys show high AI-tool adoption alongside low trust in AI output
  accuracy and high concern about agent accuracy and security.

That gap — high agent autonomy, low trust — is exactly the environment where a
legitimacy gate at the merge boundary becomes adoptable. Merge Guard fits the
*existing* GitHub control loop instead of asking the market to adopt a new one.

## Positioning

ContinuityOS is a **governance layer**, not a competitor to agent frameworks
(Claude Code, Codex, OpenAI Agents SDK, LangGraph). Those systems help agents
*act*. ContinuityOS matters only if it decides whether those actions are
legitimate enough to become **mergeable** or **executable**. The wedge is to
occupy the merge gate beneath whatever agent or human authored the PR, not to
replace the authoring tool.

## Current-state evidence (this repo)

This sandbox is a genuine external-consumer rehearsal and has already crossed
into load-bearing behavior on its own protected branch:

- [`DEPENDENCY_ASSESSMENT.md`](../DEPENDENCY_ASSESSMENT.md) — dependency
  boundary, fail-closed `BLOCKED_*` model, classification path.
- [`LOAD_BEARING_READINESS.md`](../LOAD_BEARING_READINESS.md) — canonical
  install path and readiness criteria.
- [`NULL_ENFORCEMENT_PROOF.md`](../NULL_ENFORCEMENT_PROOF.md) — real
  `NULL → failure → blocked` evidence.
- [`EXTERNAL_DEPENDENCY_PROOF.md`](../EXTERNAL_DEPENDENCY_PROOF.md) — external
  consumption proof.
- [`EXPERIMENT_MERGE_GUARD_DEPENDENCY_TEST.md`](../EXPERIMENT_MERGE_GUARD_DEPENDENCY_TEST.md)
  — the dependency-test experiment scaffold.

**Critical caveat:** all current proof is **same-owner**. Both
`ContinuityOS-` and `continuityos-sandbox` are under the same owner umbrella,
so this is *controlled* dependency proof, not the final frontier. The decisive
milestone is an **outside-owner** repository that keeps the check enabled
because removing it would make its workflow worse.

## Falsification conditions

The thesis must be killable. It fails if:

1. Outside pilot teams say GitHub protected branches, CODEOWNERS, merge queues,
   Snyk, and Semgrep already cover the real pain.
2. No one will require Merge Guard until v2 features exist (diff binding,
   automatic agent attribution, review binding, merge-commit binding).
3. The public install story stays inconsistent enough that outsiders cannot
   tell which path is canonical (stale `mindshift-demo` snippets vs. pinned
   `@v0.1.0` vs. terminal demo vs. sandbox docs).
4. No retained pilot ever reports that its workflow is materially worse without
   the check.

These are the experiments that determine whether ContinuityOS becomes
infrastructure or remains a compelling internal research artifact.

## Acceptance criteria

- [ ] Dependency-probability model documented and applied to ContinuityOS
      surfaces.
- [ ] Comparable infrastructure benchmarked against the five forces.
- [ ] Same-owner vs. outside-owner distinction made explicit.
- [ ] Falsification conditions enumerated.
- [ ] Outside-owner experiment identified as the next step.

## Open questions / next step

The central open question: **what specific event causes an outside-owner
repository to make `merge-guard` a required status check — and keep it?**

Recommended next experiment: extend the pattern in
[`EXPERIMENT_MERGE_GUARD_DEPENDENCY_TEST.md`](../EXPERIMENT_MERGE_GUARD_DEPENDENCY_TEST.md)
to an **unaffiliated** repository, treating the work as a packaging and
dependency-conversion exercise (canonical install path, pinned version,
evidence visibility) rather than an architecture exercise. The retention
signal to capture: an independent operator stating their merge path is worse
without the check.

## Constraints / non-goals

- No runtime behavior change, core-repo mutation, or canon expansion.
- No proof-registry mutation.
- No dependency claim until an outside-owner operator actually runs the test.

---

See also: `ContinuityOS-` core repo →
[`docs/strategy/DEPENDENCY_FORMATION.md`](https://github.com/joselunasrt8-creator/ContinuityOS-/blob/main/docs/strategy/DEPENDENCY_FORMATION.md)
for the compressed operational state, frontier, and acceptance criteria.
