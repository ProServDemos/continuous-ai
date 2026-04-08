# Continuous AI

Just as CI/CD transformed software development by automating integration and deployment, **Continuous AI** covers the ways in which AI can be used to automate and enhance collaboration workflows. It's a label for all uses of automated AI to support software collaboration. Any use of automated AI to support any software collaboration on any platform anywhere is Continuous AI.

The key difference between Continuous AI and individual AI productivity tools like Copilot is focus: Continuous AI is about **collaboration, not just individual productivity**. It automates the repetitive, collaborative, and auditable tasks that help projects scale, documentation, triage, code quality, fault analysis, without requiring a human in the loop for every step.

This repository contains example agentic workflows built with [GitHub Agentic Workflows (GH-AW)](https://githubnext.com/projects/agentic-workflows/) that you can use as templates or inspiration for your own Continuous AI agents.

> Learn more: [Continuous AI at GitHub Next](https://githubnext.com/projects/continuous-ai/) · [Agentic Workflows at GitHub Next](https://githubnext.com/projects/agentic-workflows/)

## What Are Agentic Workflows?

[Agentic Workflows](https://githubnext.com/projects/agentic-workflows/) are a form of natural language programming over GitHub. Instead of writing bespoke scripts that operate over GitHub using the GitHub API, you describe the desired behavior in plain language Markdown. This is compiled into an executable GitHub Actions workflow that runs using an agentic coding agent such as Claude Code or OpenAI Codex.

Key design principles:

- **GitHub-native**: repo-centric execution, team-visible logs, permissions, and security controls from GitHub Actions
- **Model and coding-agent independent**: workflows are portable across coding agents (Claude Code, Codex, etc.)
- **Security-first**: least-privilege tokens, allow-listed tools, explicit safe outputs, and human-reviewable artifacts
- **Shareable**: publish and consume reusable workflows and tool specifications

## Examples

| Workflow | Trigger | Complexity | Description |
|----------|---------|------------|-------------|
| [Auto-Triage](examples/auto-triage/) | `issues: opened, edited` | Low | Automatically labels new or edited issues to improve triage speed and backlog quality |
| [Bug Fix](examples/bug-fix/) | `issues: labeled (bug)` | Low | Attempts to reproduce and fix reported bugs, then opens a pull request with the proposed change |
| [Workflow Failure Doctor](examples/workflow-failure-doctor/) | `workflow_run: completed (failure)` | Medium | Investigates failed CI runs to identify root causes, detect recurring patterns, and create investigation issues |

### Auto-Triage

Classifies issues by content and applies labels (`bug`, `enhancement`, `documentation`, `question`, `needs-triage`) in a single action. Falls back to `needs-triage` when confidence is low.

### Bug Fix

When an issue is labeled `bug`, the agent analyzes the codebase for likely root causes, implements a high-confidence fix, and creates a pull request. Uncertain cases are routed back to humans.

### Workflow Failure Doctor

When a CI workflow fails on `main`, the agent retrieves job logs, analyzes root causes, correlates with historical failures, detects duplicates, and creates or updates an investigation issue with findings and recommended actions.

## Getting Started

### Prerequisites

- [GitHub CLI (`gh`)](https://cli.github.com/) installed
- [GH-AW CLI extension](https://github.com/github/gh-aw) installed

### Compile and Deploy a Workflow

1. Clone this repository
2. Compile a workflow Markdown file into a GitHub Actions YAML file:
   ```bash
   gh aw compile ./examples/auto-triage/auto-triage.md
   ```
3. Commit and push the generated workflow to your repository's `.github/workflows/` directory
4. The workflow will trigger based on its configured events

See the [GH-AW Quick Start](https://github.github.com/gh-aw/setup/quick-start/) for full setup details.

## Resources

- [Continuous AI — GitHub Next](https://githubnext.com/projects/continuous-ai/)
- [Agentic Workflows — GitHub Next](https://githubnext.com/projects/agentic-workflows/)
- [GH-AW CLI & Docs](https://github.com/github/gh-aw)
- [Example Agentic Workflows (githubnext/agentics)](https://github.com/githubnext/agentics)