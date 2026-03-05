# Agent Refinement: 3 Surgical Improvements

## TL;DR

> **Quick Summary**: Three targeted edits to `orchestrator.agent.md` and `planner.agent.md` that add failure recovery, clarify context forwarding, and trim redundant discovery examples — improving completeness without compromising the project's simplicity-first philosophy.
> 
> **Deliverables**:
> - `orchestrator.agent.md` with failure recovery + context forwarding specificity
> - `planner.agent.md` with trimmed discovery examples (3→1 + generalization)
> 
> **Estimated Effort**: Quick
> **Parallel Execution**: YES - 2 waves
> **Critical Path**: Task 1 → Task 2 (parallel with Task 3) → Task 4

---

## Context

### Original Request
User requested 3 specific improvements identified during a comparative analysis of 6 orchestration repos:
1. Orchestrator lacks failure recovery strategy for subagent failures
2. Orchestrator's context forwarding to Planner (step 3) and Implementer (step 4) is vague
3. Planner has 3 redundant discovery examples where 1 + generalization suffices

### Interview Summary
**Key Discussions**:
- User values "simplicity-first" — changes must not bloat files
- All changes are surgical (5-10 lines each), not structural
- Current agents scored 87/100 on simplicity-completeness balance; goal is to push toward 92+

**Research Findings**:
- Compared with copilot-orchestra, Atlas, Orchestration, Craftsman, coordinated-agent-team, gem-team
- Metis review confirmed: only Autonomous Mode steps 3 and 4 actually need context enhancement (1, 2, 5, 6 are already specific)
- JavaScript is the most common ecosystem for Copilot users — best example to keep

### Metis Review
**Identified Gaps** (addressed):
- Which discovery example to keep? → JavaScript (most common Copilot ecosystem)
- Where to place failure recovery? → After line 67 (extends "after each stage" paragraph)
- Scope of context forwarding? → Autonomous Mode steps 3-4 only
- "Retry once" semantics? → Simple re-invocation with same context
- "Report" semantics? → Summarize failure and ask user how to proceed

---

## Work Objectives

### Core Objective
Improve the orchestrator's robustness and the planner's conciseness with minimal, surgical edits that preserve the project's simplicity-first design philosophy.

### Concrete Deliverables
- `.github/agents/orchestrator.agent.md` — failure recovery + context forwarding (net +5 lines max)
- `.github/agents/planner.agent.md` — trimmed examples (net -12 lines min)

### Definition of Done
- [ ] `orchestrator.agent.md` contains failure recovery guidance (max 3 lines)
- [ ] Steps 3 and 4 in Autonomous Mode have specific context forwarding items
- [ ] `planner.agent.md` has exactly 1 discovery example + 1 generalization line
- [ ] All section headers preserved in both files
- [ ] YAML frontmatter unchanged in both files
- [ ] Net project line count decreased

### Must Have
- Failure recovery: "retry once, then report to user" semantics
- Context forwarding: specific items for Planner (step 3) and Implementer (step 4)
- Discovery example reduction: JavaScript kept, Python/Go removed, generalization added

### Must NOT Have (Guardrails)
- No new sections, headings, or subsections in either file
- No changes to YAML frontmatter in either file
- No changes to steps 1, 2, 5, 6 in Autonomous Mode (already specific)
- No changes to Handoff Mode or CLI Standalone sections
- No changes to any file other than the two targets
- No new concepts: error codes, retry counts, failure taxonomies, logging
- No cross-file references ("see Researcher for Python examples")
- No changes to README.md or copilot-instructions.md

---

## Verification Strategy (MANDATORY)

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed. No exceptions.

### Test Decision
- **Infrastructure exists**: NO (this is a documentation/config repo, not code)
- **Automated tests**: None
- **Framework**: N/A

### QA Policy
Every task MUST include agent-executed QA scenarios.
Evidence saved to `.sisyphus/evidence/task-{N}-{scenario-slug}.{ext}`.

