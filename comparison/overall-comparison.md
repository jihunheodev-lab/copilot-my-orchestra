# Multi-Agent Orchestration Repository Comparison (Overall)

## Scope
- Baseline repository: `copilot-my-orchestra`
- Baseline files:
  - `.github/copilot-instructions.md`
  - `.github/agents/orchestrator.agent.md`
  - `.github/agents/researcher.agent.md`
  - `.github/agents/planner.agent.md`
  - `.github/agents/implementer.agent.md`
  - `.github/agents/tester.agent.md`
  - `.github/agents/reviewer.agent.md`
- Compared repositories:
  - `comparison/repos/copilot-orchestra`
  - `comparison/repos/Github-Copilot-Atlas`
  - `comparison/repos/Orchestration`
  - `comparison/repos/Craftsman`
  - `comparison/repos/coordinated-agent-team`
  - `comparison/repos/gem-team`

## Executive Summary
- Your baseline is a balanced 6-role pipeline with explicit dynamic technology discovery, planning-execution conversation isolation, and clear role separation.
- `copilot-orchestra` is the simplest and highly practical TDD workflow baseline (Conductor + 3 subagents).
- `Github-Copilot-Atlas` extends Orchestra with stronger context-conservation and parallel delegation patterns.
- `Orchestration` strongly separates UI ownership (Designer) and adds multi-reviewer strategy.
- `Craftsman` is process-heavy and artifact-driven (spec/plan/task files + Ralph loop + inspector loops).
- `coordinated-agent-team` is the most governance-heavy state-machine design (contract + workflow + dispatch reference + gate persistence).
- `gem-team` is execution-automation heavy (DAG planning, wave parallelism, structured YAML/JSON outputs, PRD coupling).

## Agent Inventory Snapshot
- Baseline (`copilot-my-orchestra`): 6 agents + 1 project instruction file
- `copilot-orchestra`: 4 agent files
- `Github-Copilot-Atlas`: 7 agent files
- `Orchestration`: 8 agent files
- `Craftsman`: 2 agent files (but internally defines additional subagent personas)
- `coordinated-agent-team`: 12 role files + 3 governance docs (`CONTRACT.md`, `WORKFLOW.md`, `DISPATCH-REFERENCE.md`)
- `gem-team`: 8 agent files

## Cross-Repo Comparison Matrix

| Dimension | Baseline (`copilot-my-orchestra`) | copilot-orchestra | Atlas | Orchestration | Craftsman | coordinated-agent-team | gem-team |
|---|---|---|---|---|---|---|---|
| Core topology | Orchestrator -> Research -> Plan -> Implement -> Test -> Review | Conductor -> Planning -> Implement -> Review | Atlas + Prometheus + Oracle/Explorer/Sisyphus/Frontend/Review | Orchestrator + Planner + Designer + Coder/FastCoder + Reviewer council | Plan Mode + Ralph Loop | Full state machine with 12 agents | Orchestrator + DAG worker agents |
| Dynamic stack discovery | Explicitly required; precedence-ordered file list | Limited | Limited/indirect | Not central; repo-constraint centric | Context discovery via planning interview | Process-level discovery via spec/architecture/research stages | Focus-area research + planner synthesis |
| Planning-Execution isolation | **Yes — two separate conversations via `execute plan:`** | Commit pause rhythm | Mandatory commit pauses per phase | Strong delegation rules | Ralph loop phases | Artifact-persistent session continuity | plan.yaml continuity |
| Workflow rigor | Medium-high | Medium | High | High | Very high | Very high | Very high |
| Artifact persistence | `plans/<task-name>-plan.md` with Approval Status FSM | `plans/` docs | configurable plan dir | task orchestration, less strict artifact contract | `.agents/changes/...` rich artifacts + `PROGRESS.md` | `.agents-work/<session>/...` strict artifact model | `docs/plan/{plan_id}/plan.yaml`, PRD, logs, findings |
| Gate model | Intent classification + stage flow + approval status | Mandatory approval/commit pauses | Mandatory approval/commit pauses | Strong delegation rules + reviewer council | Task inspector + phase inspector + HITL optional | Mandatory APPROVE_DESIGN / REVIEW_STRATEGY / repair loops | Plan review + execution retries + PRD compliance gate |
| Parallelism | Supported via subagent flow | Minimal | Strongly encouraged | Strongly encouraged | Loop-oriented, mostly sequential per task | Controlled, mostly deterministic dispatch | Wave-based DAG parallel execution |
| Output contract strictness | Medium | Medium | Medium-high | Medium | High | Very high (JSON schema + persistence protocol) | High (JSON + YAML schemas) |
| Security posture | Reviewer quality checks | Review-centric | Review-centric | Review-council pattern | Inspector quality focus | Dedicated Security agent + gate semantics | Dedicated reviewer security + OWASP + PRD compliance |
| UX/UI specialization | Generalist (no dedicated designer) | No dedicated UI role | Dedicated Frontend subagent | Dedicated Designer role (hard ownership) | Indirect (task-driven) | Dedicated Designer role | Browser tester + doc writer + reviewer separation |

## Detailed Comparative Analysis

