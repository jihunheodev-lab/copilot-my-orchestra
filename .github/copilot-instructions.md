# Copilot Multi-Agent Orchestration System

This repository defines a GitHub Copilot multi-agent orchestration system. Agent definitions live in `.github/agents/*.agent.md`. Plans are saved to `plans/`.

## Agent Roster

| Agent | Role | Write |
|-------|------|-------|
| Orchestrator | Single entry point — classifies intent, coordinates pipeline, saves plans to `plans/` | No |
| Researcher | Gathers codebase and external context | No |
| Planner | Produces structured implementation plans | No |
| Implementer | Executes code changes | Yes |
| Tester | Generates and runs tests | Yes |
| Reviewer | Six-dimension quality review | No |

## Technology Detection Order

Before technology-specific decisions, agents check these files in precedence order:

`package.json` → `pyproject.toml` → `Makefile` → `go.mod` → `.ruby-version` / `Gemfile` → `pom.xml` / `build.gradle` → `Cargo.toml` → `docker-compose.yml` / `Dockerfile`