- **File edits**: Use Bash (grep, wc -l, git diff) — Count lines, verify headers, check content
- **Structural integrity**: Use grep on section headers and YAML markers

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (Start Immediately — independent file edits):
├── Task 1: Edit orchestrator.agent.md — failure recovery + context forwarding [quick]
├── Task 2: Edit planner.agent.md — trim discovery examples [quick]

Wave 2 (After Wave 1 — verification):
├── Task 3: Final verification — structural integrity + diff audit [quick]

Wave FINAL (After ALL tasks):
├── Task F1: Compliance audit [quick]
```

### Dependency Matrix

| Task | Depends On | Blocks | Wave |
|------|-----------|--------|------|
| 1 | — | 3, F1 | 1 |
| 2 | — | 3, F1 | 1 |
| 3 | 1, 2 | F1 | 2 |
| F1 | 3 | — | FINAL |

### Agent Dispatch Summary

- **Wave 1**: **2** — T1 → `quick`, T2 → `quick`
- **Wave 2**: **1** — T3 → `quick`
- **FINAL**: **1** — F1 → `quick`

---

## TODOs

- [x] 1. Edit orchestrator.agent.md — Add failure recovery + context forwarding

  **What to do**:
  - Open `.github/agents/orchestrator.agent.md`
  - **Change A — Context forwarding (steps 3 and 4)**: In the Autonomous Mode section, modify steps 3 and 4 to add specific context items using the existing parenthetical style:
    - Step 3 (line 63, starts with "Invoke `Planner`"): Change to specify forwarded context explicitly: `Invoke Planner with the synthesized context (problem statement, constraints, relevant files, recommended direction), discovery findings (detected tech stack, test framework, build commands), and scope boundaries; require a step-by-step implementation plan with acceptance criteria.`
    - Step 4 (line 64, starts with "Invoke `Implementer`"): Change to specify forwarded context explicitly: `Invoke Implementer with the approved plan, research context (relevant files, architectural patterns, existing conventions), and discovery findings; require minimal, focused changes.`
  - **Change B — Failure recovery**: After line 67 (the "After each stage, summarize outputs..." paragraph), add a new paragraph (2-3 lines):
    ```
    If a subagent returns incomplete or failed results, retry the stage once with the same context plus a summary of what went wrong. If the retry also fails, report the failure to the user with a summary of what was attempted and ask how to proceed.
    ```
  - Do NOT modify lines 1-59 (frontmatter, Intent Classification, Dynamic Discovery sections)
  - Do NOT modify steps 1, 2, 5, or 6 in Autonomous Mode (they are already specific)
  - Do NOT modify Handoff Mode or CLI Standalone sections

  **Must NOT do**:
  - Add new section headers or subsections
  - Touch YAML frontmatter (lines 1-27)
  - Introduce error codes, retry counts, or failure taxonomy concepts
  - Modify any other agent file

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Single-file edit, <10 lines changed, clear exact specifications
  - **Skills**: []
    - No special skills needed for straightforward text edits

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Task 2)
  - **Blocks**: Task 3, Task F1
  - **Blocked By**: None (can start immediately)

  **References** (CRITICAL - Be Exhaustive):

  **Pattern References** (existing code to follow):
  - `.github/agents/orchestrator.agent.md:60` — Step 1 example showing the parenthetical context-forwarding style: "with the user request, scope boundaries, discovery findings, and explicit research questions"
  - `.github/agents/orchestrator.agent.md:61` — Step 2 example showing synthesized context: "problem statement, constraints, relevant files, and recommended direction"
  - `.github/agents/orchestrator.agent.md:67` — The "After each stage" paragraph that the failure recovery text should follow

  **WHY Each Reference Matters**:
  - Line 60: This is the EXACT style to replicate for steps 3 and 4 — parenthetical lists within a single sentence
  - Line 61: Shows the vocabulary ("synthesized", "context packet") already established
  - Line 67: Shows where the failure recovery paragraph should be inserted (immediately after)

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Verify failure recovery text exists
    Tool: Bash (grep)
    Preconditions: orchestrator.agent.md has been edited
    Steps:
      1. Run: grep -c "retry" .github/agents/orchestrator.agent.md
      2. Assert output is exactly "1"
      3. Run: grep "retry" .github/agents/orchestrator.agent.md
      4. Assert output contains "retry the stage once" or similar
    Expected Result: Exactly 1 line mentioning retry, in the Autonomous Mode section
    Failure Indicators: grep returns 0 (missing) or >1 (over-inserted)
    Evidence: .sisyphus/evidence/task-1-failure-recovery.txt

  Scenario: Verify context forwarding in step 3
    Tool: Bash (grep)
    Preconditions: orchestrator.agent.md has been edited
    Steps:
      1. Run: grep "Planner" .github/agents/orchestrator.agent.md | grep -i "discovery"
      2. Assert output contains a line with both "Planner" and "discovery"
    Expected Result: Step 3 now references discovery findings explicitly
    Failure Indicators: grep returns empty (step 3 not updated)
    Evidence: .sisyphus/evidence/task-1-context-forwarding.txt

  Scenario: Verify file size within budget
    Tool: Bash (wc -l)
    Preconditions: orchestrator.agent.md has been edited
    Steps:
      1. Run: wc -l .github/agents/orchestrator.agent.md
      2. Assert line count is between 89 and 93 (was 88, max +5)
    Expected Result: File grew by 1-5 lines
    Failure Indicators: Line count >93 (too bloated) or ≤88 (nothing added)
    Evidence: .sisyphus/evidence/task-1-line-count.txt

  Scenario: Verify section headers preserved
    Tool: Bash (grep)
    Preconditions: orchestrator.agent.md has been edited
    Steps:
      1. Run: grep "^## " .github/agents/orchestrator.agent.md
      2. Assert output contains exactly: "## Intent Classification", "## Dynamic Discovery Before Delegation", "## Autonomous Mode (Subagent Delegation)", "## Handoff Mode (User-Controlled)", "## CLI Standalone Workflow"
    Expected Result: All 5 original section headers present, no new ones
    Failure Indicators: Missing header or extra header
    Evidence: .sisyphus/evidence/task-1-headers.txt

  Scenario: Verify YAML frontmatter untouched
    Tool: Bash (git diff)
    Preconditions: orchestrator.agent.md has been edited
    Steps:
      1. Run: git diff .github/agents/orchestrator.agent.md | head -40
      2. Assert no changes appear in lines 1-27 (the YAML frontmatter block)
    Expected Result: First change in diff is after line 27
    Failure Indicators: Diff shows changes in frontmatter lines
    Evidence: .sisyphus/evidence/task-1-frontmatter.txt
  ```

  **Commit**: YES (groups with Task 2)
  - Message: `docs(agents): improve orchestrator resilience and trim planner examples`
  - Files: `.github/agents/orchestrator.agent.md`
  - Pre-commit: `grep -c "^## " .github/agents/orchestrator.agent.md` (expect 5)

