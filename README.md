# ATeam Code

Evidence-driven agent teams for verified, reversible code changes.

ATeam Code is an experimental coding-agent control plane built around the
AGI Super Team role, Team, Skill, and Governor contracts.

## Status

Research and architecture planning. No runtime or safety claims have been
validated yet.

## Initial product promise

Given a development goal in an existing repository, ATeam Code proposes the
smallest sufficient agent team and verification plan, executes only after user
approval, and returns a reviewable change set with test evidence, independent
review, replayable events, and a safe undo path.

## Repository map

- `docs/product/` — users, journeys, scope, and acceptance criteria
- `docs/architecture/` — system boundaries and prototype architecture
- `docs/research/` — source-backed harness research
- `docs/decisions/` — architecture decision records
- `prototypes/` — isolated runtime experiments
- `evals/` — versioned tasks, safety cases, and scoring rubrics

## Relationship to AGI Super Team

AGI Super Team remains the canonical source for Team, Agent, Skill, routing,
and Governor content. ATeam Code will own runtime-neutral control-plane
contracts, execution evidence, policy enforcement, and the user experience.

## License

MIT. See [LICENSE](LICENSE).
