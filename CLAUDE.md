# Agent Routing Rules — ~/GitHub Workspace

This file defines when and how to delegate tasks to specialized agents and skills. Prefer delegation over inline work whenever the task matches a described trigger — agents have deeper context and tools suited to their role.

---

## Quick Reference

| Task | Tool | Agent / Skill |
|---|---|---|
| Review code I just wrote | `Agent` | `feature-dev:code-reviewer` |
| Review a pull request end-to-end | `Skill` | `pr-review-toolkit:review-pr` |
| Hunt for silent failures / bad error handling | `Agent` | `pr-review-toolkit:silent-failure-hunter` |
| Check test coverage in a PR | `Agent` | `pr-review-toolkit:pr-test-analyzer` |
| Verify comment accuracy | `Agent` | `pr-review-toolkit:comment-analyzer` |
| Analyze type design in a PR | `Agent` | `pr-review-toolkit:type-design-analyzer` |
| Simplify recently written code | `Agent` | `code-simplifier:code-simplifier` |
| Search or explore the codebase (fast) | `Agent` | `Explore` |
| Multi-step research or unclear codebase search | `Agent` | `general-purpose` |
| Deep-dive an existing feature | `Agent` | `feature-dev:code-explorer` |
| Design architecture for a new feature | `Agent` | `feature-dev:code-architect` |
| Plan an implementation strategy | `Agent` | `Plan` |
| Create a UI component or page | `Skill` | `frontend-design:frontend-design` |
| Implement from a Figma design | `Skill` | `figma:implement-design` |
| Validate/create plugin structure | `Agent` | `plugin-dev:plugin-validator` |
| Create a new agent for a plugin | `Agent` | `plugin-dev:agent-creator` |
| Review a skill for quality | `Agent` | `plugin-dev:skill-reviewer` |
| Verify a TypeScript Agent SDK app | `Agent` | `agent-sdk-dev:agent-sdk-verifier-ts` |
| Verify a Python Agent SDK app | `Agent` | `agent-sdk-dev:agent-sdk-verifier-py` |
| Questions about Claude Code / Anthropic API | `Agent` | `claude-code-guide` |
| Audit or improve a CLAUDE.md file | `Skill` | `claude-md-management:claude-md-improver` |
| Set up or review Sentry integration | `Skill` | `/sentry-setup-*` (see Sentry section) |
| HuggingFace tasks (datasets, models, jobs) | `Agent` | `huggingface-skills:AGENTS` |
| Stripe integration or error lookup | `Skill` | `stripe:stripe-best-practices` |
| Jira / Confluence / Atlassian tasks | `Skill` | `atlassian:*` (see Atlassian section) |
| Commit staged changes | `Skill` | `commit-commands:commit` |
| Commit, push, and open a PR | `Skill` | `commit-commands:commit-push-pr` |
| Debug a bug or test failure | `Skill` | `superpowers:systematic-debugging` |
| Brainstorm before building anything new | `Skill` | `superpowers:brainstorming` |
| Write an implementation plan | `Skill` | `superpowers:writing-plans` |

---

## Detailed Routing Rules

### Code Review

- **After writing or modifying code** — always delegate to `feature-dev:code-reviewer` (Agent) before committing. It checks bugs, logic errors, security issues, and project conventions. Uses confidence-based filtering (≥80%) so only real issues surface.
- **After a major feature step is complete** — use `superpowers:code-reviewer` (Agent). Validates work against the original plan and coding standards.
- **When receiving code review feedback** — invoke the `superpowers:receiving-code-review` skill before implementing suggestions. Prevents blindly accepting incorrect feedback.

### Pull Request Workflows

- **Full PR review** — use the `pr-review-toolkit:review-pr` skill. This orchestrates multiple specialized sub-agents in parallel.
- **Error handling and silent failures** — delegate to `pr-review-toolkit:silent-failure-hunter` (Agent) any time a PR includes catch blocks, fallbacks, or error-handling logic.
- **Test coverage gaps** — delegate to `pr-review-toolkit:pr-test-analyzer` (Agent) after a PR is created or updated with new functionality.
- **Comment accuracy** — delegate to `pr-review-toolkit:comment-analyzer` (Agent) after generating large documentation comments or before finalizing PRs with comment changes.
- **Type design quality** — delegate to `pr-review-toolkit:type-design-analyzer` (Agent) when a PR introduces or refactors types/interfaces.
- **Code clarity** — delegate to `pr-review-toolkit:code-simplifier` (Agent) after completing a significant code chunk in a PR.

### Feature Development

