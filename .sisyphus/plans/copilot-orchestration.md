# GitHub Copilot Multi-Agent Orchestration System

## TL;DR

> **Quick Summary**: Build a 6-agent orchestration system for GitHub Copilot using Pure Declarative approach (`.github/agents/*.agent.md` files only). The system provides a 5-stage code generation pipeline (Research → Plan → Implement → Test → Review) that works fully orchestrated in VS Code Agent Mode and gracefully degrades to standalone agents in Copilot CLI.
> 
> **Deliverables**:
> - 6 agent files in `.github/agents/` (Orchestrator + 5 worker agents)
> - `.github/copilot-instructions.md` for repo-level context
> - `README.md` with architecture docs, usage guide, and compatibility matrix
> - Validation scripts for YAML frontmatter and cross-reference integrity
> 
> **Estimated Effort**: Medium
> **Parallel Execution**: YES — 4 waves (6 agents in parallel in Wave 2)
> **Critical Path**: Task 1 → Task 2 (Orchestrator) → Task 8 (Cross-ref + README) → Task 9 (Final validation)

---

## Context

### Original Request
User wants to build a GitHub Copilot Chat / CLI orchestration system. The system should coordinate specialized agents through a declarative pipeline: plan → implement → test → review.

### Interview Summary
**Key Discussions**:
- Researched 5 approaches (Custom Agents, Copilot SDK, MCP Servers, VS Code Extension, Copilot Extensions)
- User chose Pure Declarative: only `.agent.md` markdown files, no code
- Target: VS Code Agent Mode (primary) + Copilot CLI (secondary with graceful degradation)
- Workflow: 5-stage pipeline (Research → Plan → Implement → Test → Review)
- 6 agents: Orchestrator, Researcher, Planner, Implementer, Tester, Reviewer
- Technology-agnostic via dynamic discovery (agents analyze project files to detect stack)
- Open source distribution
- Hybrid orchestration: subagents for autonomous flow + handoffs for user-controlled steps

**Research Findings**:
- microsoft/hve-core: Hierarchical subagent orchestration (researcher → implementor → evaluator)
- dotnet/maui: 4-phase PR review workflow with shared rules
- microsoft/skills: Planner with handoffs to frontend/backend
- foxminchan/BookWorm: Multi-agent with model specs and handoffs
- microsoft/PowerToys: PlanIssue agent with handoffs to FixIssue
- VS Code vs CLI feature gap: handoffs, subagents, model selection are VS Code only
- `mcp-servers` field NOT supported in VS Code custom agents
- 30,000 character limit per agent markdown body
- Subagents don't inherit conversation history — orchestrator must explicitly forward context

### Metis Review
**Identified Gaps** (addressed):
- Environment compatibility: VS Code primary, CLI secondary with graceful degradation
- Orchestration model: Hybrid (handoffs + subagents) instead of one or the other
- Technology-agnostic definition: Dynamic discovery via project file analysis
- Each agent must work standalone AND as part of pipeline
- `description` field required in ALL agents (for GitHub.com compatibility)
- Cross-reference validation needed for agent name consistency
- Context flow between subagent stages must be explicit in orchestrator instructions

---

## Work Objectives

### Core Objective
Create a production-ready, open-source GitHub Copilot multi-agent orchestration system that automates the code generation pipeline through specialized agents coordinated by a central orchestrator.

### Concrete Deliverables
- `.github/agents/orchestrator.agent.md` — Intent classification, subagent delegation, handoff coordination
- `.github/agents/researcher.agent.md` — Codebase/docs investigation (read-only)
- `.github/agents/planner.agent.md` — Requirements analysis + implementation plan generation
- `.github/agents/implementer.agent.md` — Code writing following existing patterns
- `.github/agents/tester.agent.md` — Test generation and execution
- `.github/agents/reviewer.agent.md` — Code review + quality verification
- `.github/copilot-instructions.md` — Repository-level context and conventions
- `README.md` — Architecture docs, usage guide, compatibility matrix, installation instructions

### Definition of Done
- [x] All 6 agent files have valid YAML frontmatter (parseable by Python yaml.safe_load)
- [x] All agent files have `description` field
- [x] All handoff targets and subagent references resolve to existing agents
- [x] All tool names are from the official VS Code tool taxonomy
- [x] Each agent works standalone (clear self-contained instructions without orchestrator context)
- [x] Orchestrator provides both handoff buttons and subagent delegation capability
- [x] README includes architecture diagram, usage guide, and VS Code/CLI compatibility matrix
- [x] Git repository initialized with meaningful initial commit

### Must Have
- All 6 agents with clear role separation
- Hybrid orchestration (handoffs + subagents) in orchestrator
- Dynamic technology discovery in Tester and Implementer agents
- Standalone capability for CLI compatibility
- `description` field in every agent for GitHub.com compatibility
- Read-only tools for Researcher and Reviewer (no edit/execute)
- Full edit/execute tools for Implementer and Tester
- Cross-reference integrity across all agent files

### Must NOT Have (Guardrails)
- NO MCP server configurations (`mcp-servers` field not supported in VS Code custom agents)
- NO Copilot SDK code or programmatic agents
- NO VS Code extension code
- NO concrete model names in frontmatter (leave to user's selected model)
- NO language-specific hardcoded instructions (use dynamic discovery instead)
- NO `disable-model-invocation: true` on any agent (all agents should be user-invocable)
- NO excessive comments or over-abstraction in agent instructions
- NO placeholder or TODO items left in agent files
- NO hooks, skills, or CLI plugin configurations in V1

---

## Verification Strategy (MANDATORY)

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed. No exceptions.
> Acceptance criteria requiring "user manually tests/confirms" are FORBIDDEN.

### Test Decision
- **Infrastructure exists**: NO (markdown-only project)
- **Automated tests**: None (declarative markdown files, not code)
- **Framework**: N/A
- **Validation**: YAML frontmatter parsing + cross-reference scripts via Python

### QA Policy
Every task MUST include agent-executed QA scenarios.
Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **YAML Validation**: Use Bash (Python one-liner) — Parse frontmatter, check required fields
- **Cross-Reference**: Use Bash (Python script) — Verify handoff targets and subagent names resolve
- **Tool Name Validation**: Use Bash (Python script) — Check tool names against known-valid list
- **Content Verification**: Use Bash (grep/read) — Verify agent body content completeness

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — foundation):
└── Task 1: Initialize repository structure + copilot-instructions.md [quick]

Wave 2 (After Wave 1 — ALL 6 agents in PARALLEL):
├── Task 2: Create orchestrator.agent.md [deep]
├── Task 3: Create researcher.agent.md [unspecified-high]
├── Task 4: Create planner.agent.md [unspecified-high]
├── Task 5: Create implementer.agent.md [unspecified-high]
├── Task 6: Create tester.agent.md [unspecified-high]
└── Task 7: Create reviewer.agent.md [unspecified-high]

Wave 3 (After Wave 2 — validation + documentation):
└── Task 8: Cross-reference validation + README.md [writing]

Wave FINAL (After Wave 3 — final QA + commit):
├── Task F1: Full validation suite + git commit [quick]
├── Task F2: Plan compliance audit [deep]
├── Task F3: Scope fidelity check [deep]
└── Task F4: Agent quality review [unspecified-high]

