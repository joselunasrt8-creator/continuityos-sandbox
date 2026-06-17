# Attribution Dependency Proof - AGENT_AUTHORED as a load-bearing gate

## Question

> Can the ContinuityOS attribution classification become a *dependency* rather
> than merely a *capability* - i.e. does this repo's agent-lane workflow become
> materially worse if attribution disappears?

## Mechanism

`.github/workflows/continuity-agent-attribution-gate.yml` consumes the published
action `joselunasrt8-creator/ContinuityOS-/actions/continuity-merge-guard@v0.3.0`,
which EMITS an `attribution_classification`. The action stays non-blocking; this
consumer enforces:

```
agent lane (claude/**, codex/**, ...) PR
  -> harvest Agent-Authored-By trailers + read PR labels/body
  -> action @v0.3.0 -> attribution_classification
  -> gate: AGENT_AUTHORED ? success : failure (fail-closed)
```

Non-agent PRs pass neutrally (Phase-1: human PRs are never blocked).

## Counterfactual (load-bearing test)

```
WITH attribution (current):
  claude/** PR + Agent-Authored-By      -> AGENT_AUTHORED -> gate success -> mergeable
  claude/** PR, no authoritative signal -> UNKNOWN        -> gate failure -> blocked

WITHOUT attribution (remove ContinuityOS action):
  the gate has no attribution_classification to read
  -> cannot distinguish AGENT_AUTHORED from UNKNOWN
  -> the agent lane can no longer require truthful agent attribution
  -> enforcement degrades
```

Removal degrades the workflow. The dependency is load-bearing.

## Evidence

| Path | PR | head ref | classification | gate run | conclusion |
|------|----|----|------|------|------|
| PASS | (this PR) | `claude/continuityos-workflow-topology-1rcgfa` | AGENT_AUTHORED | _pending_ | _pending_ |
| FAIL | _pending_ | `claude/...` (no trailer) | UNKNOWN | _pending_ | _pending_ |

Evidence rows are filled in from the live `agent-attribution-gate` check runs.

## Required-check status

To make the gate formally load-bearing, `agent-attribution-gate` must be added as
a **required** status check on `main` (repository-admin action, the same class as
the original `merge-guard` required-check setup documented in
`LOAD_BEARING_READINESS.md`).

## Classification

```
ATTRIBUTION_GATE_IMPLEMENTED        <-- this PR (consumer enforces AGENT_AUTHORED)
ATTRIBUTION_PASS_DEMONSTRATED       -- this PR's own gate run (AGENT_AUTHORED -> success)
ATTRIBUTION_FAIL_DEMONSTRATED       -- pending no-trailer demo PR (UNKNOWN -> failure)
ATTRIBUTION_REQUIRED_CHECK_ACTIVE   -- pending repo-admin (add required check)
ATTRIBUTION_ENFORCEMENT_CONFIRMED   -- once required + both paths shown
```