- **Starting any new feature** — invoke `superpowers:brainstorming` skill first. Do NOT skip this step even for "simple" tasks.
- **Exploring an existing feature before building** — delegate to `feature-dev:code-explorer` (Agent). It traces execution paths, maps architecture layers, and documents dependencies.
- **Designing the implementation approach** — delegate to `feature-dev:code-architect` (Agent). It analyzes codebase patterns and produces a full implementation blueprint.
- **Guided end-to-end feature workflow** — use the `feature-dev:feature-dev` skill, which orchestrates explorer → architect → reviewer automatically.
- **Writing the implementation plan** — invoke `superpowers:writing-plans` skill after design is approved.
- **Executing the plan in isolated work** — invoke `superpowers:using-git-worktrees` skill before starting; use `superpowers:executing-plans` to step through the plan with review checkpoints.

### Codebase Exploration & Research

- **Finding files by pattern or keyword** — use `Explore` agent with `thoroughness: "quick"`. For broader exploration, use `"medium"` or `"very thorough"`.
- **Open-ended research, multi-step search, or unclear search** — delegate to `general-purpose` agent. Prefer this when a simple Glob/Grep won't find the right match in a few tries.
- **Deep architectural understanding of a specific feature** — delegate to `feature-dev:code-explorer` (Agent). More thorough than Explore; maps full execution paths and dependencies.

### Planning & Architecture

- **High-level implementation planning** — use `Plan` agent. Returns step-by-step plans, identifies critical files, and considers architectural trade-offs.
- **Feature-specific architecture within an existing codebase** — use `feature-dev:code-architect` (Agent). Integrates with existing patterns rather than designing from scratch.
- **Before any non-trivial task** — use `EnterPlanMode` to document the approach before touching code.

### Code Quality & Simplification

- **Simplify recently modified code** — delegate to `code-simplifier:code-simplifier` (Agent) after completing any logical chunk of implementation. Improves clarity while preserving functionality.
- **Refine code in a PR** — use `pr-review-toolkit:code-simplifier` (Agent) specifically for PR-scoped simplification.

### Frontend & UI

- **Building new UI components, pages, or applications** — invoke `frontend-design:frontend-design` skill. Generates production-grade, visually polished code.
- **Implementing from a Figma design** — invoke `figma:implement-design` skill with the Figma URL. Achieves 1:1 visual fidelity and adapts to project stack.
- **Setting up design system rules** — use `figma:create-design-system-rules` skill.
- **Connecting Figma components to code** — use `figma:code-connect-components` skill.

### Plugin & Agent Development

- **Creating a new plugin from scratch** — invoke `plugin-dev:create-plugin` skill. Guided end-to-end workflow with component design, implementation, and validation.
- **Creating a new agent for a plugin** — delegate to `plugin-dev:agent-creator` (Agent).
- **Creating a new skill** — use `plugin-dev:skill-development` skill.
- **Reviewing a skill for quality** — delegate to `plugin-dev:skill-reviewer` (Agent) proactively after creating or modifying a skill.
- **Validating plugin structure** — delegate to `plugin-dev:plugin-validator` (Agent) after creating or modifying plugin components.
- **Developing/modifying a hook** — use `plugin-dev:hook-development` skill.

### Agent SDK Development

- **Building apps with the Claude API or Anthropic SDK** — invoke `claude-developer-platform` skill when code imports `anthropic` or `@anthropic-ai/sdk`.
- **Creating a new Agent SDK application** — invoke `agent-sdk-dev:new-sdk-app` skill.
- **Verifying a TypeScript Agent SDK app** — delegate to `agent-sdk-dev:agent-sdk-verifier-ts` (Agent) after creating or modifying the app.
- **Verifying a Python Agent SDK app** — delegate to `agent-sdk-dev:agent-sdk-verifier-py` (Agent) after creating or modifying the app.

### Claude Code & API Questions

- **Any question about Claude Code features** (hooks, slash commands, MCP servers, settings, IDE integration, keyboard shortcuts) — delegate to `claude-code-guide` agent.
- **Any question about the Anthropic API or Claude Agent SDK** — delegate to `claude-code-guide` agent.
- **Customizing keyboard shortcuts** — invoke `keybindings-help` skill.
- **Recommending Claude Code automations for a project** — invoke `claude-code-setup:claude-automation-recommender` skill.

### CLAUDE.md Management

- **Auditing or improving CLAUDE.md files** — invoke `claude-md-management:claude-md-improver` skill. Scans all CLAUDE.md files, evaluates quality, and suggests improvements.
- **Updating CLAUDE.md with session learnings** — invoke `claude-md-management:revise-claude-md` skill at end of session.

### Git & Version Control