Critical Path: Task 1 → Task 2 → Task 8 → Task F1
Parallel Speedup: ~65% faster than sequential (6 agents in parallel)
Max Concurrent: 6 (Wave 2)
```

### Dependency Matrix

| Task | Depends On | Blocks | Wave |
|------|-----------|--------|------|
| 1 | — | 2,3,4,5,6,7 | 1 |
| 2 | 1 | 8 | 2 |
| 3 | 1 | 8 | 2 |
| 4 | 1 | 8 | 2 |
| 5 | 1 | 8 | 2 |
| 6 | 1 | 8 | 2 |
| 7 | 1 | 8 | 2 |
| 8 | 2,3,4,5,6,7 | F1,F2,F3,F4 | 3 |
| F1 | 8 | — | FINAL |
| F2 | 8 | — | FINAL |
| F3 | 8 | — | FINAL |
| F4 | 8 | — | FINAL |

### Agent Dispatch Summary

- **Wave 1**: **1 task** — T1 → `quick`
- **Wave 2**: **6 tasks** — T2 → `deep`, T3-T7 → `unspecified-high`
- **Wave 3**: **1 task** — T8 → `writing`
- **Wave FINAL**: **4 tasks** — F1 → `quick`, F2 → `deep`, F3 → `deep`, F4 → `unspecified-high`

---

## TODOs

- [x] 1. Initialize Repository Structure + Copilot Instructions

  **What to do**:
  - Run `git init` in the project directory
  - Create `.github/agents/` directory
  - Create `.github/copilot-instructions.md` with repository-level context:
    - Project description: "Multi-agent orchestration system for GitHub Copilot"
    - State that this project provides specialized agents for code generation pipelines
    - Note the technology-agnostic design philosophy
    - List the 6 agents and their roles briefly
    - Include convention: agents use dynamic discovery to detect project stack
    - Include instruction that agents should analyze project files (package.json, Makefile, pyproject.toml, etc.) before making technology-specific decisions

  **Must NOT do**:
  - Do not create any agent files yet (that's Wave 2)
  - Do not create README.md yet (that's Wave 3)
  - Do not add any MCP server configurations
  - Do not install any dependencies

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Simple file creation and git init — no complex logic
  - **Skills**: [`git-master`]
    - `git-master`: Git init and directory setup

  **Parallelization**:
  - **Can Run In Parallel**: NO (foundation task)
  - **Parallel Group**: Wave 1 (solo)
  - **Blocks**: Tasks 2, 3, 4, 5, 6, 7
  - **Blocked By**: None

  **References**:

  **Pattern References**:
  - `.github/copilot-instructions.md` docs: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot — This is the official format for repo-level Copilot instructions. Follow the documented structure.

  **External References**:
  - Custom Agents directory convention: https://code.visualstudio.com/docs/copilot/customization/custom-agents — Confirms `.github/agents/` as the standard location.
  - LawrenceHwang's architecture gist: https://gist.github.com/LawrenceHwang/6194421c3bb4208fff84452b403e191a — Shows how copilot-instructions.md fits with agents in the 5-component architecture.

  **WHY Each Reference Matters**:
  - The copilot-instructions docs show the exact file format and location
  - The architecture gist provides context on how instructions + agents work together

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Git repository initialized
    Tool: Bash
    Preconditions: Empty project directory
    Steps:
      1. Run: git rev-parse --git-dir
      2. Assert output is ".git"
    Expected Result: ".git" printed, confirming git init succeeded
    Failure Indicators: "fatal: not a git repository"
    Evidence: .sisyphus/evidence/task-1-git-init.txt

  Scenario: Directory structure created correctly
    Tool: Bash
    Preconditions: Git initialized
    Steps:
      1. Run: test -d .github/agents && echo "agents dir exists" || echo "agents dir MISSING"
      2. Run: test -f .github/copilot-instructions.md && echo "instructions file exists" || echo "instructions file MISSING"
    Expected Result: Both "exists" messages printed
    Failure Indicators: Any "MISSING" message
    Evidence: .sisyphus/evidence/task-1-directory-structure.txt

  Scenario: Copilot instructions file has required content
    Tool: Bash
    Preconditions: .github/copilot-instructions.md exists
    Steps:
      1. Run: grep -c "orchestrat" .github/copilot-instructions.md
      2. Run: grep -c "dynamic" .github/copilot-instructions.md
      3. Assert both counts > 0 (file mentions orchestration and dynamic discovery)
      4. Run: wc -l .github/copilot-instructions.md
      5. Assert line count > 10 (substantive content, not a stub)
    Expected Result: Both grep counts > 0, line count > 10
    Failure Indicators: Count of 0 for either grep, or line count <= 10
    Evidence: .sisyphus/evidence/task-1-instructions-content.txt
  ```

  **Commit**: YES
  - Message: `chore: initialize repository structure with copilot instructions`
  - Files: `.github/agents/` (directory), `.github/copilot-instructions.md`
  - Pre-commit: Directory and file existence checks

