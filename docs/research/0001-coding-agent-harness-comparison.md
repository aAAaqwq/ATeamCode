# Coding-agent harness comparison

Status: Research snapshot as of 2026-08-14.

## Inspected revisions

- Claude Code public repository: `1f6015b5d578adf79c8527443328a216d6b6a3f1`
  (`v2.1.232`)
- OpenAI Codex: `8630bb3caecaff6abc6add450a88035d9f6d3f8c`
- DeepSeek Harness: `47f943859bef60e4160492346772ded9b24f765a`
- AGI Super Team: `f1ae089c830aad6f1096f84b90108a9872d175d9`

## Evidence boundary

The inspected public Claude Code repository does not contain the core runtime.
Its license is all-rights-reserved and the visible tree primarily contains
plugins, examples, policy configuration, changelog material, and operational
scripts. It is evidence for the visible repository contents and documented
extension contracts. It does not, by itself, prove product behavior or reveal
the private implementation.

Sources:

- [Claude Code license](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/LICENSE.md)
- [Plugin structure](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/README.md)
- [Settings and sandbox boundary](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/examples/settings/README.md)
- [Feature-development swarm workflow](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/feature-dev/commands/feature-dev.md)
- [Progressive Skill loading](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/plugin-dev/skills/skill-development/SKILL.md)

## Architecture synthesis to validate

The following is the author's synthesis of a useful coding-agent loop, not a
claim that the three products implement an identical internal architecture:

```text
scoped instructions + context + tool schemas
                     |
                     v
               model request
                     |
              tool calls / text
                     |
         policy -> approval -> execution
                     |
              durable observations
                     |
              repeat or complete
```

The inspected interfaces and documentation show recurring design themes. Their
semantics and strength differ by product and require conformance testing:

1. A durable session identity with resumable turns.
2. Structured tool schemas rather than unconstrained shell text alone.
3. Permissions and execution isolation as separate concerns.
4. Scoped project instructions and progressively loaded reusable Skills.
5. Extension seams for MCP, hooks, plugins, or runtime adapters.
6. Child agents with separate context and bounded authority.
7. Context compaction or summarization that reduces the active model surface.

Evidence routes for these themes follow. `Not established` means that the
inspected, version-pinned public source set did not establish the claim; it
does not mean the product lacks the capability.

