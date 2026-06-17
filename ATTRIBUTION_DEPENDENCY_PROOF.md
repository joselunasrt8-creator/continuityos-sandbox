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

## Evidence (live CI, both paths)

| Path | PR | head ref | classification | status | gate run | conclusion |
|------|----|----|------|------|------|------|
| PASS | [#19](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/19) | `claude/continuityos-workflow-topology-1rcgfa` | `AGENT_AUTHORED` | `identity_present` | [27659873583](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27659873583) | **success** |
| FAIL | [#20](https://github.com/joselunasrt8-creator/continuityos-sandbox/pull/20) | `claude/attribution-gate-fail-demo` | `UNKNOWN` | `identity_missing` | [27661985497](https://github.com/joselunasrt8-creator/continuityos-sandbox/actions/runs/27661985497) | **failure** |

Key observation: in the FAIL case the existing identity-only `merge-guard` check
still reported **success** - only `agent-attribution-gate` failed. The
attribution classification is therefore the *sole* discriminator of the
agent-lane merge outcome.

## Required-check status

To make the gate formally load-bearing, `agent-attribution-gate` must be added as
a **required** status check on `main` (repository-admin action, the same class as
the original `merge-guard` required-check setup documented in
`LOAD_BEARING_READINESS.md`). Once required, the FAIL PR (#20) reports `blocked` /
not mergeable, while the PASS PR (#19) is mergeable.

## Classification

```
ATTRIBUTION_GATE_IMPLEMENTED        DONE  (PR #19 merged: consumer enforces AGENT_AUTHORED)
ATTRIBUTION_PASS_DEMONSTRATED       DONE  (PR #19 run 27659873583: AGENT_AUTHORED -> success)
ATTRIBUTION_FAIL_DEMONSTRATED       DONE  (PR #20 run 27661985497: UNKNOWN -> failure)
ATTRIBUTION_REQUIRED_CHECK_ACTIVE   PENDING repo-admin (add agent-attribution-gate as required check)
ATTRIBUTION_ENFORCEMENT_CONFIRMED   PENDING required-check activation
```

With the gate implemented and both paths demonstrated in live CI, the capability
is proven load-bearing in mechanism; marking it a *required* check is the final
repo-admin step that converts it from demonstrated to enforced.
