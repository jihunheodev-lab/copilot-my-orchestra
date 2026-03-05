# Copilot Multi-Agent Orchestration System

A sophisticated multi-agent system for GitHub Copilot, providing specialized agents designed for coordinated code generation and development workflows. This system uses task decomposition and intelligent coordination to handle complex engineering requirements.

## Features

- **Specialized Roles**: Six distinct agents covering the full development lifecycle from research to review.
- **Dynamic Technology Discovery**: Agents analyze project metadata (package.json, pyproject.toml, etc.) to adapt to any tech stack.
- **Pipeline Orchestration**: Built-in support for Research -> Plan -> Implement -> Test -> Review workflows.
- **Autonomous & Guided Modes**: Supports both automated subagent delegation and user-controlled handoff workflows.
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
1. Open the GitHub Copilot Chat.
2. Type `@Orchestrator` followed by your request (e.g., `@Orchestrator implement a new authentication middleware`).
3. Follow the guided handoffs or let it run autonomously if enabled.

### In CLI (Standalone)
If subagent invocation is not supported in your environment, follow this manual sequence:
1. Run `@Researcher` to gather context.
2. Pass research findings to `@Planner` for an execution plan.
3. Use `@Implementer` to apply the plan.
4. Verify with `@Tester` and `@Reviewer`.

## Agent Catalog

| Agent | Purpose | Primary Tools |
|-------|---------|---------------|
| **Orchestrator** | Coordinates the pipeline and classifies intent. | agent, search, read |
| **Researcher** | Gathers codebase context and documentation. | search, read, fetch, web |
| **Planner** | Creates detailed technical implementation plans. | read, search |
| **Implementer** | Executes code changes and refactorings. | edit, editFiles, read |
| **Tester** | Generates and runs tests for validation. | execute, read, search |
| **Reviewer** | Performs quality audits and final verification. | read, search, read/problems |

## Compatibility Matrix

| Environment | Orchestrator (@) | Subagent Delegation | Handoff Buttons |
|-------------|-------------------|---------------------|-----------------|
| VS Code Insiders | Supported | Supported | Supported |
| VS Code Stable | Supported | Partial | Supported |
| GitHub.com | Supported | Supported | Supported |
| Copilot CLI | Supported | Not Supported | Not Supported |

## Pipeline Workflow

The system defaults to a five-stage pipeline:
1. **Research**: Analysis of the existing implementation and dependencies.
2. **Plan**: Technical specification and step-by-step task breakdown.
3. **Implement**: Execution of the code changes in small, atomic steps.
4. **Test**: Verification via unit, integration, or regression tests.
5. **Review**: Final check for performance, security, and project conventions.

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
