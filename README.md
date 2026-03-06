# Copilot Multi-Agent Orchestration System

A multi-agent system for GitHub Copilot. Six specialized agents coordinate code generation through a Research → Plan → Implement → Test → Review pipeline, with planning and execution split across separate conversations to prevent context bleed. The Orchestrator delegates each worker stage through subagents via the `agent` / `runSubagent` capability.

```text
User Request ──> Orchestrator (Claude Sonnet 4.6)
                      │
        ┌─────────────┼─────────────┐
        v             v             v
   Researcher    Planner       Implementer
   (Gemini 3.1   (GPT-5.2)    (GPT-5.3-Codex)
    Pro)              │             │
        └─────────────┘    ┌───────┤
                           v       v
                        Tester   Reviewer
                     (GPT-5.3   (GPT-5.2)
                      -Codex)
```

Each agent uses a model optimized for its role. Change the `model` field in `.github/agents/*.agent.md` to use a different model.

## Setup

1. Copy `.github/agents/` into your repository.
2. Restart VS Code.

## Usage

**Plan** — open Copilot Chat and type:

```
@Orchestrator <your request>
```

The Orchestrator delegates Research → Plan through subagents, saves the plan to `plans/`, and outputs:

```
@Orchestrator execute plan: <task-name>
```

**Execute** — start a new conversation and paste the command above. The Orchestrator delegates `runSubagent` calls so each plan step goes through Implementer → Reviewer, then runs Tester for final verification.

### Without subagent support

If your environment does not support subagent delegation but still exposes direct agent invocation, invoke agents manually in order:

`@Researcher` → `@Planner` → `@Implementer` → `@Tester` → `@Reviewer`

## Customization

Edit the `.agent.md` files in `.github/agents/` to adjust tools, prompts, or constraints for your team.

## License

[MIT](LICENSE)
