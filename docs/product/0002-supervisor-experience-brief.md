# ATeam Code Supervisor experience brief

Status: Draft — interaction direction for validation, not a selected visual
design or implemented interface.

## Experience thesis

ATeam Code should make a long-running coding team feel understandable and
reversible. The user must be able to answer six questions without reading a
raw transcript:

1. What is the team trying to accomplish?
2. Why is more than one agent needed?
3. What is happening now, and what is blocked?
4. What authority is being requested?
5. What changed, and what evidence supports it?
6. Can I safely stop or undo this run?

The interface is a supervisor for engineering work, not an animated org chart
and not an IDE replacement.

## Target user and context

The first user is an individual developer or small software team with an
existing repository who is already comfortable with a terminal coding agent.
They use the CLI for speed but need a clearer surface when work becomes
parallel, long-running, expensive, risky, or review-heavy.

## Dual-surface contract

- CLI and Supervisor are peers over the same run protocol.
- Starting in the CLI must immediately create a run visible in Supervisor.
- Approving in either client must resolve one canonical approval exactly once.
- Reconnecting must preserve the run, selection, and unresolved decisions.
- Features may render differently, but neither client invents hidden state.
- Automation can use non-interactive CLI commands only inside predeclared
  policy; it cannot bypass approval requirements.

## Primary journey

### 1. Choose a repository and state the goal

Show repository health before promising execution: branch/base revision,
dirty state, detected stack, available checks, active worktrees, and policy
profile. Preserve the user's prompt when setup or validation fails.

### 2. Review the proposed run

The plan explains interpretation, scope, non-goals, proposed agents, why each
agent exists, task dependencies, budget, required authority, tests, review,
and undo strategy. A simple task should visibly stay single-agent.

### 3. Supervise execution

Use a chronological activity timeline as the primary truth. A compact task DAG
and agent list are secondary navigation. Collapse routine successful events;
expand failures, decisions, file changes, and evidence. Let the user steer or
cancel without hunting through transcript text.

### 4. Resolve decisions

An approval card states the proposed action in plain language, exact scope,
risk, initiating agent, reason, and persistence. Choices should be explicit:
deny, allow once, or allow a narrowly described rule when supported. Never use
a generic `Approve` button for materially different authority.

### 5. Review the result

Present goal coverage, scoped diff, tests with passed/failed/skipped/unavailable
states, independent findings, remaining risks, cost, and elapsed time. Link
every summary claim to evidence. Keep `Accept` separate from commit, push, PR,
merge, or deploy.

### 6. Revisit or undo

History supports search by project, outcome, agent, and risk. Replay shows the
canonical event timeline, not simulated model determinism. Undo previews the
run-owned reverse change and refuses when preimages no longer match.

## Information architecture

### Core v0 views

1. **Attention Inbox** — `Needs me`, failed, ready to review, working, and
   recently completed runs ordered by required user attention.
2. **New Run and Plan** — repository health, goal, scope, team rationale, task
   graph, budget, permissions,
   checks, and approve/revise controls.
3. **Task Workspace** — status summary, task DAG, agent roster, event timeline,
   context/cost, and steer/cancel controls.
4. **Approval Center** — unresolved first, with exact authority and risk;
   resolved decisions remain auditable.
5. **Review** — outcome coverage, diff, checks, Governor findings, risks,
   accept/revise/undo.
6. **History** — run receipts, replay, artifacts, and undo state.
7. **Project and Settings** — repository/worktree health, runtimes, models,
   sandbox profiles, Skills/roles, retention, and diagnostics. It is not part
   of the main task flow.

### Live Run layout hierarchy

- Top bar: goal, state, elapsed time, cost, stop control.
- Left rail: tasks and agents, grouped by ownership and status.
- Main pane: readable event timeline with filters for decisions, changes,
  checks, and failures.
- Right inspector: selected task/event details, evidence, affected files, and
  requested authority.
- Bottom composer: steer or answer a question; it is not the only control.

## Three visual directions to explore

These are briefs for a later three-option visual ideation pass. None is
selected yet.

### A. Dense Mission Control

