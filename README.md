# Copilot Multi-Agent Orchestration System

A multi-agent system for GitHub Copilot providing specialized agents for coordinated code generation workflows. Tasks are decomposed into focused roles, and the pipeline runs across two isolated conversations to prevent context bleed between planning and execution.

## Features

- **Specialized Roles**: Six distinct agents covering the full development lifecycle.
- **Dynamic Technology Discovery**: Agents analyze project config files to adapt to any tech stack.
- **Planning-Execution Isolation**: Plans are written in one conversation and resumed via `@Orchestrator execute plan: <task-name>` in a fresh context.
- **Single Entry Point**: All tasks start with `@Orchestrator` — no manual agent switching required.
- **VS Code Native Integration**: Optimized for the latest GitHub Copilot agent capabilities.

## Architecture

```text
+----------------+       +-------------------+
|  User Request  |------>|   Orchestrator    |
+----------------+       +---------+---------+
                                   |
           +-----------------------+-----------------------+
           |                       |                       |
   +-------v-------+       +-------v-------+       +-------v-------+
   |  Researcher   |       |    Planner    |       |  Implementer  |
   | (Context/Code)|------>| (Step-by-step)|------>| (Code Change) |
   +---------------+       +---------------+       +-------+-------+
                                                           |
           +-----------------------+-----------------------+
           |                       |
   +-------v-------+       +-------v-------+
   |    Tester     |       |   Reviewer    |
   | (Validation)  |------>| (Final Audit) |
   +---------------+       +---------------+
```

## Installation

1. Create a `.github/agents/` directory in your repository.
2. Copy all `.agent.md` files from this repository into your `.github/agents/` folder.
3. Restart VS Code or refresh GitHub Copilot.

## Usage

### In VS Code

**Planning conversation:**
1. Open GitHub Copilot Chat.
2. Type `@Orchestrator` followed by your request.
3. The Orchestrator classifies intent, runs Research and Plan, then stops and outputs:
   ```
   @Orchestrator execute plan: <task-name>
   ```

**Execution conversation:**

4. Start a new conversation and enter the `execute plan:` command.
5. The Orchestrator loads the saved plan from `plans/<task-name>-plan.md` and runs Implement → Test → Review autonomously.

### In CLI (Standalone)
If subagent invocation is not supported in your environment:
1. Run `@Researcher` to gather context.
2. Pass research findings to `@Planner` for an execution plan.
3. Note the plan file path, start a new conversation, and run `@Orchestrator execute plan: <task-name>`.
4. Use `@Implementer`, then `@Tester`, then `@Reviewer` manually with relevant context.

## Agent Catalog

| Agent | Purpose | Primary Tools |
|-------|---------|---------------|
| **Orchestrator** | Classifies intent, coordinates pipeline, saves plan files. | agent, search, read, fetch, editFiles |
| **Researcher** | Gathers codebase context and external documentation. | search, read, fetch, web |
| **Planner** | Creates detailed technical implementation plans. | search, read, fetch |
| **Implementer** | Executes code changes following the plan. | editFiles, search, read, execute |
| **Tester** | Generates and runs tests for validation. | editFiles, search, read, execute |
| **Reviewer** | Performs six-dimension quality audit. | search, read |

## Compatibility Matrix

| Environment | Orchestrator (@) | Subagent Delegation |
|-------------|-------------------|---------------------|
| VS Code Insiders | Supported | Supported |
| VS Code Stable | Supported | Partial |
| GitHub.com | Supported | Not Supported |
| Copilot CLI | Supported | Not Supported |

## Pipeline Workflow

The pipeline runs across two conversations for context isolation:

**Planning conversation** (Research → Plan → Stop):
1. **Research**: Analysis of existing implementation and dependencies.
2. **Plan**: Technical specification saved to `plans/<task-name>-plan.md` with `Approval Status: pending`.
3. **Stop**: Orchestrator outputs the `execute plan:` instruction and halts.

**Execution conversation** (triggered by `@Orchestrator execute plan: <task-name>`):
4. **Implement**: Code changes following the saved plan; status set to `in_progress`.
5. **Test**: Verification via unit, integration, or regression tests.
6. **Review**: Final quality audit; Orchestrator marks plan `Approval Status: done`.

## Intent Classification

The Orchestrator selects the appropriate pipeline based on request type:

| Intent | Pipeline |
|--------|---------|
| New feature | Research → Plan → stop → `execute plan:` → Implement → Test → Review |
| Bug fix | Research → stop → `execute plan:` → Implement → Test → Review (no Planner) |
| Refactoring | Research → Plan → stop → `execute plan:` → Implement → Test → Review |
| Question / investigation | Research only — no plan file, no `execute plan:` |
| Quick fix | Research → stop → `execute plan:` → Implement → Test (no Planner) |

## Customization

You can customize agent behavior by editing the `.agent.md` files.
- **Modify Tools**: Update the `tools` list in the YAML frontmatter.
- **Adjust Prompts**: Change the instructions below the frontmatter to match your team's specific standards.
- **Add Constraints**: Add project-specific rules to the Orchestrator's intent classification section.

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Submit a Pull Request with a clear description of changes.

## License

MIT License
