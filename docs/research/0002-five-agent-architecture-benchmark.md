# Five-agent architecture benchmark

Status: Research snapshot as of 2026-08-14.

## Question

What do Codex, Claude Code, OpenClaw, Hermes Agent, and DeepSeek Harness
actually implement, where do their architectures converge, and which patterns
should ATeam Code reuse without inheriting their product or safety boundaries?

This note uses official source repositories and official documentation. Facts,
design judgments, and unknowns are separated. Product popularity is outside
the evidence boundary; stars, launch copy, and anecdotal enthusiasm are not
used as proof of usability.

## Inspected anchors

| Project | Revision or release anchor | Public reuse boundary |
|---|---|---|
| Codex | [`8630bb3caecaff6abc6add450a88035d9f6d3f8c`](https://github.com/openai/codex/commit/8630bb3caecaff6abc6add450a88035d9f6d3f8c); latest release observed `rust-v0.147.0` | Apache-2.0 |
| Claude Code | [`v2.1.232` / `1f6015b5d578adf79c8527443328a216d6b6a3f1`](https://github.com/anthropics/claude-code/tree/v2.1.232) | Public repository is all-rights-reserved; core runtime source is not present |
| OpenClaw | [`c2269f7a6c4115972496e1a5ae1a79ad9af457ae`](https://github.com/openclaw/openclaw/tree/c2269f7a6c4115972496e1a5ae1a79ad9af457ae) | MIT; inspected `main` is ahead of the observed stable release, so main behavior is not labeled released behavior |
| Hermes Agent | [`56a41715dc3b8bf6f50a740ff9416c4036ef4259`](https://github.com/NousResearch/hermes-agent/tree/56a41715dc3b8bf6f50a740ff9416c4036ef4259); observed release `v2026.8.13` | MIT |
| DeepSeek Harness | [`47f943859bef60e4160492346772ded9b24f765a`](https://github.com/deepseek-ai/deepseek-harness/commit/47f943859bef60e4160492346772ded9b24f765a), package `0.1.0-rc.5` | MIT; explicitly Developer Preview with expected breaking changes |

`Hermes` means the official Nous Research
[`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent),
not a same-name model or unrelated project.

## Executive finding

All five converge on an iterative model/tool loop, durable session identity,
scoped context, structured tools, approval or policy controls, compaction, and
some form of delegated work. They do not converge on one safe universal API.
Their protocols preserve different semantics for approval, sandboxing,
branching, cancellation, tools, reasoning blocks, and child agents.

The useful ATeam Code move is therefore not to fork one product or normalize
everything to chat completions. It should own a small canonical control plane,
event ledger, policy boundary, task DAG, ChangeSet, and review receipt, then use
capability-declared runtime drivers. Codex App Server is the strongest first
prototype candidate because it exposes a typed rich-client protocol and an
auditable open core. DeepSeek Harness is the best early architectural
counterexample because its replaceable plugin seams test whether the control
plane is truly runtime-neutral. OpenClaw contributes the clearest single-
gateway/multi-client pattern; Hermes contributes practical multi-surface,
provider, sandbox-backend, and delegation patterns; Claude Code contributes
product workflow and extension contracts, not reusable core source.

No candidate has passed an ATeam Code conformance, adversarial safety,
recovery, or undo test.

## Architecture cards

### Codex

**Facts.** The Rust core exposes an iterative turn loop, session state and
context management, explicit sandbox and approval policies, MCP/tools, Skills,
hooks, multi-agent primitives, telemetry, and rollout traces. App Server makes
`Thread -> Turn -> Item` a bidirectional JSON-RPC protocol with start, resume,
fork, interrupt, streaming item events, diffs, approvals, review, and
compaction. Its generated schemas are version-specific. Sources:
[turn loop](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/core/src/session/turn.rs#L145-L159),
[App Server](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/app-server/README.md),
[sandbox and approvals](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/protocol/src/protocol.rs#L914-L1049),
and [rollout traces](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/rollout-trace/README.md).

**Judgment.** It is the best first rich runtime adapter, not ATeam Code's
domain model. Experimental fields and transports require version pinning,
capability negotiation, and contract tests.

**Unknown.** The repository does not establish the server-side design of Codex
Web or its cloud scheduler and isolation.

### Claude Code

**Facts.** The public repository at the pinned tag contains official plugins,
examples, configuration, and changelog material but not the core runtime. The
official docs expose the loop behavior, local sessions and resume/fork,
compaction, checkpoint limitations, tools, permission and Bash sandbox
interaction, hooks, Skills, MCP, subagents, experimental agent teams,
telemetry, and CLI/Desktop/IDE/Web surfaces. Agent teams use a lead, separate
teammate sessions, shared task list, and mailbox; each teammate has its own
context. Sources: [license](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/LICENSE.md),
[agent loop](https://code.claude.com/docs/en/agent-sdk/agent-loop),
[permissions](https://code.claude.com/docs/en/permissions),
[subagents](https://code.claude.com/docs/en/sub-agents),
[agent teams](https://code.claude.com/docs/en/agent-teams), and
[public plugins](https://github.com/anthropics/claude-code/tree/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins).

**Judgment.** Claude Code is a product and workflow reference. Its visible
prompt-programmed workflows, progressive Skill disclosure, hook lifecycle,
permission UX, and agent-team coordination are valuable patterns. Its private
core and license make it a poor basis for ATeam Code's own runtime.

**Unknown.** Core queues, locks, internal RPC, storage schema, and UI/runtime
boundary are not established by the public repository and must not be inferred
from plugins or packaged artifacts.

### OpenClaw

**Facts.** OpenClaw is a TypeScript multi-channel gateway with a built-in agent
runtime. The Gateway is the authority for sessions, authentication, channels,
and state; clients communicate through its request/response/event WebSocket
protocol. Its runtime has a serialized per-session loop, persistent sessions,
compaction, tools and policies, plugin hooks, multi-agent routing, subagents,
model/provider abstraction, and a Web Control UI. Sandbox placement, visible
tool policy, and elevated host execution are separate controls. Workspace
selection alone is not isolation. Sources:
[runtime architecture](https://github.com/openclaw/openclaw/blob/c2269f7a6c4115972496e1a5ae1a79ad9af457ae/docs/agent-runtime-architecture.md),
[agent loop](https://docs.openclaw.ai/agent-loop),
[Gateway protocol](https://docs.openclaw.ai/gateway/protocol),
[sandbox versus policy](https://docs.openclaw.ai/gateway/sandbox-vs-tool-policy-vs-elevated),
[subagents](https://docs.openclaw.ai/tools/subagents), and
[Control UI](https://docs.openclaw.ai/web/control-ui).

**Judgment.** Its strongest transferable pattern is one authoritative local
control plane with multiple clients and execution nodes. Its broad channel,
personal-agent, and plugin surface is larger than ATeam Code v0 needs.

**Unknown.** External tool effects are not deterministic replay, and internal
database or plugin APIs should not be assumed stable across fast-moving main
revisions.

### Hermes Agent

**Facts.** Hermes Agent is a Python harness exposed through CLI/TUI, Gateway,
ACP, API server, library, Desktop, and dashboard surfaces. `AIAgent` performs
the iterative conversation/tool loop. It stores sessions and tool history in
SQLite/WAL/FTS5, supports provider adapters and fallback, context compression,
60+ built-in tools, MCP, progressively loaded Skills, multiple terminal
backends, approval modes, profiles, and delegated child agents. The default
local terminal backend does not isolate the host. Agent-created Skills and
memory form a persistent learning loop, but writable persistent instructions
also create a supply-chain and prompt-injection boundary. Sources:
[architecture](https://hermes-agent.nousresearch.com/docs/developer-guide/architecture),
[agent loop](https://hermes-agent.nousresearch.com/docs/developer-guide/agent-loop/),
[sessions](https://hermes-agent.nousresearch.com/docs/user-guide/sessions/),
[security](https://hermes-agent.nousresearch.com/docs/user-guide/security/),
[configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration),
[delegation](https://hermes-agent.nousresearch.com/docs/user-guide/features/delegation/),
and [Skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills/).

**Judgment.** Hermes is a useful experimental adapter and product reference for
provider choice, sandbox backends, delegation visibility, and shared TUI/Web
state. LLM-based smart approval must not become final policy authority, and
agent-authored Skills must be staged, diffed, and approved.

**Unknown.** Public contracts do not establish exactly-once tool effects or
durable active-child recovery across every crash and transport failure.

### DeepSeek Harness

**Facts.** DeepSeek Harness treats the model adapter, tools, session log,
approval, sandbox, compaction, subagents, and agent loop as Cordis plugins.
Profiles, bundles, and patches compose those capabilities. A turn contains
zero or more steps; typed `SessionEvent` entries form an append-only source of
model-visible history. Tool execution uses ordered guards, wrappers, post
processing, and immutable results. Missing or failed approval answers fail
closed. Its sandbox vocabulary covers filesystem effects, not all process or
network authority. Sources:
[architecture](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md),
[agent lifecycle](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/agent-lifecycle.md),
[session subsystem](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/subsystems/session.md),
[tool pipeline](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/tool-execution-pipeline.md),
[approval](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/subsystems/approval.md), and
[subagents](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/subsystems/subagent.md).

**Judgment.** It is the clearest microkernel experiment and a strong test of
ATeam Code runtime neutrality. Its developer-preview status and unvalidated
security behavior make a direct fork or default dependency premature.

**Unknown.** Stability, upgrade compatibility, real-world recovery, redaction,
and cross-platform enforcement require pinned conformance tests.

## Comparison matrix

| Dimension | Codex | Claude Code | OpenClaw | Hermes Agent | DeepSeek Harness |
|---|---|---|---|---|---|
| Core source | Open Rust core | Not in public repo | Open TypeScript | Open Python | Open TypeScript |
| Primary state model | Thread/Turn/Item plus rollout history | Documented sessions/JSONL behavior | Gateway-owned sessions and transcript state | SQLite sessions/messages/tools | Append-only typed SessionEvent log |
| Rich client boundary | App Server JSON-RPC | Product/SDK behavior contracts | Gateway WebSocket req/res/event | TUI gateway, ACP, API/dashboard | Web/headless bundles and remote API seams |
| Extension model | AGENTS, Skills, MCP, hooks, plugins | CLAUDE.md, Skills, hooks, subagents, plugins, MCP | Plugins, hooks, Skills, MCP, providers, harnesses | Tools/toolsets, Skills, plugins, MCP, providers | Nearly every capability is a plugin |
| Delegation | Multi-agent child threads | Subagents and experimental Teams | Routed agents and subagent sessions | Profiles and delegated children | Replaceable subagent providers |
| Safety shape | Sandbox and approvals explicit | Permission rules plus Bash sandbox | Sandbox, tool policy, elevated separate | Backend sandbox plus guard/approval modes | Replaceable guard, approval, filesystem sandbox |
| Visual lesson | Typed events can power App/IDE review and approvals | Mature cross-surface task/review workflows | One gateway can power rich supervision clients | TUI/Web parity and live delegation visibility | UI is replaceable composition, not the loop |
| Principal caution | Experimental protocol surface | Private core and restrictive reuse boundary | Huge general-agent scope and trusted in-process plugins | Default local execution and writable learning surfaces | Developer preview and incomplete authority vocabulary |

## What is genuinely similar

The shared skeleton is:

```text
instructions + scoped context + tool schemas
                    |
                    v
              model sampling
                    |
          text or structured tool calls
                    |
       policy -> approval -> execution
                    |
        durable observations and artifacts
                    |
          repeat, compact, or complete
```

The products also converge on progressive context loading, session resume,
child contexts, cancellation, model/provider abstraction, and multiple user
surfaces. These are table stakes for long-horizon agents, not a differentiator
by themselves.

## Where ATeam Code should be different

1. **Team value is measured.** Multi-agent routing must justify each child and
   compare against a capable single-agent baseline.
2. **The run is the product.** Goal, scope, DAG, approvals, evidence, change
   ownership, Governor review, and undo are canonical objects, not transcript
   conventions.
3. **Policy outranks runtimes.** Drivers declare capabilities; missing required
   interception or cancellation fails closed.
4. **Evidence is inspectable.** Summaries link to typed events, artifacts,
   checks, and hashes. Transcript replay is never called side-effect replay.
5. **Skills are supply-chain inputs.** AGI Super Team roles and Skills are
   versioned, progressively loaded, and bounded by the RunSpec; writable Skills
   require review.
6. **The UI optimizes attention.** It leads with decisions, failures, review,
   and recovery, not token streams or animated agents.

## Patterns to reuse and avoid

### Reuse

- Codex's typed rich-client protocol, explicit approval flow, detached review,
  and version-generated schemas.
- Claude Code's prompt-programmed workflows, progressive Skills, hook events,
  subagent contexts, and team task/mailbox concepts.
- OpenClaw's single authoritative gateway, multi-client projection, writer
  fencing, and separation of sandbox, policy, and elevation.
- Hermes's provider profiles, shared terminal/visual state, delegation
  observability, and pluggable execution backends.
- DeepSeek Harness's append-only model-visible history, capability seams,
  monotonic guards, reversible plugin effects, and replaceable persistence.

### Avoid

- treating a workspace or working directory as a sandbox;
- using an LLM risk classifier as the final authorization decision;
- loading in-process plugins with control-plane secrets by default;
- permitting agents to silently overwrite persistent Skills or memory;
- compressing all providers and runtimes into a lowest-common-denominator chat
  API;
- replaying unknown external mutations after a crash;
- making a terminal emulator the visual product; or
- forking a fast-moving harness before the product hypothesis is validated.

## Required proof before backend selection

Pin release revisions and run the same suite through every candidate adapter:

- approval timeout, deny, duplicated response, and competing clients;
- unapproved file, network, credential, git publication, and MCP mutations;
- repeated tool delivery and unknown external result;
- cancel cascade, orphan child, stale writer, runtime crash, and reconnect;
- compaction preserving identifiers, constraints, and unresolved approvals;
- provider fallback preserving tool and reasoning semantics;
- dirty-worktree preservation and run-owned undo; and
- structured event, artifact, cost, and review completeness.

Until this proof exists, architecture visibility is not runtime safety.

