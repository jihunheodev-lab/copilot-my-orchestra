# GitHub Copilot Instructions - Copilot Multi-Agent Orchestration System

## Project Description

This repository implements a **multi-agent orchestration system for GitHub Copilot**, providing specialized agents designed to work together in coordinated code generation and development workflows. The system enables sophisticated automation by decomposing complex tasks into focused, composable agent roles.

## Purpose

This project provides specialized agents for code generation pipelines. Each agent is designed with a specific role in the development lifecycle, enabling teams to orchestrate complex workflows through intelligent task decomposition and agent coordination.

## Design Philosophy: Technology-Agnostic Architecture

This system is built on a **technology-agnostic design philosophy**. Rather than hardcoding assumptions about the tech stack, agents dynamically discover the project's technologies by analyzing configuration and build files. This approach ensures the system remains adaptable across diverse tech stacks and frameworks.

## Core Agents and Roles

The multi-agent orchestration system includes six specialized agents:

1. **Orchestrator** - Analyzes incoming requests, decomposes complex tasks into subtasks, coordinates agent execution flow, and manages dependencies between agents
2. **Researcher** - Gathers context about codebases, dependencies, and technologies; performs exploratory analysis to inform other agents
3. **Planner** - Creates detailed implementation plans, defines technical specifications, and outlines step-by-step execution strategies
4. **Implementer** - Executes code changes, refactors existing code, and implements features based on plans from the Planner agent
5. **Tester** - Writes and executes tests, validates implementation correctness, and ensures code quality standards
6. **Reviewer** - Performs comprehensive code review, validates against specifications, and ensures adherence to project conventions

## Dynamic Technology Discovery Convention

Agents in this system use **dynamic discovery** to detect the project's technology stack rather than relying on hardcoded assumptions. Before making any technology-specific decisions, agents analyze project configuration files to understand the actual tech stack in use.

### Standard Technology Detection Files

Agents should analyze the following files (in order of precedence) to detect technologies:

- `package.json` - Node.js / JavaScript ecosystem
- `pyproject.toml` - Python projects
- `Makefile` - Build system and tool indicators
- `go.mod` - Go projects
- `.ruby-version` / `Gemfile` - Ruby projects
- `pom.xml` / `build.gradle` - Java/Kotlin projects
- `Cargo.toml` - Rust projects
- `docker-compose.yml` / `Dockerfile` - Containerization tech

### Agent Directive

**Before making any technology-specific decisions or recommendations**, agents MUST:
1. Analyze the root directory for technology detection files
2. Parse relevant configuration files to determine actual technologies in use
3. Verify assumptions about the tech stack before proceeding
4. Adapt responses and code suggestions to the detected technology context

This ensures agents provide accurate, context-aware guidance regardless of the underlying technology choices.
