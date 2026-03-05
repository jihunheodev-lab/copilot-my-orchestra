# 1vs1 Comparison (Baseline vs Each Repository)

## Baseline Used
- Repository: `copilot-my-orchestra`
- Core files:
  - `.github/copilot-instructions.md`
  - `.github/agents/orchestrator.agent.md`
  - `.github/agents/researcher.agent.md`
  - `.github/agents/planner.agent.md`
  - `.github/agents/implementer.agent.md`
  - `.github/agents/tester.agent.md`
  - `.github/agents/reviewer.agent.md`

---

## 1) Baseline vs `copilot-orchestra`

### Common Ground
- Both use an orchestrator-led lifecycle.
- Both separate planning/implementation/review concerns.
- Both emphasize TDD and staged progress.

### Key Differences
- Baseline has 6-role model including dedicated Researcher and Tester.
- `copilot-orchestra` is leaner (Conductor + 3 subagents), with explicit commit pause rhythm.
- Baseline enforces read-only boundaries at the toolset level (YAML `tools` field); `copilot-orchestra` relies on instructions only.
- Baseline uses a two-conversation planning/execution split via `execute plan:`; `copilot-orchestra` pauses at commit points within one conversation.

### What Baseline Can Borrow
- Lightweight mandatory pause points for user approval.
- Simple phase completion artifact templates for progress traceability.

### What Baseline Should Keep
- Richer role granularity (Researcher/Tester separation).
- Dynamic discovery-first principle with precedence-ordered file list.
- Planning-execution conversation isolation.

---

## 2) Baseline vs `Github-Copilot-Atlas`

### Common Ground
- Shared lineage and similar conductor-subagent architecture.
- TDD-first implementation flow and review gate pattern.

### Key Differences
- `Atlas` introduces stronger context-conservation strategy and explicit parallel subagent playbook (up to 10 concurrent).
- `Atlas` adds specialist roles (Explorer, Frontend-Engineer) and a separate autonomous planner (`Prometheus`).
- Baseline's planning-execution split is comparable to Atlas's per-phase commit pauses, but with cleaner conversation isolation.
- Baseline is simpler to reason about; Atlas is more scalable for large-scope tasks.

### What Baseline Can Borrow
- Explicit parallel research policy (Explorer-like pre-mapping).
- Context budget heuristic (when to delegate vs when to read directly).
- Optional frontend specialist role.

### What Baseline Should Keep
- Cleaner role set for routine workloads.
- Less cognitive overhead in operator flow.

---

## 3) Baseline vs `Orchestration`

### Common Ground
- Clear orchestrator-centric delegation model.
- Strong separation between planning and coding.

### Key Differences
- `Orchestration` enforces design ownership strictly (Designer handles all frontend decisions/changes).
- Uses reviewer council pattern (multiple reviewer agents) and FastCoder/Coder split.
- Baseline currently has no dedicated Designer and no multi-reviewer consensus layer.

### What Baseline Can Borrow
- Optional Designer role for UI-heavy tasks.
- Optional fast/complex implementer split for parallel speed.
- Multi-reviewer pattern for critical changes only.

### What Baseline Should Keep
- Single-review path for standard tasks to minimize cost/latency.

---

## 4) Baseline vs `Craftsman`

### Common Ground
- Orchestration-first mindset.
- Strong intent to avoid direct orchestrator coding.
- Planning and implementation responsibilities are separated.

### Key Differences
- `Craftsman` is highly artifact-driven:
  - `.agents/changes/<JIRA>/01-specification.md`
  - `.agents/changes/<JIRA>/02-plan.md`
  - `.agents/changes/<JIRA>/03-tasks-*`
  - `PROGRESS.md`
- Uses Ralph loop + Task Inspector + Phase Inspector verification chain.
- Baseline is lighter and faster for ad-hoc feature delivery.

### What Baseline Can Borrow
- Optional `PROGRESS.md` style explicit task tracker for long runs.
- Inspector concept for stricter quality gates in complex phases.
- HITL phase boundary control option.

### What Baseline Should Keep
- Current lower-friction workflow for everyday coding.

---

## 5) Baseline vs `coordinated-agent-team`

### Common Ground
- Both define orchestrator role boundaries and multi-agent specialization.
- Both value process discipline and explicit gates.

### Key Differences
- `coordinated-agent-team` is contract-first and state-machine-first:
  - `.github/agents/CONTRACT.md`
  - `.github/agents/WORKFLOW.md`
  - `.github/agents/DISPATCH-REFERENCE.md`
- Enforces JSON-only inter-agent contracts, strict gate persistence, retry budgets (max 3 per loop type), and durable decision records.
- Maintains session artifacts in `.agents-work/<session>/...` as system-of-record.
- Baseline does not yet enforce machine-verifiable protocol — agents communicate in natural language.

### What Baseline Can Borrow
- Minimal `CONTRACT.md` with input/output schema.
- Minimal `.agents-work/<session>/status.json` and `tasks.yaml` persistence.
- Explicit repair-loop retry budget (e.g., max 3).

### What Baseline Should Keep
- Simpler flow without full enterprise-grade overhead by default.

---

## 6) Baseline vs `gem-team`

### Common Ground
- Explicit role boundaries and delegation-first approach.
- Research -> planning -> implementation -> review pattern.

### Key Differences
- `gem-team` is DAG/wave execution oriented with strong machine-readable planning:
  - `docs/plan/{plan_id}/plan.yaml`
  - `docs/plan/{plan_id}/research_findings_*.yaml`
  - `docs/prd.yaml`
- Strong structured output requirements (JSON/YAML), failure typing, and logging.
- Dedicated browser tester/devops/doc writer roles.
- Baseline is linear-pipeline centric and less artifact-rigid.

### What Baseline Can Borrow
- Optional DAG-mode for independent tasks.
- Structured failure taxonomy (`transient`, `needs_replan`, `escalate`).
- Lightweight PRD alignment checks for large initiatives.

### What Baseline Should Keep
- Linear default for speed and lower orchestration complexity.

---

## Synthesis Across All 1vs1 Comparisons

### Best Reference by Purpose
- Simplicity and quick startup: `copilot-orchestra`
- Parallel context-efficient scaling: `Github-Copilot-Atlas`
- Strict UI ownership and review plurality: `Orchestration`
- Planning depth + autonomous implementation loops: `Craftsman`
- Enterprise-grade process governance: `coordinated-agent-team`
- DAG and artifact-driven execution at scale: `gem-team`

### Practical Upgrade Set for Baseline (Priority)
1. Add minimal session persistence (`.agents-work/<session>/status.json`, `tasks.yaml`, `report.md`) for long-running tasks.
2. Add explicit gate and retry-loop semantics (fix loops with max retry count).
3. Add one optional specialist role (`Designer` or `Frontend-Engineer`).
4. Add optional advanced mode for parallel waves (DAG) only when task graph demands it.

### Expected Outcome
- Maintain baseline usability for normal tasks.
- Gain traceability and reliability for large or long-running tasks.
- Improve team collaboration with clearer artifacts and decision records.
