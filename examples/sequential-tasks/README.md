# Sequential Tasks: Chaining Agentic Workflows

Demonstrates how multiple independent agentic workflows can be strung together into a continuous pipeline, where the output of one workflow automatically triggers the next.

Built for GitHub Agentic Workflows (GH-AW): Markdown workflow + frontmatter, compiled to GitHub Actions, and constrained by safe outputs.

Reference: https://github.github.com/gh-aw/setup/quick-start/

## Complexity
- **High**: Composes multiple agentic workflows into a sequential pipeline using label-based triggers and workflow dispatch. Requires understanding how safe outputs from one workflow become the activation events for the next. The individual workflows are simple, but the emergent chain introduces coordination concerns around trigger design, label semantics, and execution ordering.

## Why This Is Valuable

- Shows how to build complex automation from simple, single-purpose agents
- Each workflow remains independently testable and replaceable
- New stages can be added to the pipeline without modifying existing workflows
- Demonstrates Continuous AI as a system, not just isolated automations
- Mirrors real team workflows: triage → action → implementation → review

## The Core Idea

Instead of building one large, monolithic agent, you compose a pipeline from small, focused workflows. Each workflow does one job and produces a safe output (a label, an issue update, a dispatched workflow) that activates the next stage. The workflows don't know about each other — they're connected only through GitHub's event system.

This is the agentic equivalent of a Unix pipeline: small tools, loosely joined, each doing one thing well.

## How It Works

An issue arrives and flows through a chain of agents, each triggered by the previous agent's output:

```
Issue opened
  └─▶ Auto-Triage (classifies and labels the issue)
        ├─ labeled "bug"
        │    └─▶ Bug Fix (analyzes, patches, opens PR)
        │
        └─ labeled "enhancement"
             └─▶ Plan (creates implementation plan, dispatches workers)
                   ├─▶ dispatch: Architect
                   ├─▶ dispatch: Guardian
                   └─▶ dispatch: Swift
                         ↓         ↓         ↓
                      PR #1      PR #2      PR #3
```

### Stage 1: Auto-Triage (`issue-triage.md`)

**Trigger**: `issues: opened, edited`

The triage agent reads the issue content, classifies it, and applies labels (`bug`, `enhancement`, `documentation`, `question`, or `needs-triage`). This is the entry point to the pipeline — its label output determines which downstream workflow activates next.

### Stage 2a: Bug Fix (`bug-fix.md`)

**Trigger**: `issues: labeled (bug)`

When triage labels an issue as `bug`, this agent automatically activates. It analyzes the codebase for likely root causes, implements a fix when confidence is high, and opens a pull request. Uncertain cases are routed back to humans via issue comments.

### Stage 2b: Parallel Prototype (`plan.md` → `architect.md`, `guardian.md`, `swift.md`)

**Trigger**: `issues: labeled (enhancement)`

When triage labels an issue as `enhancement`, the planner agent activates. It reads the issue specification, creates a structured implementation plan, appends it to the issue body, and dispatches three worker workflows in parallel:

| Persona | Philosophy | Output |
|---|---|---|
| **The Architect** | Clean design, clear abstractions | PR with `[architect]` prefix |
| **The Guardian** | Security-first, defensive coding | PR with `[guardian]` prefix |
| **The Swift** | Speed, pragmatism, minimal complexity | PR with `[swift]` prefix |

## What Makes This Sequential

The key mechanism is **label-based chaining**. Each workflow's safe outputs are designed to produce events that downstream workflows listen for:

1. **Auto-Triage** uses `add-labels` as its safe output → produces `issues: labeled` events
2. **Bug Fix** listens for `issues: labeled (bug)` → produces `create-pull-request` and `update-issue`
3. **Plan** listens for `issues: labeled (enhancement)` → uses `dispatch-workflow` to trigger workers
4. **Workers** are triggered via `workflow_dispatch` → each produces `create-pull-request`

No workflow calls another directly. The connection is implicit through GitHub's event model, which means:
- Each stage can be developed, tested, and deployed independently
- New branches can be added to the pipeline by creating a workflow that listens for an existing label
- Stages can be replaced without touching any other workflow

## Files

| File | Role | Trigger |
|---|---|---|
| `issue-triage.md` | Classifies issues and applies labels | `issues: opened, edited` |
| `bug-fix.md` | Fixes bugs and opens PRs | `issues: labeled (bug)` |
| `plan.md` | Creates implementation plan, dispatches workers | `issues: labeled (enhancement)` |
| `architect.md` | Implements with clean design focus | `workflow_dispatch` from plan |
| `guardian.md` | Implements with security focus | `workflow_dispatch` from plan |
| `swift.md` | Implements with speed focus | `workflow_dispatch` from plan |

## Customization

- **Add new pipeline branches** by creating workflows that trigger on other triage labels (e.g., a `documentation` label could trigger a docs-generation agent)
- **Extend the chain** by having downstream agents apply additional labels that trigger further workflows
- **Swap individual stages** — replace the bug-fix agent with a more sophisticated one without changing triage or prototype workflows
- **Adjust triage rules** to control which issues flow into which pipeline branches
- **Add gates** by introducing human-approval labels that must be applied before certain stages activate
- **Connect to other examples** — the [Workflow Failure Doctor](../workflow-failure-doctor/) could be added to monitor the pipeline itself, investigating failures in any stage

## Operational Notes
- Each workflow runs with minimal permissions (`contents: read`, `issues: read`, `pull-requests: read`) and uses safe outputs for all write operations
- The triage agent uses a GitHub App for label operations, keeping bot-applied labels distinguishable from human labels
- Worker workflows are triggered via `dispatch-workflow` and run independently — a failure in one worker does not block the others
- Issue content is treated as untrusted input across all stages