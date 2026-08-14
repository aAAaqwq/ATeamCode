# Product brief: verified agent-team coding

Status: Draft — user and core promise confirmed; autonomy boundary pending.

No runtime, safety, recovery, replay, or undo capability described here has
been implemented or validated.

## First users

Developers and small teams with an existing repository who already use
terminal coding agents such as Claude Code or Codex.

## Core promise

Given a development goal, the prototype is intended to propose the smallest
sufficient agent team and verification plan. After user approval, it is
intended to return a reviewable code change with test evidence, independent
review, replayable events, and a safe undo path.

## Pending autonomy decision

The user has not yet selected the default approval model:

- approve the run plan once and allow policy-bounded tools to proceed; or
- also require individual approval for sensitive tool actions.

Until that decision is made, the design rule is fail closed. External actions,
credential use, commit, push, merge, deployment, and publication remain
forbidden to automatic execution.

## Primary journey

1. Open an existing repository and describe one development goal.
2. Inspect ATeam Code's interpretation, scope, non-goals, proposed team, and
   verification plan.
3. Approve or revise the plan before writes are allowed.
4. Follow role ownership, handoffs, tool actions, and evidence during the run.
5. Inspect tests, the independent review decision, remaining risks, and diff.
6. Accept, request revision, or undo only the run-owned change set.

## Prototype acceptance criteria

- No write occurs before the run scope and policy are approved.
- Every agent is selected for a stated evidence gap; simple tasks stay single
  agent.
- Passed, failed, skipped, and unavailable checks remain distinguishable.
- Independent review is a separate read-only session with evidence-bound
  findings.
- Every tool action maps to a run, turn, actor, policy decision, and artifact.
- A dirty starting worktree is preserved or the run fails closed before writes.
- Cancellation leaves no orphan processes or silently unknown side effects.
- Undo restores run-owned files without reverting pre-existing user changes.

## MVP non-goals

- Loading every AGI Super Team role and Skill into every run
- IDE, web console, remote multi-user operation, or marketplace
- Automatic commit, push, merge, PR, deployment, or credential use
- Byte-for-byte deterministic model replay
- Claims of zero defects or fully autonomous software delivery

## Product hypothesis to test

The smallest sufficient agent team plus explicit verification and independent
review improves first-pass task success and reduces human rework enough to
justify its added latency and cost over a capable single agent.

The prototype must measure this against a single-agent baseline. Multi-agent
activity alone is not product value.
