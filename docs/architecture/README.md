# Architecture

This directory owns runtime-neutral system boundaries, protocols, threat
models, failure modes, and prototype architecture.

The current proposed direction keeps a runtime-neutral control plane and uses
capability-declared execution drivers. Codex App Server is the first prototype
candidate; DeepSeek Harness, OpenClaw, Hermes Agent, Claude Agent SDK behavior,
and a possible minimal native runtime remain research inputs.

- [Control plane and Supervisor](0001-control-plane-and-supervisor.md)