- [x] 2. Edit planner.agent.md — Trim discovery examples from 3 to 1

  **What to do**:
  - Open `.github/agents/planner.agent.md`
  - Locate the "Dynamic Discovery in Practice" section (line 179+)
  - **Keep**: The JavaScript/Node.js example (lines 181-189, "Example: Discovering a JavaScript Project")
  - **Remove**: The Python example (lines 191-198, "Example: Discovering a Python Project")
  - **Remove**: The Go example (lines 200-207, "Example: Discovering a Go Project")
  - **Add**: After the JavaScript example, add one generalization line:
    ```
    Apply this same discover → parse → adapt pattern for any technology ecosystem (Python, Go, Rust, Java, Ruby, etc.).
    ```
  - Do NOT modify lines 1-180 or lines 208+ (Technology-Agnostic Philosophy section and beyond)

  **Must NOT do**:
  - Add new section headers
  - Touch YAML frontmatter (lines 1-17)
  - Add cross-references to other agent files
  - Change the JavaScript example content
  - Modify the Technology-Agnostic Philosophy section below

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Single-file edit, deletion-heavy, clear exact specifications
  - **Skills**: []
    - No special skills needed

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 1 (with Task 1)
  - **Blocks**: Task 3, Task F1
  - **Blocked By**: None (can start immediately)

  **References** (CRITICAL - Be Exhaustive):

  **Pattern References** (existing code to follow):
  - `.github/agents/planner.agent.md:179-189` — The "Dynamic Discovery in Practice" heading and JavaScript example to KEEP
  - `.github/agents/planner.agent.md:191-207` — The Python and Go examples to REMOVE
  - `.github/agents/planner.agent.md:209-218` — The "Technology-Agnostic Philosophy" section that must NOT be touched (boundary marker)

  **WHY Each Reference Matters**:
  - Lines 179-189: The surviving example — implementer must verify it's unchanged
  - Lines 191-207: The deletion targets — implementer must remove these exactly
  - Lines 209+: Boundary — nothing beyond this point should change

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Verify example count reduced to 1
    Tool: Bash (grep)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: grep -c "Example: Discovering" .github/agents/planner.agent.md
      2. Assert output is exactly "1"
    Expected Result: Only 1 discovery example remains (JavaScript)
    Failure Indicators: Count is 0 (all removed) or >1 (not enough removed)
    Evidence: .sisyphus/evidence/task-2-example-count.txt

  Scenario: Verify JavaScript example preserved
    Tool: Bash (grep)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: grep "JavaScript Project" .github/agents/planner.agent.md
      2. Assert output contains "Discovering a JavaScript Project"
    Expected Result: JavaScript example header still present
    Failure Indicators: grep returns empty (JavaScript example was accidentally removed)
    Evidence: .sisyphus/evidence/task-2-js-example.txt

  Scenario: Verify Python and Go examples removed
    Tool: Bash (grep)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: grep -c "Python Project" .github/agents/planner.agent.md
      2. Assert output is "0"
      3. Run: grep -c "Go Project" .github/agents/planner.agent.md
      4. Assert output is "0"
    Expected Result: No Python or Go example headers
    Failure Indicators: Either grep returns >0
    Evidence: .sisyphus/evidence/task-2-removed-examples.txt

  Scenario: Verify generalization line exists
    Tool: Bash (grep)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: grep -i "any.*ecosystem\|any.*technology" .github/agents/planner.agent.md
      2. Assert output contains a generalization line
    Expected Result: A line generalizing the pattern to other ecosystems exists
    Failure Indicators: grep returns empty (generalization not added)
    Evidence: .sisyphus/evidence/task-2-generalization.txt

  Scenario: Verify file size decreased
    Tool: Bash (wc -l)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: wc -l .github/agents/planner.agent.md
      2. Assert line count is ≤255 (was 267, must decrease by at least 12)
    Expected Result: File is 250-255 lines
    Failure Indicators: Line count >255 (not enough removed) or <240 (too much removed)
    Evidence: .sisyphus/evidence/task-2-line-count.txt

  Scenario: Verify section headers preserved
    Tool: Bash (grep)
    Preconditions: planner.agent.md has been edited
    Steps:
      1. Run: grep "^## " .github/agents/planner.agent.md
      2. Assert all original ## headers are present in same order
    Expected Result: No headers removed or added
    Failure Indicators: Header count changed
    Evidence: .sisyphus/evidence/task-2-headers.txt
  ```

  **Commit**: YES (groups with Task 1)
  - Message: `docs(agents): improve orchestrator resilience and trim planner examples`
  - Files: `.github/agents/planner.agent.md`
  - Pre-commit: `grep -c "Example: Discovering" .github/agents/planner.agent.md` (expect 1)

- [x] 3. Final verification — Structural integrity and diff audit

  **What to do**:
  - Run the full verification suite across both modified files:
    1. `git diff --name-only` — verify exactly 2 files changed
    2. Verify all section headers preserved in both files (grep `^## `)
    3. Verify YAML frontmatter unchanged in both files (compare first 17/27 lines)
    4. Verify code fences properly closed (even count of ``` lines in each file)
    5. Verify net line counts: orchestrator ≤93, planner ≤255
    6. Read the new failure recovery paragraph — verify voice matches (imperative, concise)
    7. Read the modified steps 3-4 — verify parenthetical style matches steps 1, 2, 5, 6
    8. Read the generalization line — verify it's a single line, not a paragraph

  **Must NOT do**:
  - Make any file changes during verification
  - Touch any files

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: Read-only verification, no code changes
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 2 (sequential after Wave 1)
  - **Blocks**: Task F1
  - **Blocked By**: Task 1, Task 2

  **References**:
  - `.github/agents/orchestrator.agent.md` — full file, post-edit
  - `.github/agents/planner.agent.md` — full file, post-edit

  **Acceptance Criteria**:

  **QA Scenarios (MANDATORY):**

  ```
  Scenario: Verify exactly 2 files changed
    Tool: Bash (git diff)
    Preconditions: Both edits complete
    Steps:
      1. Run: git diff --name-only
      2. Assert output is exactly 2 lines: .github/agents/orchestrator.agent.md and .github/agents/planner.agent.md
    Expected Result: Exactly 2 files in diff
    Failure Indicators: More or fewer files listed
    Evidence: .sisyphus/evidence/task-3-file-count.txt

  Scenario: Verify net line reduction
    Tool: Bash (git diff --stat)
    Preconditions: Both edits complete
    Steps:
      1. Run: git diff --stat
      2. Assert total deletions > total insertions (net reduction)
    Expected Result: More lines removed than added across both files
    Failure Indicators: Net positive line change
    Evidence: .sisyphus/evidence/task-3-net-lines.txt

  Scenario: Verify code fences balanced in both files
    Tool: Bash (grep)
    Preconditions: Both edits complete
    Steps:
      1. For each file, count lines matching exactly "```"
      2. Assert count is even for both files
    Expected Result: All code fences properly opened and closed
    Failure Indicators: Odd count in either file
    Evidence: .sisyphus/evidence/task-3-code-fences.txt
  ```

  **Commit**: NO (verification only, no changes to commit)

---

## Final Verification Wave

- [x] F1. **Compliance Audit** — `quick`
  Read both modified files end-to-end. Verify: all "Must Have" items present, all "Must NOT Have" items absent, YAML frontmatter identical to originals, section headers unchanged, net line count decreased, prose voice matches existing style (imperative, concise, semicolon-separated lists). Report: `APPROVE/REJECT` with specific file:line citations for any issues.

---

## Commit Strategy

- **Single commit after all tasks**: `docs(agents): improve orchestrator resilience and trim planner examples` — `.github/agents/orchestrator.agent.md`, `.github/agents/planner.agent.md`

---

## Success Criteria

### Verification Commands
```bash
git diff --name-only           # Expected: exactly 2 files
wc -l .github/agents/orchestrator.agent.md  # Expected: ≤93 (was 88)
wc -l .github/agents/planner.agent.md       # Expected: ≤255 (was 267)
grep -c "retry" .github/agents/orchestrator.agent.md     # Expected: 1
grep -c "Example: Discovering" .github/agents/planner.agent.md  # Expected: 1
```

### Final Checklist
- [ ] All "Must Have" present
- [ ] All "Must NOT Have" absent
- [ ] orchestrator.agent.md: failure recovery + context forwarding added
- [ ] planner.agent.md: examples trimmed from 3 to 1 + generalization
- [ ] Net file size: decreased overall
