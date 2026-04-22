# Workflow Additions Prototype

Prototype workflow that converts a feature request into a new **draft GH-AW workflow example** and opens a pull request with the proposed files.

## Complexity
- **Medium**: Requires issue analysis, constrained file creation, and safe pull request generation while keeping changes scoped to a dedicated example folder.

## Why This Is Valuable

- Speeds up creation of new workflow examples
- Standardizes frontmatter and structure across contributed examples
- Keeps proposals reviewable by opening draft pull requests instead of committing directly to the default branch

## What It Does

When a maintainer runs `/add-workflow` on an issue, the agent:

1. Reads the issue requirements
2. Drafts a new workflow example under `examples/`
3. Adds a matching README for the example
4. Opens a draft pull request with the generated files
5. Adds an issue update summarizing what was created

Primary safe outputs:

- create-pull-request
- update-issue

## Customization
- Change the slash command name if your team prefers another trigger
- Restrict generated paths further if you want all output in a single folder
- Tune PR title prefix and labels to match repository conventions
