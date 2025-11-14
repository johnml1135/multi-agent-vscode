# Multi-Agent Workspace Extension Reimplementation Plan

This document captures a Spec-Kit-friendly plan for rebuilding the example PowerShell workflow under a fully-integrated VS Code extension that honors the project constitution (isolation, determinism, simplicity, inner-loop safety, observability).

## 1. Objectives & Guardrails
- **Isolation-First Agents**: Maintain per-agent git worktrees, Docker containers, NuGet caches, and VS Code windows without host side effects.
- **Deterministic Execution**: Pin Dockerfiles, container tags, git refs, dependency versions, and log every command; record repro metadata per agent.
- **Transparent Simplicity**: Provide explicit commands/UI, output structured logs, and minimize hidden automation.
- **Inner-Loop Safety & Performance**: Keep spin-up under 60s, command dispatch <250 ms, log streaming gap <250 ms, and enforce resource caps before starting agents.
- **Multi-Agent Observability & Ergonomics**: Emit per-agent logs with correlation IDs, color-code windows, allow pause/stop/teardown, and expose telemetry opt-in controls.

## 2. Example Implementation Inventory (What We Must Recreate)
| File                                                   | Functionality to Preserve                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spin-up-agents.ps1`                                   | Validate prereqs, ensure git fetch/reset, create worktrees/branches, assert NTFS drives, build Docker images (`docker-build-ipv4.ps1`), ensure per-agent containers + NuGet caches, write `.fw-agent/config.json`, generate `.vscode/tasks.json` + workspace color themes, open VS Code windows, table summary output. |
| `tear-down-agents.ps1`                                 | Discover agents, stop/remove containers, optionally delete worktrees & branches, clean caches, warn on dirty worktrees, close VS Code windows.                                                                                                                                                                         |
| `Invoke-AgentTask.ps1`                                 | Run build/clean/test inside container using msbuild/vstest, streaming logs back through Docker exec.                                                                                                                                                                                                                   |
| `AgentInfrastructure.psm1`                             | Shared helpers: tool detection, docker wrappers, git dir resolution, NTFS enforcement, bind-mount mapping, safe file writes.                                                                                                                                                                                           |
| `git-utilities.ps1`                                    | Safe git invocations, worktree inspection/removal/reset, metadata detachment.                                                                                                                                                                                                                                          |
| `VsCodeControl.psm1` & `open-code-with-containers.ps1` | Detect VS Code processes, open/close workspaces, propagate `FW_AGENT_CONTAINER`, color customization.                                                                                                                                                                                                                  |
| `multi-agent-containers-workflow.md`                   | User-facing workflow + constraints (Windows containers, FieldWorks `.sln`, memory budgets, env vars).                                                                                                                                                                                                                  |

## 3. Target Extension Capabilities
1. **Agent Creation Wizard** (command + custom view)
   - Collect repo root, agent count, base ref, solution path, memory caps, VS Code window options.
   - Validate Docker Windows mode, git status, drive types, resource availability.
   - Kick off git + Docker workflows with progress notifications and per-step logs.
2. **Agent Dashboard View**
   - Tree/List of agents showing branch, container, VS Code workspace, NuGet cache health, log status.
   - Inline actions: open VS Code window, reveal folder, copy logs, start/stop container, rebuild image.
3. **Task Runner Integration**
   - Register VS Code tasks (`multiAgent.build/clean/test`) that call container exec; expose quick-pick for agent selection and stream logs to terminals.
4. **Teardown & Cleanup Commands**
   - Options for stop-only vs. full removal, forced cleanup, handling dirty worktrees, closing VS Code windows.
5. **State & Configuration Management**
   - Store metadata in `.multiagent/state.json` per workspace, mirroring `.fw-agent/config.json`, plus per-agent logs in `.multiagent/logs`.
6. **Observability Surfaces**
   - Dedicated output channel per agent, correlation IDs, metrics for spin-up time and task latency, Constitution Check summary.
7. **Safety/Determinism Automation**
   - Automated Constitution Check script/command verifying pinned versions, sandbox evidence, performance data before PR merge.

## 4. Proposed Architecture
- **Core Services**
  - `GitService`: uses `simple-git` or spawned git CLI to fetch, create, reset worktrees, ensure excludes, detect dirty state.
  - `DockerService`: built on `dockerode` + custom IPv4 resolver; handles image build, container lifecycle, exec with streaming.
  - `FilesystemService`: NTFS + path validations, per-agent directory setup, config serialization.
  - `VSCodeWorkspaceService`: manage workspace files, color themes, open/close windows via `vscode.commands.executeCommand` and `vscode.env.openExternal` fallbacks.
  - `StateStore`: JSON persistence for agents, caches reproduction metadata, guardrail evidence.
- **Extension UI**
  - `AgentTreeDataProvider` for the explorer view.
  - Input forms (QuickPick/Webview) for spin-up parameters.
  - Output channels per agent + global log.
- **Task Runner Bridge**
  - VS Code `tasks.json` snippets generated programmatically or contributed tasks hooking into `AgentTaskRunner` that proxies docker exec.
- **Validation Layer**
  - Constitution enforcement module invoked before running spin-up/plan tasks; logs compliance status.

## 5. Spec-Kit Workflow
1. **Specification (`/speckit.specify`)**
   - Feature name: "Multi-Agent Workspace VS Code Extension".
   - User stories (P1): create agents; inspect/control agents. (P2): teardown resources. (P3): run build/test tasks.
   - Edge cases: missing Docker, non-NTFS drives, detached HEAD, dirty worktrees, resource exhaustion, non-Windows host.
   - Constitution Alignment: fill the new template section with concrete strategies described above.
2. **Plan (`/speckit.plan`)**
   - Use the updated Constitution Check to document: isolation proof (per-agent directories + containers), deterministic replay (pinned Docker tags, recorded commands), simplicity/logging (UI flows + log schema), performance budgets, observability hooks.
   - Define technical context (TypeScript 5.x, VS Code API, dockerode, simple-git, Jest/Vitest).
   - Choose structure: `src/agents`, `src/services`, `src/docker`, `src/git`, `src/views`, `resources/templates`, `specs/*`.
   - Complexity tracking for Windows-only assumptions or long-running containers.
3. **Tasks (`/speckit.tasks`)**
   - Phase 1 Setup: extension scaffolding, lint/test tooling, docker/git wrappers, Constitution validation script stub.
   - Phase 2 Foundational: implement GitService, DockerService, StateStore, Observability instrumentation.
   - Phase 3 US1: spin-up command + UI + workspace generation; add instrumentation tasks.
   - Phase 4 US2: dashboard view, per-agent controls, container start/stop, log streaming.
   - Phase 5 US3: teardown command, cache cleanup, VS Code close integration.
   - Phase 6 US4: task runner integration and build/test streaming.
   - Final phase: docs (`README`, quickstart), automate Constitution Check in CI.

## 6. Implementation Phases & Milestones
1. **Bootstrap (Week 0-1)**
   - Initialize VS Code extension project, add docker/git libs, implement Constitution Check script, set up Jest/Vitest + ESLint.
2. **Foundational Services (Week 1-2)**
   - GitService + tests for worktree management.
   - DockerService + IPv4 resolver + tests (mock docker socket).
   - StateStore & config schema definitions.
3. **Agent Creation Flow (Week 2-3)**
   - UI wizard, progress notifications, logging, workspace template generator, color theming, auto-open windows, `.fw-agent` compatibility layer.
4. **Agent Dashboard & Controls (Week 3-4)**
   - Tree view, per-agent actions, log streams, health indicators, pause/stop/resume.
5. **Teardown & Cleanup (Week 4)**
   - Command palette + view actions to stop/remove containers, detach worktrees, delete caches with safety prompts.
6. **Task Runner & Build/Test Integration (Week 4-5)**
   - Contributed tasks hooking into Docker exec, streaming output to terminals, success/failure notifications.
7. **Polish & Compliance (Week 5)**
   - Documentation, quickstart, telemetry opt-in, cross-platform guards (error early on non-Windows), final Constitution Check evidence.

## 7. Risks & Mitigations
- **Windows-only APIs**: Detect OS early; provide guidance if user attempts on unsupported host; keep core logic abstracted for future Linux support.
- **Docker Permissions/Mode**: Run pre-flight checks mirroring PowerShell scripts; show actionable remediation steps.
- **Git Worktree Corruption**: Unit tests plus dry-run mode; allow force cleanup similar to `-ForceCleanup` script flag.
- **Long Build Times**: Surface progress telemetry and warnings; allow staggering builds, set default memory cap per container.
- **VS Code CLI Availability**: Detect `code` binary, fall back to `vscode.openFolder` command.

## 8. Next Actions
1. Execute `/speckit.specify` with the inventory + guardrails above.
2. Run `/speckit.plan` referencing this document to lock architecture and Constitution Check evidence.
3. Run `/speckit.tasks` to generate implementation tasks grouped per user story, ensuring Constitution guardrails are annotated per task list.
4. Begin Phase 1 tasks; integrate automated Constitution Check into CI once foundational services land.
