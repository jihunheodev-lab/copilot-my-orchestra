# Copilot Multi-Agent Orchestration System

A multi-agent orchestration pack for GitHub Copilot. The Orchestrator delegates work through specialized subagents across an Explore → Plan → Implement → Test → Review pipeline, while planning and execution stay split across separate conversations to prevent context bleed.

For bug fixes and quick fixes, the Orchestrator can skip `Planner` and write the lightweight plan directly after Explore and clarification.

```text
User Request ──> Orchestrator (GPT-5.4)
                      │
              Planning conversation
                      │
               Explore (×N, parallel)
               (Haiku / Gemini Flash)
                         │
         Clarify at top level if needed
                         │
         Planner (GPT-5.4, optional)
                      │
         plans/<task-name>-plan.md
                      │
             New conversation starts
                      │
       @Orchestrator execute plan: ...
                      │
           Implementer (GPT-5.4)
                      │
              Tester (GPT-5.4)
                      │
              Reviewer (GPT-5.4)
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

The Orchestrator:
1. Classifies the request
2. Runs `Explore` subagents (in parallel for multi-area tasks)
3. Resolves clarifying questions at the top level when needed
4. For new features and refactorings, delegates to `Planner` which saves `plans/<task-name>-plan.md`
5. For bug fixes and quick fixes, writes the lightweight plan directly and skips `Planner`
6. Outputs:

```
@Orchestrator execute plan: <task-name>
```

**Execute** — start a new conversation and paste the command above. The Orchestrator resumes from the plan file and delegates Implement → Test → Review.

### Without subagent support

All worker agents are `user-invocable: false`. The automated workflow requires a VS Code Copilot environment with agent delegation enabled. If your environment exposes worker agents for direct invocation despite the flag, keep clarification in the top-level Orchestrator conversation, skip `Planner` for bug fixes and quick fixes, and carry outputs forward at each step.

## Customization

Edit the `.agent.md` files in `.github/agents/` to adjust tools, prompts, or constraints for your team.

## License

[MIT](LICENSE)