- [x] 2. Create Orchestrator Agent (orchestrator.agent.md)

  **What to do**:
  - Create `.github/agents/orchestrator.agent.md` with YAML frontmatter:
    - `name: Orchestrator`
    - `description: Coordinates the code generation pipeline by classifying intent and delegating to specialized agents. Supports autonomous subagent delegation and user-controlled handoff workflows.`
    - `tools: ['agent', 'search', 'read', 'fetch']` — needs `agent` for subagent invocation, plus search/read/fetch for context gathering
    - `agents: ['Researcher', 'Planner', 'Implementer', 'Tester', 'Reviewer']` — all 5 workers as subagents
    - `handoffs:` — 5 handoff entries, one per worker agent:
      - `label: Research Codebase` → `agent: Researcher` → `prompt: Research the codebase and relevant documentation to understand the context for this task.` → `send: false`
      - `label: Create Implementation Plan` → `agent: Planner` → `prompt: Based on the research findings, create a detailed implementation plan.` → `send: false`
      - `label: Implement Changes` → `agent: Implementer` → `prompt: Implement the changes according to the plan.` → `send: false`
      - `label: Generate Tests` → `agent: Tester` → `prompt: Generate and run tests for the implemented changes.` → `send: false`
      - `label: Review Code` → `agent: Reviewer` → `prompt: Review the implemented changes and test results for quality and correctness.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Orchestrator — the central coordinator of a 5-stage code generation pipeline
    2. **Intent classification**: When user provides a request, classify it into categories:
       - New feature → full pipeline (Research → Plan → Implement → Test → Review)
       - Bug fix → abbreviated pipeline (Research → Implement → Test → Review)
       - Refactoring → Research → Plan → Implement → Test → Review
       - Question/investigation → Research only
       - Quick fix → Implement → Test
    3. **Autonomous mode instructions**: When working autonomously with subagents:
       - Invoke Researcher subagent with specific research questions
       - Synthesize research results and pass FULL context to Planner subagent
       - Pass the plan + research context to Implementer subagent
       - Pass implementation details to Tester subagent
       - Pass all prior context to Reviewer subagent
       - Explicitly describe how to forward context between subagent calls (subagents don't inherit conversation history)
    4. **Handoff mode instructions**: When user prefers step-by-step control, explain that handoff buttons are available and the user can click through the pipeline stages
    5. **CLI standalone instructions**: When running outside VS Code (no handoffs/subagents available), provide a manual workflow guide:
       - Instruct user to invoke @Researcher first, then @Planner, etc.
       - Suggest copy-pasting key findings between agent invocations
    6. **Dynamic discovery**: Before delegating, analyze the project to understand:
       - Language/framework (check package.json, requirements.txt, go.mod, Cargo.toml, Makefile, etc.)
       - Test framework (jest, vitest, pytest, go test, etc.)
       - Build system and project structure
       - Pass this context to subagents

  **Must NOT do**:
  - Do not use `mcp-servers` field
  - Do not specify concrete model names (no `model:` field)
  - Do not use `disable-model-invocation: true`
  - Do not include language-specific instructions (use dynamic discovery)
  - Do not exceed 30,000 characters in the markdown body

  **Recommended Agent Profile**:
  - **Category**: `deep`
    - Reason: Most complex agent — requires careful instruction design for hybrid orchestration (subagents + handoffs + CLI fallback), intent classification logic, and context forwarding patterns
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Explore design options before writing the agent instructions

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 3, 4, 5, 6, 7)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - microsoft/hve-core `rpi-agent.agent.md`: Hierarchical subagent orchestration. The Orchestrator calls subagents via `agents` field and `agent` tool. Subagent call protocol: provide task context in prompt, receive results, pass to next subagent. Key pattern: orchestrator synthesizes between stages.
  - microsoft/skills planner agent: Shows handoff pattern with `label`, `agent`, `prompt`, `send: false`. Good example of how handoff buttons work in sequential workflow.
  - foxminchan/BookWorm agents: Multi-agent with handoffs and model specs. Shows practical handoff chain: Plan → Implement with `send: false` for user approval.
  - microsoft/PowerToys PlanIssue agent: Shows how orchestrator hands off to FixIssue with templated prompt.

  **API/Type References**:
  - Custom Agents YAML schema: https://docs.github.com/en/copilot/reference/custom-agents-configuration — Exact field names, types, and constraints for frontmatter
  - VS Code Custom Agents docs: https://code.visualstudio.com/docs/copilot/customization/custom-agents — Features supported in VS Code (handoffs, subagents, model)

  **External References**:
  - VS Code multi-agent blog: https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development — Explains runSubAgent behavior, context isolation, and how to pass context between subagents
  - Mission Control blog: https://github.blog/ai-and-ml/github-copilot/how-to-orchestrate-agents-using-mission-control/ — Patterns for orchestrating multiple agent workflows

  **WHY Each Reference Matters**:
  - hve-core shows the EXACT pattern for subagent orchestration — how to call, receive, forward
  - microsoft/skills shows the EXACT handoff button pattern — label, agent, prompt structure
  - Custom Agents config reference is the source of truth for YAML field definitions
  - VS Code multi-agent blog explains subagent context isolation (critical for instruction design)

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Orchestrator YAML frontmatter is valid and complete
    Tool: Bash
    Preconditions: .github/agents/orchestrator.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/orchestrator.agent.md').read().split('---')[1]); assert data.get('name')=='Orchestrator', f'name is {data.get(\"name\")}'; assert 'description' in data, 'missing description'; assert 'agent' in data.get('tools',[]), 'missing agent tool'; assert len(data.get('agents',[]))>=5, f'only {len(data.get(\"agents\",[]))} agents'; assert len(data.get('handoffs',[]))>=5, f'only {len(data.get(\"handoffs\",[]))} handoffs'; print('Orchestrator frontmatter OK')"
      2. Assert exit code 0
    Expected Result: "Orchestrator frontmatter OK"
    Failure Indicators: AssertionError with specific field failure
    Evidence: .sisyphus/evidence/task-2-orchestrator-frontmatter.txt

  Scenario: Orchestrator has all 5 handoff targets pointing to valid agent names
    Tool: Bash
    Preconditions: orchestrator.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/orchestrator.agent.md').read().split('---')[1]); targets=set(h['agent'] for h in data.get('handoffs',[])); expected={'Researcher','Planner','Implementer','Tester','Reviewer'}; missing=expected-targets; assert not missing, f'missing handoffs to: {missing}'; print(f'All 5 handoff targets present: {targets}')"
      2. Assert exit code 0
    Expected Result: "All 5 handoff targets present: {Researcher, Planner, Implementer, Tester, Reviewer}"
    Failure Indicators: AssertionError listing missing agent targets
    Evidence: .sisyphus/evidence/task-2-orchestrator-handoffs.txt

  Scenario: Orchestrator body contains required instruction sections
    Tool: Bash
    Preconditions: orchestrator.agent.md exists
    Steps:
      1. Read the file content below the second "---"
      2. Grep for "intent" or "classify" (intent classification section)
      3. Grep for "subagent" or "autonomous" (autonomous mode instructions)
      4. Grep for "handoff" or "button" (handoff mode instructions)
      5. Grep for "CLI" or "standalone" (CLI fallback instructions)
      6. Grep for "package.json" or "requirements.txt" or "discovery" (dynamic discovery)
      7. Assert all 5 greps find matches
    Expected Result: All 5 content sections present
    Failure Indicators: Any grep returns 0 matches
    Evidence: .sisyphus/evidence/task-2-orchestrator-body-sections.txt

  Scenario: Orchestrator does NOT contain forbidden elements
    Tool: Bash
    Preconditions: orchestrator.agent.md exists
    Steps:
      1. Grep for "mcp-servers" in frontmatter — must return 0 matches
      2. Grep for "model:" in frontmatter — must return 0 matches
      3. Grep for "disable-model-invocation" — must return 0 matches
    Expected Result: Zero matches for all forbidden patterns
    Failure Indicators: Any match found
    Evidence: .sisyphus/evidence/task-2-orchestrator-forbidden.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 3. Create Researcher Agent (researcher.agent.md)

  **What to do**:
  - Create `.github/agents/researcher.agent.md` with YAML frontmatter:
    - `name: Researcher`
    - `description: Investigates the codebase, documentation, and external resources to gather context for development tasks. Read-only — never modifies files.`
    - `tools: ['search', 'search/codebase', 'search/usages', 'read', 'read/problems', 'fetch', 'web', 'web/fetch', 'web/githubRepo']` — comprehensive read-only tool set
    - `agents: []` — no subagent capability (worker agent)
    - `handoffs:` — single handoff to Planner:
      - `label: Create Plan from Research` → `agent: Planner` → `prompt: Based on these research findings, create a detailed implementation plan.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Researcher — a read-only investigator that gathers context before any changes are made
    2. **Research methodology**:
       - Codebase investigation: search for relevant files, read implementations, trace dependencies, find usages
       - Pattern discovery: identify existing patterns, conventions, naming schemes, directory structures
       - Documentation review: check README, CONTRIBUTING, inline docs, comments
       - External research: fetch relevant docs, API references, library documentation
    3. **Output format**: Research findings must be structured as:
       - **Relevant files**: List of files with their roles
       - **Existing patterns**: Code patterns found (with examples)
       - **Dependencies**: What the target code depends on and what depends on it
       - **Constraints**: Discovered limitations, edge cases, or requirements
       - **Recommendations**: Suggested approach based on findings
    4. **Dynamic discovery**: At the start of any research task:
       - Detect language/framework by checking project config files (package.json, pyproject.toml, go.mod, Cargo.toml, Makefile, pom.xml, build.gradle, etc.)
       - Identify test framework and test file locations
       - Identify build system and entry points
    5. **CLI standalone instructions**: When invoked directly (not via Orchestrator), ask the user what they want investigated, then produce a structured research report

  **Must NOT do**:
  - Do not include ANY edit tools (`edit`, `editFiles`)
  - Do not include `execute` tool
  - Do not include `agent` tool (no subagent capability)
  - Do not use `mcp-servers` field
  - Do not specify model
  - Do not include instructions that modify files in any way

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Requires careful instruction design for structured research output format and dynamic discovery
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Design the research methodology and output format

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 2, 4, 5, 6, 7)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - microsoft/hve-core Researcher subagent: Uses read-only tools for codebase investigation. Produces structured findings that feed into the implementor. Key pattern: explicit output format for research results.
  - EmeaAppGbb/spec2cloud: Research agent pattern for gathering context before implementation.

  **API/Type References**:
  - VS Code tool names: https://code.visualstudio.com/docs/copilot/customization/custom-agents — Full list of available tool identifiers for the `tools` field. Use exact names: `search`, `search/codebase`, `search/usages`, `read`, `fetch`, `web`, etc.

  **External References**:
  - Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration — YAML frontmatter field definitions

  **WHY Each Reference Matters**:
  - hve-core Researcher shows how to structure read-only research output that feeds into next pipeline stage
  - VS Code tool names doc is needed to use exact valid tool identifiers

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Researcher YAML frontmatter is valid and read-only
    Tool: Bash
    Preconditions: .github/agents/researcher.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/researcher.agent.md').read().split('---')[1]); assert data.get('name')=='Researcher'; assert 'description' in data; tools=data.get('tools',[]); forbidden=['edit','editFiles','execute','agent']; violations=[t for t in tools if t in forbidden]; assert not violations, f'FORBIDDEN tools found: {violations}'; assert 'search' in tools or 'search/codebase' in tools, 'missing search tool'; assert 'read' in tools, 'missing read tool'; print('Researcher frontmatter OK')"
      2. Assert exit code 0
    Expected Result: "Researcher frontmatter OK"
    Failure Indicators: AssertionError with specific violation
    Evidence: .sisyphus/evidence/task-3-researcher-frontmatter.txt

  Scenario: Researcher has handoff to Planner
    Tool: Bash
    Preconditions: researcher.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/researcher.agent.md').read().split('---')[1]); targets=[h['agent'] for h in data.get('handoffs',[])]; assert 'Planner' in targets, f'No handoff to Planner, targets: {targets}'; print('Researcher handoff OK')"
    Expected Result: "Researcher handoff OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-3-researcher-handoff.txt

  Scenario: Researcher body contains structured research methodology
    Tool: Bash
    Preconditions: researcher.agent.md exists
    Steps:
      1. Grep for "pattern" or "convention" (pattern discovery)
      2. Grep for "dependency" or "depends" (dependency analysis)
      3. Grep for "package.json" or "pyproject" or "discovery" (dynamic discovery)
      4. Grep for "standalone" or "CLI" or "direct" (standalone instructions)
      5. Assert all 4 greps find matches
    Expected Result: All 4 sections present
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-3-researcher-body.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 4. Create Planner Agent (planner.agent.md)

  **What to do**:
  - Create `.github/agents/planner.agent.md` with YAML frontmatter:
    - `name: Planner`
    - `description: Analyzes requirements and research findings to create detailed, actionable implementation plans. Read-only — focuses on analysis and planning, never modifies code.`
    - `tools: ['search', 'search/codebase', 'search/usages', 'read', 'read/problems', 'fetch']` — read-only tools for codebase analysis during planning
    - `agents: []` — no subagent capability (worker agent)
    - `handoffs:` — single handoff to Implementer:
      - `label: Start Implementation` → `agent: Implementer` → `prompt: Implement the changes according to this plan.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Planner — you transform research findings and user requirements into structured, actionable implementation plans
    2. **Plan generation methodology**:
       - Analyze research findings (from Researcher or user-provided context)
       - Break down the task into discrete, ordered steps
       - Identify which files need to be created/modified
       - Specify what changes each file needs
       - Define acceptance criteria for each step
       - Identify risks and edge cases
    3. **Plan output format** (structured markdown):
       - **Objective**: What we're building/changing (1-2 sentences)
       - **Prerequisites**: What must be true before starting
       - **Steps**: Numbered list, each with:
         - File(s) to create/modify
         - What to do (specific, actionable)
         - Why (rationale)
         - Acceptance criteria
       - **Testing approach**: How to verify the changes work
       - **Risks**: Potential issues and mitigations
    4. **Dynamic discovery**: Analyze codebase to inform the plan:
       - Project structure and conventions
       - Testing patterns to follow
       - Build/lint commands
       - Existing similar implementations to reference
    5. **CLI standalone instructions**: When invoked directly, ask for task description, then analyze the codebase and produce a plan

  **Must NOT do**:
  - Do not include edit/execute tools
  - Do not include `agent` tool
  - Do not use `mcp-servers` field
  - Do not specify model
  - Do not create plans that assume a specific language/framework (use dynamic discovery)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Requires structured output format design and careful plan methodology
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Design the plan template and methodology

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 2, 3, 5, 6, 7)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - microsoft/skills planner agent: Shows planning agent with read-only tools that produces structured plans and hands off to implementation agents. Key pattern: plan template with steps, files, and acceptance criteria.
  - dotnet/maui PLAN-TEMPLATE.md: Plan template used across multiple agents. Shows how to structure implementation plans with phases and criteria.
  - microsoft/PowerToys PlanIssue agent: Planning agent that creates implementation plans for issues, then hands off to FixIssue.

  **External References**:
  - Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration — YAML field definitions

  **WHY Each Reference Matters**:
  - microsoft/skills planner shows the exact read-only + handoff pattern for a planning agent
  - PLAN-TEMPLATE.md from dotnet/maui shows a production plan template format
  - PowerToys shows how to create a plan then hand off to implementation

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Planner YAML frontmatter is valid and read-only
    Tool: Bash
    Preconditions: .github/agents/planner.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/planner.agent.md').read().split('---')[1]); assert data.get('name')=='Planner'; assert 'description' in data; tools=data.get('tools',[]); forbidden=['edit','editFiles','execute','agent']; violations=[t for t in tools if t in forbidden]; assert not violations, f'FORBIDDEN tools: {violations}'; print('Planner frontmatter OK')"
      2. Assert exit code 0
    Expected Result: "Planner frontmatter OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-4-planner-frontmatter.txt

  Scenario: Planner has handoff to Implementer
    Tool: Bash
    Preconditions: planner.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/planner.agent.md').read().split('---')[1]); targets=[h['agent'] for h in data.get('handoffs',[])]; assert 'Implementer' in targets, f'No handoff to Implementer'; print('Planner handoff OK')"
    Expected Result: "Planner handoff OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-4-planner-handoff.txt

  Scenario: Planner body contains structured plan output format
    Tool: Bash
    Preconditions: planner.agent.md exists
    Steps:
      1. Grep for "Objective" or "objective" (plan structure)
      2. Grep for "Step" or "step" (step-by-step breakdown)
      3. Grep for "acceptance" or "criteria" or "verify" (acceptance criteria)
      4. Grep for "risk" or "edge case" (risk assessment)
      5. Assert all 4 greps find matches
    Expected Result: All 4 plan sections present
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-4-planner-body.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 5. Create Implementer Agent (implementer.agent.md)

  **What to do**:
  - Create `.github/agents/implementer.agent.md` with YAML frontmatter:
    - `name: Implementer`
    - `description: Writes production code following existing patterns and conventions. Creates and modifies files according to implementation plans.`
    - `tools: ['editFiles', 'search', 'search/codebase', 'search/usages', 'read', 'read/problems', 'execute', 'fetch']` — full edit capability
    - `agents: []` — no subagent capability (worker agent)
    - `handoffs:` — single handoff to Tester:
      - `label: Generate Tests` → `agent: Tester` → `prompt: Generate and run tests for the changes I just implemented.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Implementer — you write production-quality code following existing codebase patterns and conventions
    2. **Implementation methodology**:
       - Read the plan (from Planner or user-provided)
       - Before writing code, search the codebase for existing patterns, naming conventions, and similar implementations
       - Follow the project's established patterns EXACTLY (don't invent new patterns)
       - Make minimal, focused changes — only what the plan specifies
       - Use existing utilities and helpers when available instead of writing new ones
    3. **Dynamic discovery rules**:
       - Detect language/framework from project config files
       - Find and follow existing code style (indentation, naming, imports)
       - Check for linter/formatter configs (.eslintrc, .prettierrc, pyproject.toml, etc.)
       - Run linter/formatter after changes if config exists
    4. **Code quality rules**:
       - No `any` types (TypeScript) or equivalent loose typing
       - No empty catch blocks
       - No console.log/print in production code (only in debug/development sections)
       - No commented-out code
       - No TODOs or FIXMEs left behind
       - Meaningful variable and function names
    5. **CLI standalone instructions**: When invoked directly, ask what needs to be implemented, analyze the codebase for patterns, then implement

  **Must NOT do**:
  - Do not use `mcp-servers` field
  - Do not specify model
  - Do not include `agent` tool
  - Do not include instructions that are specific to any single language (use dynamic discovery)
  - Do not include excessive abstraction instructions ("extract to utility", "create a base class")

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Requires nuanced instructions for pattern-following, dynamic discovery, and code quality rules
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Design implementation methodology and quality rules

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 2, 3, 4, 6, 7)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - microsoft/hve-core Implementor subagent: Implementation agent with full edit tools. Key pattern: follows conventions discovered by researcher, makes minimal changes, follows the plan step-by-step.
  - foxminchan/BookWorm implementation agents: Shows code quality rules and pattern-following instructions for implementation agents.

  **API/Type References**:
  - VS Code tools: `editFiles` is the primary edit tool. `execute` for running commands (build, lint, format).

  **External References**:
  - Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration

  **WHY Each Reference Matters**:
  - hve-core Implementor shows how to instruct an agent to follow discovered patterns
  - BookWorm shows practical code quality guard rules in agent instructions

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Implementer YAML frontmatter has edit tools
    Tool: Bash
    Preconditions: .github/agents/implementer.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/implementer.agent.md').read().split('---')[1]); assert data.get('name')=='Implementer'; assert 'description' in data; tools=data.get('tools',[]); assert 'editFiles' in tools or 'edit' in tools, 'missing edit tool'; assert 'search' in tools or 'search/codebase' in tools, 'missing search'; assert 'agent' not in tools, 'agent tool forbidden'; print('Implementer frontmatter OK')"
    Expected Result: "Implementer frontmatter OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-5-implementer-frontmatter.txt

  Scenario: Implementer has handoff to Tester
    Tool: Bash
    Preconditions: implementer.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/implementer.agent.md').read().split('---')[1]); targets=[h['agent'] for h in data.get('handoffs',[])]; assert 'Tester' in targets; print('Implementer handoff OK')"
    Expected Result: "Implementer handoff OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-5-implementer-handoff.txt

  Scenario: Implementer body contains pattern-following and dynamic discovery
    Tool: Bash
    Preconditions: implementer.agent.md exists
    Steps:
      1. Grep for "pattern" or "convention" (pattern-following instructions)
      2. Grep for "package.json" or "pyproject" or "discovery" (dynamic discovery)
      3. Grep for "minimal" or "focused" (scope control)
      4. Grep for "standalone" or "CLI" or "direct" (standalone instructions)
      5. Assert all 4 greps find matches
    Expected Result: All 4 sections present
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-5-implementer-body.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 6. Create Tester Agent (tester.agent.md)

  **What to do**:
  - Create `.github/agents/tester.agent.md` with YAML frontmatter:
    - `name: Tester`
    - `description: Generates and runs tests for code changes. Discovers the project's testing framework dynamically and follows existing test patterns.`
    - `tools: ['editFiles', 'search', 'search/codebase', 'read', 'read/problems', 'read/terminalLastCommand', 'execute']` — edit + execute for test creation and running
    - `agents: []` — no subagent capability (worker agent)
    - `handoffs:` — single handoff to Reviewer:
      - `label: Review Changes` → `agent: Reviewer` → `prompt: Review the implemented changes and test results for quality and correctness.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Tester — you generate comprehensive tests and verify implementations work correctly
    2. **Test discovery methodology** (dynamic — most critical section):
       - Step 1: Identify test framework from project config:
         - JavaScript/TypeScript: check package.json scripts and devDependencies for jest, vitest, mocha, bun test
         - Python: check pyproject.toml/setup.cfg for pytest, unittest
         - Go: built-in `go test`
         - Rust: built-in `cargo test`
         - Java: check pom.xml/build.gradle for JUnit, TestNG
         - etc. — list common frameworks per language ecosystem
       - Step 2: Find existing test files and analyze patterns:
         - Test file naming convention (*.test.ts, *_test.go, test_*.py, etc.)
         - Test directory structure (src/__tests__/, tests/, test/, etc.)
         - Assertion style (expect().toBe(), assert, assertEqual, etc.)
         - Mock/stub patterns
         - Test fixture patterns
       - Step 3: Follow discovered patterns exactly when generating new tests
    3. **Test generation rules**:
       - Write tests that match the project's existing test style
       - Cover happy path, edge cases, and error cases
       - Use descriptive test names that explain what's being tested
       - Keep tests focused — one behavior per test
       - Use existing test utilities/helpers when available
    4. **Test execution**:
       - Run the discovered test command (npm test, pytest, go test, etc.)
       - Report results clearly: passed, failed (with details), or errored
       - If tests fail, analyze the failure and suggest fixes
    5. **CLI standalone instructions**: When invoked directly, discover the test framework, analyze what needs testing, generate and run tests

  **Must NOT do**:
  - Do not use `mcp-servers` field
  - Do not specify model
  - Do not include `agent` tool
  - Do not hardcode test framework commands (always discover dynamically)
  - Do not generate tests that depend on specific test infrastructure not in the project

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Complex dynamic discovery logic across many language ecosystems
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Design the dynamic test discovery methodology

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 2, 3, 4, 5, 7)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - microsoft/hve-core Evaluator subagent: Test/evaluation agent that verifies implementations. Key pattern: runs tests, reports results, suggests fixes for failures.
  - dotnet/maui test patterns: Shows test agent behavior in the PR review pipeline — discovers test commands, runs them, reports results.

  **External References**:
  - Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration

  **WHY Each Reference Matters**:
  - hve-core Evaluator shows how to structure a test agent that dynamically runs and reports results
  - dotnet/maui shows test discovery and execution in a real pipeline

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Tester YAML frontmatter has edit + execute tools
    Tool: Bash
    Preconditions: .github/agents/tester.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/tester.agent.md').read().split('---')[1]); assert data.get('name')=='Tester'; assert 'description' in data; tools=data.get('tools',[]); assert 'editFiles' in tools or 'edit' in tools, 'missing edit tool'; assert 'execute' in tools, 'missing execute tool'; assert 'agent' not in tools, 'agent tool forbidden'; print('Tester frontmatter OK')"
    Expected Result: "Tester frontmatter OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-6-tester-frontmatter.txt

  Scenario: Tester has handoff to Reviewer
    Tool: Bash
    Preconditions: tester.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/tester.agent.md').read().split('---')[1]); targets=[h['agent'] for h in data.get('handoffs',[])]; assert 'Reviewer' in targets; print('Tester handoff OK')"
    Expected Result: "Tester handoff OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-6-tester-handoff.txt

  Scenario: Tester body contains dynamic test discovery methodology
    Tool: Bash
    Preconditions: tester.agent.md exists
    Steps:
      1. Grep for "package.json" (JavaScript/TypeScript framework detection)
      2. Grep for "pytest" or "pyproject" (Python framework detection)
      3. Grep for "go test" or "cargo test" (Go/Rust detection)
      4. Grep for "pattern" or "convention" (follow existing test patterns)
      5. Grep for "standalone" or "CLI" (standalone instructions)
      6. Assert all 5 greps find matches
    Expected Result: All 5 sections present, covering multiple ecosystems
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-6-tester-body.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 7. Create Reviewer Agent (reviewer.agent.md)

  **What to do**:
  - Create `.github/agents/reviewer.agent.md` with YAML frontmatter:
    - `name: Reviewer`
    - `description: Reviews code changes for correctness, quality, security, and adherence to project patterns. Read-only — provides feedback but never modifies code directly.`
    - `tools: ['search', 'search/codebase', 'search/usages', 'read', 'read/problems']` — read-only, focused on analysis
    - `agents: []` — no subagent capability (worker agent)
    - `handoffs:` — two handoffs:
      - `label: Request Changes` → `agent: Implementer` → `prompt: Please address the review feedback above.` → `send: false`
      - `label: Back to Orchestrator` → `agent: Orchestrator` → `prompt: Review is complete. Here are the findings.` → `send: false`
  - Markdown body (system prompt) must include:
    1. **Role statement**: You are the Reviewer — the final quality gate in the code generation pipeline. You review code for correctness, quality, security, and convention adherence.
    2. **Review methodology**:
       - **Correctness**: Does the code do what the plan specified? Are there logic errors?
       - **Patterns**: Does the code follow existing codebase patterns and conventions?
       - **Security**: Input validation, injection risks, credential exposure, dependency risks
       - **Quality**: Code readability, naming, DRY principle, appropriate abstraction level
       - **Testing**: Are tests comprehensive? Do they cover edge cases? Do they pass?
       - **Scope**: Does the code stay within the plan's scope? No scope creep?
    3. **Review output format** (structured):
       - **Summary**: Overall assessment (Approve / Request Changes / Needs Discussion)
       - **Correctness**: ✅/❌ with details
       - **Patterns**: ✅/❌ with specifics
       - **Security**: ✅/❌ with issues
       - **Quality**: ✅/❌ with suggestions
       - **Testing**: ✅/❌ with gaps
       - **Scope**: ✅/❌ with deviations
       - **Action items**: Numbered list of required changes (if any)
    4. **Dynamic discovery**: Analyze project to inform review:
       - Find linter/formatter configs and check compliance
       - Find coding standards docs (CONTRIBUTING.md, style guides)
       - Compare new code against existing patterns in the same directory
    5. **CLI standalone instructions**: When invoked directly, ask what to review, then produce a structured review report. Include instructions for reviewing specific files or recent changes.
    6. **Iteration support**: If changes are requested, the Reviewer provides specific, actionable feedback. When the Implementer makes changes, the Reviewer can re-review.

  **Must NOT do**:
  - Do not include ANY edit tools (`edit`, `editFiles`)
  - Do not include `execute` tool
  - Do not include `agent` tool
  - Do not use `mcp-servers` field
  - Do not specify model
  - Do not provide vague feedback ("looks good", "needs improvement") — always be specific

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: Requires comprehensive review methodology and structured output format
  - **Skills**: [`superpowers/brainstorming`]
    - `superpowers/brainstorming`: Design the review checklist and feedback format

  **Parallelization**:
  - **Can Run In Parallel**: YES (with Tasks 2, 3, 4, 5, 6)
  - **Parallel Group**: Wave 2
  - **Blocks**: Task 8
  - **Blocked By**: Task 1

  **References**:

  **Pattern References**:
  - dotnet/maui PR review agent (`pr.md`): 4-phase review workflow. Key pattern: structured review output with specific categories (correctness, patterns, security). Shows how to provide actionable feedback vs vague comments.
  - microsoft/hve-core RPI Validator: Review agent that validates implementations against plans. Key pattern: explicit acceptance criteria checking.
  - dotnet/maui SHARED-RULES.md: Shared review rules across multiple agents. Shows how to define quality standards.

  **External References**:
  - Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration

  **WHY Each Reference Matters**:
  - dotnet/maui PR review agent shows the exact structured review output format for a production review pipeline
  - hve-core Validator shows how to check implementations against plans
  - SHARED-RULES.md shows how to define shared quality standards

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Reviewer YAML frontmatter is valid and read-only
    Tool: Bash
    Preconditions: .github/agents/reviewer.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/reviewer.agent.md').read().split('---')[1]); assert data.get('name')=='Reviewer'; assert 'description' in data; tools=data.get('tools',[]); forbidden=['edit','editFiles','execute','agent']; violations=[t for t in tools if t in forbidden]; assert not violations, f'FORBIDDEN tools: {violations}'; print('Reviewer frontmatter OK')"
    Expected Result: "Reviewer frontmatter OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-7-reviewer-frontmatter.txt

  Scenario: Reviewer has handoffs to Implementer and Orchestrator
    Tool: Bash
    Preconditions: reviewer.agent.md exists
    Steps:
      1. Run: python -c "import yaml; data=yaml.safe_load(open('.github/agents/reviewer.agent.md').read().split('---')[1]); targets=[h['agent'] for h in data.get('handoffs',[])]; assert 'Implementer' in targets, f'No handoff to Implementer'; assert 'Orchestrator' in targets, f'No handoff to Orchestrator'; print('Reviewer handoffs OK')"
    Expected Result: "Reviewer handoffs OK"
    Failure Indicators: AssertionError
    Evidence: .sisyphus/evidence/task-7-reviewer-handoffs.txt

  Scenario: Reviewer body contains structured review methodology
    Tool: Bash
    Preconditions: reviewer.agent.md exists
    Steps:
      1. Grep for "correctness" or "correct" (correctness check)
      2. Grep for "security" or "Security" (security review)
      3. Grep for "pattern" or "convention" (pattern adherence)
      4. Grep for "Approve" or "Request Changes" (review verdict)
      5. Grep for "standalone" or "CLI" (standalone instructions)
      6. Assert all 5 greps find matches
    Expected Result: All 5 sections present
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-7-reviewer-body.txt
  ```

  **Commit**: NO (grouped with Wave 2)

