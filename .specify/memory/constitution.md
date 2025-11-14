<!--
Sync Impact Report
Version change: 0.0.0 → 1.0.0
Modified principles:
- [PRINCIPLE_1_NAME] → I. Isolation-First Agents
- [PRINCIPLE_2_NAME] → II. Deterministic Execution & Reproducibility
- [PRINCIPLE_3_NAME] → III. Transparent Simplicity
- [PRINCIPLE_4_NAME] → IV. Inner-Loop Safety & Performance
- [PRINCIPLE_5_NAME] → V. Multi-Agent Observability & Ergonomics
Added sections:
- Operational Constraints
- Development Workflow
Removed sections: None
Templates requiring updates:
- ✅ .specify/templates/plan-template.md
- ✅ .specify/templates/spec-template.md
- ✅ .specify/templates/tasks-template.md
Follow-up TODOs: None
-->
# Multi-Agent Workspace Constitution

## Core Principles

### I. Isolation-First Agents
- Every agent session runs in a sandboxed workspace with no shared state, no host-level side effects, and explicit allow-lists for tools and files.
- Shared services (e.g., terminals, kernels, containers) must expose mediated channels that can be audited and reset between runs.
- Reviews MUST reject any change that increases host coupling without a documented teardown plan.
*Rationale: Isolation keeps concurrent agents safe, debuggable, and compliant with user expectations.*

### II. Deterministic Execution & Reproducibility
- All agent workflows pin container images, toolchains, and seeds so that rerunning the same command anywhere yields identical artifacts and logs.
- Specs and plans MUST declare dependency versions and state what inputs are required to reproduce results.
- Caches are allowed only when their invalidation strategy is documented in the plan.md Constitution Check.
*Rationale: Determinism enables the "rebuild anywhere, same results" promise.*

### III. Transparent Simplicity
- Commands, scripts, and UX copy favor explicit parameters over magic; every automation step emits structured logs plus a human-readable summary.
- Documentation MUST show the minimum number of steps to get from spec to running agents and highlight how to inspect logs.
- New features default to off unless their user story demonstrates they reduce steps or cognitive overhead.
*Rationale: Simple, inspectable flows keep the extension ergonomic even with many agents.*

### IV. Inner-Loop Safety & Performance
- No change may regress the measured latency of the "edit → run → observe" loop beyond agreed budgets (target: <2s command dispatch, <250ms log streaming gap).
- Guardrails (sandbox validation, disk quotas, CPU caps) are enforced before agents start; failing a guardrail aborts execution instead of degrading silently.
- Pull requests must include benchmarks or reasoning for performance-neutrality when touching core agent lifecycle paths.
*Rationale: A fast, predictable inner loop is how multi-agent collaboration stays useful.*

### V. Multi-Agent Observability & Ergonomics
- Each agent emits scoped logs with correlation IDs so concurrent sessions are traceable and replayable.
- Users can inspect, pause, or terminate any agent without affecting others; UI interventions call into the same audited command surface used by automation.
- Telemetry is opt-in, anonymized, and stored locally by default; exporting requires explicit consent surfaced in docs and UI.
*Rationale: Visibility plus ergonomic controls prevent cross-agent interference and surprises.*

## Operational Constraints

- All automation runs locally inside VS Code with zero hidden network or filesystem mutations outside the workspace directory.
- Sensitive credentials remain outside agent sandboxes unless a spec declares the secret name, scope, and revocation path.
- Extension upgrades MUST be reversible: provide scripts or commands to restore the prior version and cached artifacts.
- Compatibility guardrails: changes touching sandboxing, dependency resolution, or logging require integration tests across Windows, macOS, and Linux.

## Development Workflow

1. Draft the feature spec (`/specs/.../spec.md`) capturing isolation, determinism, and simplicity constraints for each user story.
2. Generate the implementation plan via `/speckit.plan`; the "Constitution Check" must document how the feature satisfies each principle or list violations in the Complexity Tracking table.
3. Populate `/speckit.tasks` output so tasks are grouped per user story, each referencing the guardrails it exercises (sandboxing, deterministic replay, or observability hooks).
4. Before merging, run the automated Constitution Check (script TBD) plus manual review verifying: isolation proof, deterministic rerun instructions, log transparency demo, and inner-loop performance evidence.

## Governance

- The constitution supersedes other process docs for safety, reproducibility, and performance matters; conflicting guidance is invalid until amended here.
- Amendment proposals live in a PR that updates this file plus any impacted templates, includes a changelog entry, and cites benchmarks or experiments when touching safety/performance rules.
- **Versioning**: MAJOR for removing or redefining principles, MINOR for new principles or sections, PATCH for clarifications. Each release records ISO dates for ratification and amendment.
- **Compliance Reviews**: Every release and significant feature PR must document Constitution adherence in its checklist; missing evidence blocks merge until resolved.

**Version**: 1.0.0 | **Ratified**: 2025-11-14 | **Last Amended**: 2025-11-14
