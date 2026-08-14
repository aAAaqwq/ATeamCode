# ADR 0001: Keep the control plane independent of execution runtimes

Status: Proposed — pending user decisions and prototype evidence.

Two product decisions remain unresolved:

1. whether owning a native runtime is a non-negotiable v0 requirement; and
2. whether plan approval is sufficient or sensitive tools also require
   per-action approval.

This ADR cannot become Accepted before those decisions are recorded.

## Context

ATeam Code must validate its product difference: selecting a minimal agent
team, enforcing an evidence-based workflow, performing independent review, and
returning a replayable and reversible change set.

Reimplementing a complete coding-agent runtime before validating that promise
would require model streaming, tool protocols, cancellation, process cleanup,
context compaction, persistence, provider compatibility, sandboxing, and
approval handling at once.

## Proposed decision

ATeam Code should own a runtime-neutral control plane and should use
replaceable execution drivers during the prototype:

```text
CLI
 |
 v
ATeam Code control plane
|- Goal and Team compiler
|- AGI manifest and Skill lock
|- Run DAG, budgets, and cancellation
|- Policy and approval authority
|- Append-only event ledger
|- ChangeSet and undo manager
`- Independent review gate
 |
 v
RuntimeDriver capability handshake
|- Codex App Server adapter        (prototype primary)
|- pinned DeepSeek Harness adapter (experimental)
`- minimal native runtime          (conditional later option)
 |
 v
isolated worktree + bounded tools + tests
```

The proposed control plane, not a vendor runtime, would own the canonical run
record, approval decisions, artifact hashes, and final review receipt.

## Proposed prototype contracts — validation pending

- `RunSpec`: goal, base revision, allowed scope, team, budgets, policy, and
  verification commands
- `RuntimeCapabilities`: explicit supported and missing guarantees
- `RuntimeDriver`: probe, start session/turn, stream, interrupt, approve, close
- `EventEnvelope`: ordered actor, causation, schema version, and raw event link
- `ToolInvocation`: requested, policy checked, approved or denied, started,
  succeeded, failed, or unknown
- `ChangeSet`: preimage hashes, patch, tests, and declared side effects
- `GovernorReceipt`: reviewed artifact hashes, findings, decision, and residual
  risk

## Required safety invariants — validation pending

- Missing capabilities fail closed.
- Repository, Skill, model, and tool output are untrusted text, not authority.
- File, shell, network, credential, MCP, and external actions have separate
  grants.
- A dirty worktree is isolated or rejected before mutation.
- Every child process belongs to the run cancellation scope.
- Unknown side-effect outcomes require human resolution; they are not retried
  automatically.
- Independent review receives read-only access to the exact artifact hashes it
  reviews.
- Credentials and sensitive tool output must be redacted before entering the
  event ledger, artifacts, logs, prompts, or review receipts.

## Alternatives

### Build a native runtime immediately

Maximum long-term control, but the slowest and riskiest way to test the product
hypothesis. Start only if measured adapter limits block policy enforcement,
event evidence, required providers, or core scheduling differentiation.

### Build directly on Codex App Server

Fastest prototype, but risks leaking Codex-specific concepts into the product
and canonical state.

### Fork DeepSeek Harness

Its inspected revision uses the
[MIT license](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/LICENSE),
exposes plugin seams, and declares itself a
[developer preview](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/README.md#developer-preview).
A direct fork would create upgrade and patch maintenance before its security
behavior has been independently tested.

## Validation gate

The decision remains proposed until version-pinned drivers run the same task
set and demonstrate:

- no unauthorized write, network access, or credential exposure observed
  within a predefined, observable conformance and adversarial test suite
- complete tool-to-approval-to-artifact traceability
- recovery after cancellation or injected runtime failure
- undo that preserves pre-existing user changes
- no orphan process or silent unknown side effect
- at least one common task passing through two drivers under the same control
  contracts

## Executable rollback procedure

Each driver must be version pinned and enabled through an explicit feature
flag. When a driver fails a required invariant:

1. interrupt and close its active sessions;
2. disable the feature flag and fail closed rather than silently switching
   backends;
3. preserve the canonical ledger and redacted evidence artifacts for review;
4. restore only the run-owned ChangeSet using verified preimage hashes;
5. keep the driver disabled until its conformance suite passes again.

Canonical schema changes require a separate migration ADR with a tested
backward path. If no adapter can satisfy the required invariants, the prototype
stops and the team reopens the native-runtime decision. A native runtime
experiment remains another feature-flagged driver until it passes the same
suite.

Raw vendor events are not preserved by default. If a future diagnostic mode
requires them, it must use isolated encrypted storage, least-privilege access,
a documented retention period, and redaction before any event is exposed to an
agent, reviewer, ledger, artifact, or ordinary log.
