# WEDGE — Sandbox Dependency Formation Surface

## Current status

```text
Merge Guard required-check dependency ........ COMPLETE / RETAINED
Agent attribution gate required-check proof .. COMPLETE / SAME_OWNER
Independent outside-owner dependency proof ... OPEN
```

This sandbox is the controlled reference surface for ContinuityOS dependency formation. It proves that a repository outside the ContinuityOS source tree can consume ContinuityOS checks, make them load-bearing, and retain them because removing them would make the merge workflow worse.

The remaining frontier is not another same-owner demonstration. It is one unaffiliated maintainer who installs the `agent-attribution-gate`, makes it required on a protected branch, observes both blocked and passing agent-lane PRs, and answers that their merge path would be materially worse without it.

## What this repo proves

| Wedge | Status | Evidence |
|---|---|---|
| Merge Guard required check | COMPLETE / RETAINED | `EXTERNAL_DEPENDENCY_PROOF.md`, `NULL_ENFORCEMENT_PROOF.md`, `RETENTION_SIGNAL.md` |
| Agent attribution gate | COMPLETE / SAME_OWNER | `ATTRIBUTION_DEPENDENCY_PROOF.md`; PR #24 flipped from `UNKNOWN` blocked to `AGENT_AUTHORED` mergeable on the same branch |
| Break-glass confidence | AVAILABLE | `BREAK_GLASS.md` documents the governed override path |
| Outside-owner dependency | OPEN | Must be proven in a repo not owned by `joselunasrt8-creator` |

## Primary wedge now

The sharpest wedge is `agent-attribution-gate`, not generic Merge Guard.

Why:

1. The pain is concrete: AI/agent PRs can look operationally normal without declaring agent authorship.
2. The check is native GitHub CI: no app, no secret, no platform migration.
3. Human PRs pass neutrally.
4. Agent-lane PRs fail closed unless they carry an authoritative `AGENT_AUTHORED` signal.
5. The same-owner proof is complete, so the next useful signal is independent retention.

## Outside-owner success condition

```text
unaffiliated maintainer
  -> installs ContinuityOS agent-attribution-gate
  -> pins `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.3.0`
  -> makes `agent-attribution-gate` a required check on their protected branch
  -> one agent-lane PR is blocked for missing attribution
  -> the same PR or a follow-up passes after attribution is added
  -> maintainer says removal would make their merge path materially worse
```

That single retained outside install closes the open dependency proof.

## Execution handoff

Use the main-repo install and outreach package:

- `actions/continuity-merge-guard/ADOPT_AGENT_ATTRIBUTION_GATE.md`
- `actions/continuity-merge-guard/examples/continuity-agent-attribution-gate.yml`
- `docs/adoption/agent-attribution-gate-outreach.md`
- `docs/adoption/agent-attribution-gate-candidates.md`

Use this sandbox only as evidence. Do not add more architecture or same-owner demos unless a candidate asks for a specific clarification.

## Non-goals during cooldown

- new canon
- new ontology
- new governance layers
- more same-owner proof loops
- broad visualization or course material before outside-owner retention

## Classification

```text
DEPENDENCY_REFERENCE_SURFACE_ACTIVE
OUTSIDE_OWNER_PROOF_OPEN
```
