# ADR 0002: Ship CLI execution and a local visual Supervisor

Status: Proposed — visual direction and implementation remain unapproved.

## Context

Terminal interfaces are fast and scriptable, but multi-agent runs create more
simultaneous state than a linear transcript communicates well: task ownership,
dependencies, waiting approvals, file changes, checks, costs, review findings,
and recovery actions. A web-only or desktop-only product would add onboarding
and automation friction for the initial CLI-native users.

## Proposed decision

Ship two clients over one local control protocol:

- `ateam` is the default install, fastest start path, and automation surface.
- Supervisor is an optional but first-class local visual application for
  planning, supervision, decisions, review, history, and undo.

The initial implementation may serve a loopback web application. Packaging it
in a desktop shell is a later distribution choice, not an architectural
boundary. The control plane and its state must not live inside the browser or
desktop renderer.

The recommended shell is **Calm Decision Inbox**, with **Agent Workbench** as
the selected-task view. **Dense Mission Control** is deferred until measured
concurrency justifies it. A three-option visual ideation pass and user selection
are required before UI implementation.

## Consequences

### Benefits

- preserves terminal speed and scripting;
- gives long-running team work an understandable supervisory surface;
- makes approvals, evidence, review, and undo first-class rather than transcript
  conventions; and
- allows CLI, local web, and a future desktop shell to share one protocol.

### Costs and risks

- two clients increase compatibility and accessibility testing;
- event ordering, reconnect, and exactly-once approval resolution become core
  protocol requirements;
- a loopback UI still needs authentication, origin checks, CSRF protection,
  safe binding defaults, and output redaction; and
- feature drift between CLI and Supervisor could create hidden authority.

## Alternatives

### CLI only

Fastest to build, but leaves the central multi-agent comprehension problem
unsolved and makes independent evidence hard to inspect.

### Visual application only

More approachable to some users, but weakens scripting, remote terminal work,
and adoption by the initial CLI-native audience.

### IDE extension first

Excellent file context, but introduces host-specific lifecycle and marketplace
constraints before the control protocol is stable. It remains a later client.

### Graph-first agent canvas

Visually distinctive, but optimizes for showing activity rather than making
safe decisions. A task graph can exist as a secondary Plan view.

## Acceptance gate

This ADR can become Accepted only after:

- the user selects the dual-surface product decision and a visual direction;
- CLI and Supervisor pass shared command/event contract tests;
- reconnect and competing approval-client tests show one canonical outcome;
- core Supervisor workflows pass keyboard and screen-reader checks; and
- target-user usability tests outperform a transcript-only baseline on run
  comprehension without materially slowing simple task initiation.