A high-density fleet view: graphite canvas, compact tables, tabular numbers,
status filters, keyboard-first peek, and optional task topology. It scans well
when many repositories and runs are active, but can make activity look like
value and overwhelm the first individual users. Defer it to an optional dense
mode after real concurrency demand is measured.

### B. Agent Workbench

A familiar code-review workspace: project tree, central diff/timeline tabs,
terminal drawer, and right-side evidence inspector. Lower learning cost for
IDE users and strong for file-level work, but risks becoming an inferior IDE
and hiding the team/run model behind panels.

### C. Calm Decision Inbox — recommended default

A quiet attention surface that shows `Needs me`, failures, and ready-to-review
work first while routine execution stays collapsed. Approval cards use a
problem, impact, evidence, and choices structure. Generous spacing, restrained
color, explicit text status, and minimal motion reduce supervision cost. Raw
events remain one step away so calm does not become concealment.

The intended composition is C for the default shell, B for the selected task,
and A only as a later dense mode. A task graph remains a secondary Plan view,
not the main product metaphor.

## Current source map

Official product surfaces show convergence toward persistent tasks, explicit
attention states, and visual review, but they do not establish ATeam Code user
demand:

- [Codex App](https://openai.com/index/introducing-the-codex-app/) and
  [Git worktrees](https://learn.chatgpt.com/docs/environments/git-worktrees)
- [Claude Code agent view](https://code.claude.com/docs/en/agent-view) and
  [Desktop](https://code.claude.com/docs/en/desktop)
- [OpenClaw Control UI](https://docs.openclaw.ai/web/control-ui) and
  [exec approvals](https://docs.openclaw.ai/tools/exec-approvals)
- [Hermes TUI](https://hermes-agent.nousresearch.com/docs/user-guide/tui),
  [web dashboard](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-dashboard),
  and [delegation](https://hermes-agent.nousresearch.com/docs/user-guide/features/delegation/)
- [DeepSeek Harness repository](https://github.com/deepseek-ai/deepseek-harness/tree/47f943859bef60e4160492346772ded9b24f765a)

No interviews, support corpus, usage analytics, or prototype usability tests
were available. Cross-product feature convergence is only an opportunity
signal, not a frequency estimate for user pain.

## Interaction rules

- Progressive disclosure: summary first; raw command, reasoning excerpt, and
  logs on demand.
- Quiet success, loud uncertainty: failures, skipped checks, unknown side
  effects, and waiting approvals must dominate routine success noise.
- Status uses text and icon in addition to color.
- Destructive or external actions show target and reversibility before the
  decision.
- The primary keyboard path covers project selection, plan approval, timeline
  navigation, approval decisions, review, and cancel.
- Reduced-motion mode removes live graph movement and nonessential streaming
  animation.
- Screen-reader announcements summarize state changes without reading every
  token or log line.

## Usability acceptance criteria

- A new target user can start a safe run and explain its scope without docs.
- In a five-agent simulated run, a user can identify the active task, blocked
  task, waiting approval, latest changed file, and failing check within 10
  seconds each.
- The user can distinguish plan approval, one-action approval, accept changes,
  commit, and publish; none are visually or semantically conflated.
- Passed, failed, skipped, unavailable, cancelled, and unknown are distinct in
  text, not color alone.
- All core workflows are keyboard operable with visible focus and logical
  order; dialogs trap and restore focus correctly.
- Dense views remain usable at 200% zoom and at a 1280-pixel desktop width.
- Reconnect, stale state, partial event loss, backend crash, and cancelled run
  each have an explicit recovery message and next action.
- A visual comparison never substitutes for functional, accessibility, or
  runtime verification.

## Product success measures

- activation: first repository connected and first approved plan completed;
- comprehension: time and accuracy for the five live-run questions above;
- safety: unapproved side effects and approval-scope violations;
- usefulness: accepted changes and human rework versus a single-agent baseline;
- efficiency: time, tokens, and cost per accepted change;
- trust: cancellation success, verified undo success, and evidence-link usage;
- restraint: percentage of routine tasks correctly kept single-agent.

No claim that “everyone will like it” is testable. The nearer goal is that the
defined first users prefer ATeam Code for work where ordinary coding-agent
transcripts become hard to supervise.