- **Creating a commit** — invoke `commit-commands:commit` skill.
- **Commit + push + open PR** — invoke `commit-commands:commit-push-pr` skill.
- **Clean up gone branches** — invoke `commit-commands:clean_gone` skill.
- **Working on isolated feature work** — invoke `superpowers:using-git-worktrees` skill before starting.
- **Finishing a development branch** — invoke `superpowers:finishing-a-development-branch` skill when implementation is complete.

### Sentry Integration

Invoke the appropriate skill based on what you're setting up:

| Task | Skill |
|---|---|
| Analyze Sentry comments on a GitHub PR | `sentry-code-review` |
| Set up AI agent monitoring (OpenAI, Anthropic, etc.) | `sentry-setup-ai-monitoring` |
| Set up structured logging | `sentry-setup-logging` |
| Set up performance tracing | `sentry-setup-tracing` |
| Set up metrics (counters, gauges, distributions) | `sentry-setup-metrics` |
| Set up Sentry in iOS/Swift app | `sentry-ios-swift-setup` |

### Stripe Integration

- **Building any Stripe integration** — invoke `stripe:stripe-best-practices` skill first.
- **Understanding a Stripe error code** — invoke `stripe:explain-error` skill.
- **Getting test card numbers** — invoke `stripe:test-cards` skill.

### Atlassian (Jira / Confluence)

| Task | Skill |
|---|---|
| Triage a bug report — check for duplicates in Jira | `atlassian:triage-issue` |
| Create Jira tasks from meeting notes | `atlassian:capture-tasks-from-meeting-notes` |
| Convert a Confluence spec into a Jira backlog | `atlassian:spec-to-backlog` |
| Generate a project status report | `atlassian:generate-status-report` |
| Search company knowledge base | `atlassian:search-company-knowledge` |

### HuggingFace

- **Any HuggingFace task** (datasets, model training, jobs, evaluation, Gradio, paper publishing) — delegate to `huggingface-skills:AGENTS` agent or invoke the matching `huggingface-skills:*` skill.

### Debugging

- **Any bug, test failure, or unexpected behavior** — invoke `superpowers:systematic-debugging` skill before proposing fixes. Never guess at the root cause.

### Parallel & Multi-Agent Work

- **Two or more independent tasks** — invoke `superpowers:dispatching-parallel-agents` skill to set up parallel execution.
- **Executing a plan with independent tasks** — invoke `superpowers:subagent-driven-development` skill.

### Before Claiming Work Complete

- **Always run `superpowers:verification-before-completion` skill** before stating that work is done, tests pass, or a fix is confirmed. Evidence before assertions.

---

## Agent Capability Index

All agents available via the `Agent` tool:

| Agent | Role |
|---|---|
| `general-purpose` | General research, multi-step searches, complex tasks across any domain |
| `Explore` | Fast codebase search by file pattern, keyword, or architecture question |
| `Plan` | Software architecture design; produces step-by-step implementation plans |
| `claude-code-guide` | Answers questions about Claude Code CLI, Anthropic API, and Agent SDK |
| `statusline-setup` | Configures Claude Code status line settings |
| `superpowers:code-reviewer` | Reviews completed major feature steps against plan and coding standards |
| `feature-dev:code-reviewer` | Reviews code for bugs, security vulnerabilities, and quality (confidence-filtered) |
| `feature-dev:code-explorer` | Deep analysis of existing features: execution paths, architecture, dependencies |
| `feature-dev:code-architect` | Designs feature architecture aligned with existing codebase patterns |
| `code-simplifier:code-simplifier` | Simplifies recently modified code for clarity and maintainability |
| `pr-review-toolkit:code-reviewer` | Expert PR review against project guidelines and best practices |
| `pr-review-toolkit:silent-failure-hunter` | Identifies silent failures, missing error propagation, and suppressed exceptions |
| `pr-review-toolkit:code-simplifier` | Simplifies code in PR context while preserving functionality |
| `pr-review-toolkit:pr-test-analyzer` | Analyzes test coverage quality and completeness in a PR |
| `pr-review-toolkit:comment-analyzer` | Verifies code comment accuracy, completeness, and long-term value |
| `pr-review-toolkit:type-design-analyzer` | Rates type design for encapsulation, invariant expression, and enforcement |
| `agent-sdk-dev:agent-sdk-verifier-ts` | Verifies TypeScript Agent SDK apps follow SDK best practices |
| `agent-sdk-dev:agent-sdk-verifier-py` | Verifies Python Agent SDK apps follow SDK best practices |
| `plugin-dev:agent-creator` | Generates agent configurations from user requirements |
| `plugin-dev:skill-reviewer` | Reviews skills for quality, description effectiveness, best practices |
| `plugin-dev:plugin-validator` | Validates plugin structure, metadata, and dependency compliance |
| `huggingface-skills:AGENTS` | HuggingFace Hub operations: datasets, models, jobs, Gradio, evaluation |
