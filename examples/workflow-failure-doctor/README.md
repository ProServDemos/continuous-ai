# Workflow Failure Doctor Agentic Workflow

Investigates failed CI workflow runs to identify root causes, detect recurring patterns, and create actionable investigation issues.

Built for GitHub Agentic Workflows (GH-AW): Markdown workflow + frontmatter, compiled to GitHub Actions, and constrained by safe outputs.

Reference: https://github.github.com/gh-aw/setup/quick-start/

Source inspiration: https://github.com/github/gh-aw/blob/v0.45.5/.github/workflows/ci-doctor.md?plain=1

## Complexity
- **Medium**: Multi-phase diagnostic workflow with workflow log collection, historical pattern matching, duplicate detection, and structured issue reporting. No multi-agent dispatch, but requires careful prompt engineering and safe output design to ensure actionable and accurate diagnostics without overwhelming noise.

## Why This Is Valuable

- Speeds incident response for broken CI pipelines
- Produces consistent, actionable diagnostics instead of ad hoc failure notes
- Reduces noise by closing or linking duplicate investigation issues
- Builds a reusable knowledge base of recurring failure patterns
- Improves team velocity by recommending concrete remediation steps

## What It Does

On `workflow_run` completion for the `main` branch, the agent:

1. Checks the run conclusion and exits with `noop` when CI succeeded
2. Retrieves workflow, job, and failed-job log details
3. Analyzes logs for root cause signals and classifies failure type
4. Correlates with historical investigations and similar GitHub issues
5. Detects duplicates and closes older redundant issues when appropriate
6. Creates or updates an investigation issue with findings and recommended actions
7. Stores pattern data to improve future investigations

Failure gate:

- Workflow-level guard only proceeds when `github.event.workflow_run.conclusion == 'failure'`

Primary safe outputs:

- create-issue
- add-comment
- update-issue
- noop

Default issue behavior:

- Title prefix: `[CI Failure Doctor] `
- Labels: `cookie`
- Older related issues can be auto-closed when duplicates are confirmed

## What It Analyzes

- Failed jobs and logs from GitHub Actions
- Error signatures, stack traces, and timing/resource patterns
- Commit and PR context related to the failed run
- Existing issues for recurrence or duplication
- Cached investigation artifacts for historical comparison

## Customization
- Expand branch filters beyond `main` for monorepos or release branches
- Tune duplicate detection logic and closure policy for your team
- Adjust issue labels and title prefix to match triage conventions
- Customize report depth (brief triage vs. full forensic output)
- Add repository-specific troubleshooting checks for common CI failures
- Modify safe-output message templates to align with team tone and runbook links

## Operational Notes
- This workflow reads Actions metadata and logs, so `actions: read` is required
- It is designed to be diagnosis-first: no code changes are made directly
- Cache-backed memory is enabled to improve pattern recognition over time