- [x] 8. Cross-Reference Validation + README.md

  **What to do**:
  - **Part A: Cross-reference validation** — Run validation scripts to verify integrity:
    1. YAML frontmatter syntax validation for all 6 agent files
    2. `description` field presence in all agents
    3. Handoff target validation: every `handoffs[].agent` value references an existing agent `name`
    4. Subagent reference validation: every `agents[]` value (except `*` and `[]`) references an existing agent `name`
    5. Tool name validation: every tool in `tools[]` is a valid VS Code tool name from the official taxonomy
    6. Fix any issues found (agent files may need minor corrections from Wave 2)
  - **Part B: README.md creation** — Create comprehensive documentation:
    - **Project title and description**: "Copilot My Orchestra — Multi-Agent Orchestration System for GitHub Copilot"
    - **Features list**: 5-stage pipeline, hybrid orchestration, dynamic tech discovery, etc.
    - **Architecture diagram** (ASCII art showing agent flow):
      ```
      User Request → [Orchestrator] → [Researcher] → [Planner] → [Implementer] → [Tester] → [Reviewer] → Done
      ```
    - **Installation**: Clone the repo or copy `.github/agents/` to your project
    - **Usage guide**:
      - VS Code: How to invoke agents (@Orchestrator, @Researcher, etc.)
      - CLI: How to invoke agents (copilot chat with @agent-name)
    - **Per-agent documentation**: Table with name, description, tools, role
    - **Compatibility matrix**: What works in VS Code vs CLI vs GitHub.com
      - VS Code: Full orchestration (handoffs + subagents)
      - CLI: Standalone agents only
      - GitHub.com Coding Agent: Standalone agents only
    - **Pipeline workflow explanation**: How the 5 stages connect
    - **Customization guide**: How to modify agents for specific needs
    - **Contributing**: Brief contribution guidelines
    - **License**: MIT

  **Must NOT do**:
  - Do not add emojis unless they serve a documentation purpose
  - Do not write excessive prose — keep README focused and scannable
  - Do not include implementation details about the validation scripts in README
  - Do not claim features that don't exist (e.g., don't claim CLI orchestration works)

  **Recommended Agent Profile**:
  - **Category**: `writing`
    - Reason: Primary deliverable is documentation (README.md) with validation as precursor
  - **Skills**: [`superpowers/verification-before-completion`]
    - `superpowers/verification-before-completion`: Ensure validation passes before moving to README

  **Parallelization**:
  - **Can Run In Parallel**: NO (depends on ALL Wave 2 tasks)
  - **Parallel Group**: Wave 3 (solo)
  - **Blocks**: F1, F2, F3, F4
  - **Blocked By**: Tasks 2, 3, 4, 5, 6, 7

  **References**:

  **Pattern References**:
  - EmeaAppGbb/spec2cloud README: Multi-agent orchestration documentation with architecture diagram, agent catalog, and usage instructions. Key pattern: clear agent documentation table.
  - foxminchan/BookWorm README: Shows how to document a multi-agent Copilot setup with per-agent descriptions.

  **API/Type References**:
  - Known valid VS Code tool names for validation: `search`, `search/codebase`, `search/usages`, `read`, `read/problems`, `read/terminalLastCommand`, `read/terminalSelection`, `edit`, `editFiles`, `execute`, `fetch`, `web`, `web/fetch`, `web/githubRepo`, `github/*`, `github-artifacts/*`, `agent`, `runSubagent`, `todo`

  **External References**:
  - VS Code Custom Agents: https://code.visualstudio.com/docs/copilot/customization/custom-agents — Compatibility info for the matrix
  - GitHub Custom Agents config: https://docs.github.com/en/copilot/reference/custom-agents-configuration — GitHub.com compatibility limitations

  **WHY Each Reference Matters**:
  - spec2cloud and BookWorm READMEs show how other projects document multi-agent systems
  - VS Code and GitHub docs provide the authoritative compatibility information

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: All 6 agent files pass YAML validation
    Tool: Bash
    Preconditions: All 6 agent files exist in .github/agents/
    Steps:
      1. Run Python script that:
         a. Parses YAML frontmatter of each file
         b. Checks description field exists
         c. Reports errors per file
      2. Assert zero errors
    Expected Result: "All 6 agent files valid"
    Failure Indicators: Any "invalid" or "missing" errors
    Evidence: .sisyphus/evidence/task-8-yaml-validation.txt

  Scenario: Cross-reference integrity passes
    Tool: Bash
    Preconditions: All 6 agent files validated
    Steps:
      1. Run Python script that:
         a. Collects all agent names from frontmatter
         b. Checks every handoffs[].agent references an existing name
         c. Checks every agents[] item references an existing name (except * and [])
      2. Assert zero "BROKEN" references
    Expected Result: "Cross-reference check complete" with no broken references
    Failure Indicators: Any "BROKEN:" line
    Evidence: .sisyphus/evidence/task-8-crossref-validation.txt

  Scenario: Tool names are all valid
    Tool: Bash
    Preconditions: All 6 agent files validated
    Steps:
      1. Run Python script that:
         a. Collects all tools from all agents
         b. Checks each tool against the known-valid list
         c. Reports unknown tools
      2. Assert zero unknown tools
    Expected Result: All tool names recognized
    Failure Indicators: Any "UNKNOWN tool:" line
    Evidence: .sisyphus/evidence/task-8-tool-validation.txt

  Scenario: README.md has required sections
    Tool: Bash
    Preconditions: README.md created
    Steps:
      1. Grep for architecture diagram or flow chart (ASCII art or mermaid)
      2. Grep for "Installation" or "install" (installation section)
      3. Grep for "Usage" or "usage" (usage guide)
      4. Grep for "VS Code" (VS Code instructions)
      5. Grep for "CLI" (CLI instructions)
      6. Grep for "Compatibility" or "compatibility" (compatibility matrix)
      7. Grep for "MIT" or "License" (license)
      8. Assert all 7 greps find matches
    Expected Result: All 7 README sections present
    Failure Indicators: Any grep returns 0
    Evidence: .sisyphus/evidence/task-8-readme-sections.txt

  Scenario: README compatibility matrix is accurate
    Tool: Bash
    Preconditions: README.md exists
    Steps:
      1. Verify README mentions VS Code supports handoffs and subagents
      2. Verify README mentions CLI does NOT support handoffs/subagents
      3. Verify README mentions GitHub.com has limited support
    Expected Result: Accurate compatibility information
    Failure Indicators: Incorrect claims about CLI/GitHub.com supporting handoffs
    Evidence: .sisyphus/evidence/task-8-readme-compatibility.txt
  ```

  **Commit**: YES
  - Message: `feat: add multi-agent orchestration system with 6 agents and documentation`
  - Files: All `.github/agents/*.agent.md`, `README.md`, any corrected files
  - Pre-commit: All validation scripts pass

---

## Final Verification Wave (MANDATORY — after ALL implementation tasks)

> 4 review agents run in PARALLEL. ALL must APPROVE. Rejection → fix → re-run.

- [x] F1. **Full Validation Suite + Git Commit**

  **What to do**:
  - Run complete YAML frontmatter validation on all 6 agent files
  - Run cross-reference validation (handoff targets, subagent names)
  - Run tool name validation against official VS Code tool taxonomy
  - Verify all files have `description` field
  - Verify `.github/copilot-instructions.md` exists and is non-empty
  - Verify `README.md` exists with required sections
  - Create initial git commit with all files

  **Must NOT do**:
  - Do not modify any agent files — only validate
  - Do not push to remote

  **Recommended Agent Profile**:
  - **Category**: `quick`
  - **Skills**: [`git-master`, `superpowers/verification-before-completion`]

  **Parallelization**:
  - **Can Run In Parallel**: YES (with F2, F3, F4)
  - **Parallel Group**: Wave FINAL
  - **Blocks**: None
  - **Blocked By**: Task 8

  **References**:
  - Python validation scripts defined in Task 8's QA scenarios
  - Official VS Code tool names: `search`, `search/codebase`, `search/usages`, `read`, `read/problems`, `read/terminalLastCommand`, `read/terminalSelection`, `edit`, `editFiles`, `execute`, `fetch`, `web`, `web/fetch`, `web/githubRepo`, `github/*`, `agent`, `runSubagent`, `todo`

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: YAML frontmatter validation passes for all agents
    Tool: Bash
    Preconditions: All 6 agent files exist in .github/agents/
    Steps:
      1. Run: python -c "import yaml, glob, sys; errors=[]; [errors.append(f'{f}: invalid YAML') for f in glob.glob('.github/agents/*.agent.md') if not (lambda c: c.startswith('---') and yaml.safe_load(c.split('---')[1]))(open(f).read())]; [errors.append(f'{f}: missing description') for f in glob.glob('.github/agents/*.agent.md') if 'description' not in yaml.safe_load(open(f).read().split('---')[1])]; print('\n'.join(errors)) if errors else print('All agent files valid'); sys.exit(1 if errors else 0)"
      2. Assert exit code is 0
    Expected Result: "All agent files valid" printed, exit code 0
    Failure Indicators: Any line containing "invalid YAML" or "missing description"
    Evidence: .sisyphus/evidence/task-F1-yaml-validation.txt

  Scenario: Cross-reference integrity check passes
    Tool: Bash
    Preconditions: All 6 agent files exist
    Steps:
      1. Run the Python cross-reference script that collects all agent names, then checks handoffs[].agent and agents[] references
      2. Assert no "BROKEN:" lines in output
    Expected Result: "Cross-reference check complete" with no BROKEN lines
    Failure Indicators: Any line starting with "BROKEN:"
    Evidence: .sisyphus/evidence/task-F1-crossref-validation.txt

  Scenario: Git initial commit succeeds
    Tool: Bash
    Preconditions: All validation passes
    Steps:
      1. Run: git init && git add -A && git commit -m "feat: initial copilot multi-agent orchestration system"
      2. Run: git log --oneline
    Expected Result: Commit created, git log shows at least 1 commit
    Failure Indicators: Git error messages, empty log
    Evidence: .sisyphus/evidence/task-F1-git-commit.txt
  ```

  **Commit**: YES
  - Message: `feat: initial copilot multi-agent orchestration system`
  - Files: all `.github/agents/*.agent.md`, `.github/copilot-instructions.md`, `README.md`
  - Pre-commit: YAML validation + cross-reference validation

- [x] F2. **Plan Compliance Audit**

  **What to do**:
  - Read this plan end-to-end
  - For each "Must Have": verify implementation exists (read file, check content)
  - For each "Must NOT Have": search codebase for forbidden patterns — reject with file:line if found
  - Check evidence files exist in `.sisyphus/evidence/`
  - Compare deliverables against plan

  **Recommended Agent Profile**:
  - **Category**: `deep`
  - **Skills**: [`superpowers/verification-before-completion`]

  **Parallelization**:
  - **Can Run In Parallel**: YES (with F1, F3, F4)
  - **Parallel Group**: Wave FINAL
  - **Blocks**: None
  - **Blocked By**: Task 8

  **References**:
  - This plan file: `.sisyphus/plans/copilot-orchestration.md`
  - "Must Have" and "Must NOT Have" sections above

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: All Must Have items verified
    Tool: Bash
    Steps:
      1. Read each agent file, verify role separation matches plan
      2. Verify orchestrator has both handoffs and agents field
      3. Verify researcher and reviewer have read-only tools (no edit/execute)
      4. Verify implementer and tester have edit/execute tools
      5. Verify dynamic discovery instructions exist in tester and implementer
      6. Verify standalone instructions exist in each agent
      7. Verify description field in every agent
      8. Verify README has architecture diagram, usage guide, compatibility matrix
    Expected Result: All Must Have items present
    Evidence: .sisyphus/evidence/task-F2-compliance-audit.md

  Scenario: No Must NOT Have violations
    Tool: Bash
    Steps:
      1. Grep for "mcp-servers" in all agent files — must not appear
      2. Grep for "disable-model-invocation" — must not appear
      3. Grep for specific model names (e.g., "gpt-4", "claude", "opus") in frontmatter — must not appear
      4. Grep for TODO/FIXME/placeholder — must not appear in agent files
    Expected Result: Zero violations found
    Evidence: .sisyphus/evidence/task-F2-must-not-have.md
  ```

  **Commit**: NO

- [x] F3. **Scope Fidelity Check**

  **What to do**:
  - For each task: read "What to do" from plan, read actual file content
  - Verify 1:1 — everything in spec was built (no missing), nothing beyond spec was built (no creep)
  - Check "Must NOT do" compliance per task
  - Detect scope creep: agent files containing features not in the plan
  - Flag unaccounted files

  **Recommended Agent Profile**:
  - **Category**: `deep`
  - **Skills**: [`superpowers/verification-before-completion`]

  **Parallelization**:
  - **Can Run In Parallel**: YES (with F1, F2, F4)
  - **Parallel Group**: Wave FINAL
  - **Blocks**: None
  - **Blocked By**: Task 8

  **References**:
  - This plan file: `.sisyphus/plans/copilot-orchestration.md`
  - All TODO task descriptions

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: No scope creep detected
    Tool: Bash
    Steps:
      1. List all files in repo (excluding .sisyphus/ and .git/)
      2. Verify each file is accounted for in the plan deliverables
      3. Verify no agent file contains features not described in its task spec
    Expected Result: All files accounted for, no unexpected content
    Evidence: .sisyphus/evidence/task-F3-scope-fidelity.md
  ```

  **Commit**: NO

- [x] F4. **Agent Quality Review**

  **What to do**:
  - Read each agent file for instruction quality
  - Verify instructions are clear, specific, and actionable
  - Check for AI slop: vague language, excessive caveats, generic instructions
  - Verify consistent formatting and structure across all agents
  - Check standalone vs pipeline instructions are clearly separated
  - Verify dynamic discovery instructions are concrete (which files to check, what to look for)

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
  - **Skills**: [`superpowers/verification-before-completion`]

  **Parallelization**:
  - **Can Run In Parallel**: YES (with F1, F2, F3)
  - **Parallel Group**: Wave FINAL
  - **Blocks**: None
  - **Blocked By**: Task 8

  **References**:
  - All 6 agent files in `.github/agents/`
  - Real-world examples: microsoft/hve-core, dotnet/maui, foxminchan/BookWorm

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Agent instructions meet quality bar
    Tool: Bash
    Steps:
      1. Read each agent file body (below frontmatter)
      2. Check word count is between 200-5000 words (not too short/long)
      3. Verify no vague phrases like "as needed", "if appropriate", "consider", "you might want to"
      4. Verify each agent has concrete action verbs and specific procedures
      5. Verify formatting consistency across all agents
    Expected Result: All agents pass quality checks
    Evidence: .sisyphus/evidence/task-F4-quality-review.md
  ```

  **Commit**: NO

---

## Commit Strategy

- **F1**: `feat: initial copilot multi-agent orchestration system` — all files, after full validation

---

## Success Criteria

### Verification Commands
```bash
# YAML frontmatter validation
python -c "import yaml, glob; [yaml.safe_load(open(f).read().split('---')[1]) for f in glob.glob('.github/agents/*.agent.md')]; print('YAML OK')"

# Cross-reference check
python -c "
import yaml, glob
agents = {}
for f in glob.glob('.github/agents/*.agent.md'):
    data = yaml.safe_load(open(f).read().split('---')[1])
    agents[data.get('name', '')] = data
broken = []
for name, data in agents.items():
    for h in data.get('handoffs', []):
        if h.get('agent', '') not in agents:
            broken.append(f'{name} -> {h[\"agent\"]}')
    for a in data.get('agents', []):
        if a != '*' and a not in agents:
            broken.append(f'{name} -> {a}')
print('BROKEN: ' + str(broken) if broken else 'Cross-ref OK')
"

# File count
ls -la .github/agents/*.agent.md | wc -l  # Expected: 6

# Description field check
grep -l "^description:" .github/agents/*.agent.md | wc -l  # Expected: 6
```

### Final Checklist
- [x] All 6 agent files exist with valid YAML frontmatter
- [x] All agents have `description` field
- [x] All cross-references resolve (handoffs and subagents)
- [x] Orchestrator has both handoffs AND agents field (hybrid)
- [x] Researcher and Reviewer are read-only (no edit/execute tools)
- [x] Implementer and Tester have edit/execute tools
- [x] Dynamic technology discovery in Tester and Implementer
- [x] Each agent has standalone instructions for CLI compatibility
- [x] README.md with architecture, usage, and compatibility matrix
- [x] `.github/copilot-instructions.md` with repo-level context
- [x] Git repository initialized with initial commit
- [x] All "Must NOT Have" guardrails verified absent