### 1) Role Separation and Ownership
- Baseline is clean and understandable: each role has a clear responsibility and boundaries. Read-only boundaries (Researcher, Planner, Reviewer) are enforced at the toolset level via YAML frontmatter.
- `Orchestration` and `coordinated-agent-team` push role ownership further, especially for design and review governance.
- `gem-team` also has clear role boundaries, with explicit prohibition of cross-role implementation.
- `Craftsman` keeps only two top-level agents but enforces strict sub-persona loop behavior.

### 2) Workflow Determinism vs Flexibility
- Baseline: balanced; the two-conversation split (planning / execution) adds clear structure without over-engineering.
- `copilot-orchestra`/`Atlas`: pragmatic deterministic loops with user checkpoints.
- `coordinated-agent-team`: strongest determinism, explicit state transitions and persistence checks.
- `gem-team`: deterministic execution engine via DAG/waves rather than linear phase flow.
- `Craftsman`: deterministic loops at task/phase level with inspector-driven correction.

### 3) State and Artifact Management
- Baseline uses a lightweight `plans/<task-name>-plan.md` with an `Approval Status` FSM (pending → in_progress → done). Simpler than peers but sufficient for single-session tasks.
- `coordinated-agent-team` is strongest in auditable state (`status.json`, `tasks.yaml`, gate tracking, decision tracking).
- `gem-team` is strongest in execution-state planning (`plan.yaml`) and domain findings artifacts.
- `Craftsman` is strongest in human-reviewable planning artifacts.

### 4) Quality and Verification Strategy
- Baseline includes Tester (dynamic framework detection, 3-phase methodology) and Reviewer (six-dimension audit) with clear read-only boundaries.
- `copilot-orchestra`/`Atlas` emphasize TDD and phase-by-phase review/commit rhythm.
- `Craftsman` adds explicit Task Inspector and Phase Inspector loops.
- `coordinated-agent-team` formalizes repair loops and retry budgets.
- `gem-team` formalizes verification per agent and PRD compliance.

### 5) Scalability for Large Tasks
- Baseline can scale, but may need stronger artifact contracts for long-running/multi-day efforts.
- `Atlas` and `gem-team` are stronger for large contexts due to heavy delegation and parallel strategy.
- `coordinated-agent-team` is strongest for team-scale governance and reproducibility.
- `Craftsman` is strongest for deep-planning + autonomous implementation loops.

### 6) Complexity Cost
- Baseline and `copilot-orchestra` are easiest to adopt quickly.
- `Atlas` adds moderate complexity for better scale.
- `Orchestration` adds policy complexity for role ownership.
- `Craftsman`, `coordinated-agent-team`, and `gem-team` have higher setup/maintenance overhead.

## Strengths and Gaps of Baseline (Relative View)

### Strengths
- **Planning-Execution isolation** via `execute plan:` two-conversation split is unique among compared repos.
- **Explicit dynamic technology discovery** policy with precedence-ordered file list — more concrete than any peer.
- Read-only boundaries enforced at the toolset level (YAML `tools` field) — not just instructed, but architecturally constrained.
- Role definitions are understandable and practical with low onboarding overhead.
- Tester role includes the most systematic framework-detection methodology (3-phase: detect → analyze → replicate).
- Reviewer enforces six-dimension review with mandatory file:line citations.

### Gaps
- No durable session artifact beyond the plan file (no `status.json`, `tasks.yaml`, `report.md`).
- No explicit retry budget and repair loop semantics (FIX_REVIEW / FIX_TESTS loops with max count).
- No dedicated UX/UI ownership role (Designer/Frontend specialist).
- No machine-readable inter-agent I/O contract — agents communicate in natural language.
- No parallel execution path for independent tasks.

## Recommended Evolution Path for Baseline

### Phase A (Low friction)
- Add a lightweight session artifact alongside the plan file:
  - A simple `status` field already exists as `Approval Status` in the plan file.
  - Extend to a minimal `plans/<task-name>-status.json` for longer-running tasks if needed.
- Keep existing 6-agent topology unchanged.

### Phase B (Quality gates)
- Add explicit retry budget rules:
  - `FIX_REVIEW` / `FIX_TESTS` loops with max 3 retries before escalating to user.
  - Currently handled by "retry once, then report" — could be made more explicit.

### Phase C (Specialization)
- Add one optional specialist first (recommended): `Designer` or `Frontend-Engineer`.
- Add optional `CONTRACT.md` (minimal version) to formalize inter-agent context forwarding schemas.

### Phase D (Scale)
- If needed, evolve from linear pipeline to DAG/wave execution for independent tasks.
- Keep this optional to avoid over-engineering for small tasks.

## Bottom Line
- Your baseline is a strong, practical middle point with clear differentiators in planning isolation and discovery policy.
- For team governance and strict reproducibility, the best reference is `coordinated-agent-team`.
- For high-throughput parallel execution, the best reference is `gem-team` and `Atlas`.
- For planning depth + autonomous loop discipline, the best reference is `Craftsman`.
- For simplicity and quick adoption, `copilot-orchestra` remains the easiest baseline comparator.
