# Attribution gate — fail-closed demonstration

This file exists only to carry a deliberate **negative** test of the
`agent-attribution-gate`:

- head branch is an agent lane (`claude/**`)
- the commit carries **no** `Agent-Authored-By:` trailer
- the PR has **no** `agent-authored` label and **no** PR-body attribution block

Expected: the gate harvests no authoritative signal, the ContinuityOS action
classifies the PR `UNKNOWN`, and `agent-attribution-gate` **fails** (fail-closed).

This PR is an evidence artifact and is **not** intended to merge. Together with
the PASS case (sandbox PR #19), it forms the counterfactual in
`ATTRIBUTION_DEPENDENCY_PROOF.md`: the agent-lane merge outcome is determined by
the attribution classification, so removing it degrades the workflow.