| Theme | Claude evidence | Codex evidence | DeepSeek evidence |
|---|---|---|---|
| Iterative loop and structured tools | [mutable official agent-loop docs, accessed 2026-08-14](https://code.claude.com/docs/en/agent-sdk/agent-loop) | [core/CLI overview](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/README.md) | [turn flow](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#turn-flow) |
| Durable session identity and resumable turns | Not established by the inspected commit | [App Server thread/turn protocol](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/app-server/README.md) | [Session log](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#session-log) |
| Permission and isolation separation | [settings boundary](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/examples/settings/README.md) | [App Server approval configuration](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/app-server/README.md#approvals) | [capability seams](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/capability-seams.md) |
| Scoped instructions and progressive Skills | [Skill loading guidance](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/plugin-dev/skills/skill-development/SKILL.md#progressive-disclosure-design-principle) | Not established by the inspected fixed sources used here | Not established by the inspected fixed sources used here |
| Extension seams | [plugin layout](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/README.md#plugin-structure) | [MCP client/server boundary](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/README.md#model-context-protocol-support) | [profiles, bundles, and capability seams](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#profiles-and-bundles) |
| Child agents with separate context and authority | [feature workflow declares parallel specialized agents](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/feature-dev/commands/feature-dev.md#phase-2-codebase-exploration); runtime isolation not established | Not established by the inspected fixed sources used here | [subagent provider documentation](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/subagent/subagent/README.md) |
| Context compaction outside the active model surface | Not established by the inspected commit | [App Server compact surface](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/app-server/README.md) | [compaction package](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/compaction/compaction/README.md) |

Online Claude documentation is mutable and is identified by access date rather
than presented as commit-pinned source evidence.

## Where they differ

| Dimension | Claude Code | Codex | DeepSeek Harness |
|---|---|---|---|
| Reusable implementation | Core runtime is not present in the inspected public repository; extension examples and Agent SDK surface are visible | Apache-2.0 Rust core, CLI, SDK, and App Server | MIT plugin-oriented runtime |
| Control protocol | Product behavior and SDK callbacks | Explicit thread/turn/item App Server protocol | Turn/step plus durable Session events and live agent events |
| Extension model | Instructions, Skills, hooks, agents, plugins, MCP | AGENTS, Skills, MCP, config, SDK, App Server | Cordis profile/bundle/patch; nearly every capability is a plugin |
| State | Resume/fork and transcript behavior exposed by product/docs | Threads, rollouts, fork, resume, compact, interrupt | Append-only Session is the source of model-visible history |
| Tool policy | Permission rules, hooks, Bash sandbox, managed settings | Sandbox and approval policy are explicit and independently configured | Guard, approval, sandbox, and tool execution are replaceable seams |
| Main reuse value | Product UX and extension-contract reference | Open execution backend and protocol reference | Experimental model-neutral microkernel and adapter target |

## Claude Code visible-source lessons

### Workflows are prompt programs

The `feature-dev` plugin expresses discovery, parallel exploration,
clarification, competing architecture proposals, implementation, and review as
a phase contract. The runtime stays generic while commands and agents carry
the workflow. ATeam Code should likewise compile AGI Super Team manifests into
run contracts instead of hard-coding every Team inside the loop.

### Extensions combine static and event-driven behavior

The public plugin layout combines commands, agents, Skills, hooks, and MCP
configuration. At the inspected revision, the `security-guidance` configuration
declares checks for `SessionStart`, `UserPromptSubmit`, `PostToolUse`, and
`Stop`, including asynchronous re-wake fields. This is configuration evidence,
not an observed runtime result:

- [hook configuration](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/plugins/security-guidance/hooks/hooks.json)

ATeam Code needs an event ledger and policy pipeline that can support the same
class of extension without allowing a late plugin to weaken a hard guard.

### Progressive disclosure is essential

Claude's visible Skill guidance keeps metadata always available, loads the
main Skill body only when triggered, and reads references or executes scripts
only as needed. ATeam Code cannot inject hundreds of AGI Super Team Skills into
every prompt; it needs a catalog, eligibility filter, and budgeted loader.

### Sandbox claims must be narrow

Claude's public settings example explicitly says its `sandbox` setting applies
to Bash, not automatically to Read, Write, Web, MCP, hooks, or internal
commands. ATeam Code must model filesystem, command, network, credential, MCP,
and external-side-effect authority as separate axes.

## Implication for ATeam Code

The provisional recommendation is for ATeam Code to own the control plane and
evidence model while treating each candidate harness as a capability-declared
execution driver. Based on interface visibility, prototype speed, event access,
licensing, and exit cost, Codex is the current candidate first backend. A pinned
DeepSeek Harness adapter is a candidate early test of backend neutrality, with
developer-preview instability and unverified security behavior as major
limitations. Claude Code and the Agent SDK remain behavior and extension
references unless their licensing and runtime dependency fit an explicitly
approved integration.

No backend has yet passed an ATeam Code runtime, safety, recovery, or replay
test. Repository structure is not runtime evidence.

## Primary external sources

- [Claude Code agent loop](https://code.claude.com/docs/en/agent-sdk/agent-loop)
- [Claude Code permissions](https://code.claude.com/docs/en/permissions)
- [Claude Code public repository license](https://github.com/anthropics/claude-code/blob/1f6015b5d578adf79c8527443328a216d6b6a3f1/LICENSE.md)
- [Codex Apache-2.0 license](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/LICENSE)
- [Codex App Server](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/app-server/README.md)
- [Codex CLI architecture](https://github.com/openai/codex/blob/8630bb3caecaff6abc6add450a88035d9f6d3f8c/codex-rs/README.md)
- [DeepSeek Harness MIT license](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/LICENSE)
- [DeepSeek Harness architecture](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md)
- [DeepSeek tool pipeline](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/tool-execution-pipeline.md)

The two `code.claude.com` links above were accessed on 2026-08-14 and may
change independently of the inspected Git revision.
