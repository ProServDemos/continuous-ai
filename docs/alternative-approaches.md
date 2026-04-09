# Alternative Approaches
This repository contains examples of various agentic workflows built with GitHub Agentic Workflows (GH-AW). Each workflow is designed to solve a specific problem or automate a particular process, demonstrating the flexibility and power of agentic workflows in different contexts.

What happens when you are not using GitHub Actions? Or when your workflow requires capabilities beyond what GitHub Agentic Workflows provide? The answer is that you can still use agentic coding agents to build custom automation, but you will need to handle the orchestration, security, and integration aspects yourself. This can be done using a variety of tools and platforms, such as:
- **GitHub Copilot CLI**: Copilot CLI can be directly used in CI/CD pipelines.
- **GitHub Copilot SDK**: For programmatic access to Copilot's capabilities in your own applications or scripts.
- **Microsoft Agent Framework**: Need complex multi-agent orchestration with shared memory and tool use? Microsoft Agent Framework provides a robust platform for building and managing AI agents with advanced capabilities and you can use GitHub Copilot as a coding agent within that framework.

## Copilot CLI

[GitHub Copilot CLI](https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/) can run directly inside any CI/CD pipeline — GitHub Actions, GitLab CI, Jenkins, or anything else that can execute a shell command. Unlike GH-AW, there is no Markdown-to-YAML compilation step. You write the prompt inline and call the CLI, which makes it a good fit for one-off automation tasks or environments where GitHub Actions is not available.

### When to Use It

- You need AI analysis in a non-GitHub-Actions CI system (Jenkins, GitLab CI, CircleCI)
- You want to add a single AI-powered step to an existing pipeline without adopting a full workflow framework
- You need fine-grained control over model selection, tool permissions, and prompt content per step

### Example: PR Review Step in GitHub Actions

```yaml
name: Copilot Review
on:
  pull_request:
    branches: [main]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install Copilot CLI
        run: |
          curl -fsSL https://gh.io/copilot-install | bash
          echo "$HOME/.local/bin" >> $GITHUB_PATH

      - name: Review changed files
        env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.COPILOT_TOKEN }}
        run: |
          CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)
          copilot --prompt "Review these files for security issues and bugs: $CHANGED" \
            --model gpt-4.1 \
            --allow-tool "read_file" \
            --deny-tool "run_terminal_cmd"
```

### Tradeoffs vs GH-AW

| | GH-AW | Copilot CLI |
|---|---|---|
| Authoring | Natural language Markdown compiled to Actions YAML | Shell commands with inline prompts |
| Portability | GitHub Actions only | Any CI/CD system |
| Safe outputs | Declarative, constrained by frontmatter | Manual — you control what the CLI can do via `--allow-tool` / `--deny-tool` |
| Multi-step orchestration | Built in (triggers, dispatch, parallel agents) | Roll your own with shell scripting or pipeline stages |

## Copilot SDK

The [GitHub Copilot SDK](https://github.com/github/copilot-sdk) (`@github/copilot-sdk` on npm, also available for Python, Go, and .NET) gives you programmatic access to Copilot's agentic capabilities. Use it when the automation you need lives inside your own application code rather than a CI/CD pipeline.

### When to Use It

- You are building a custom tool, platform, or service that needs embedded AI coding capabilities
- You need multi-turn agent sessions with fine-grained control over tool invocation and streaming
- You want to integrate Copilot into a larger application alongside other services and business logic

### Example: Node.js

```bash
npm install @github/copilot-sdk
```

```js
import { CopilotClient } from "@github/copilot-sdk";

const client = new CopilotClient();
await client.start();

const session = await client.createSession({ model: "gpt-5" });
const response = await session.send({
  prompt: "Analyze this log file for error patterns and suggest fixes.",
});
console.log(response.text);

await client.stop();
```

The SDK communicates with the Copilot CLI over JSON-RPC (the CLI is bundled and managed automatically). It supports streaming responses, custom tool definitions, file and URL operations, and multi-turn conversations — everything the CLI can do, but wrapped in a typed API you can call from code.

### Tradeoffs vs GH-AW

| | GH-AW | Copilot SDK |
|---|---|---|
| Runtime | GitHub Actions | Your own process (server, Lambda, container, etc.) |
| Language | Natural language Markdown | TypeScript, Python, Go, .NET |
| Orchestration | Declarative workflow triggers and dispatch | Imperative — you write the control flow |
| Best for | Repo-centric automation tied to GitHub events | Custom applications and platform integrations |

## Microsoft Agent Framework

The [Microsoft Agent Framework](https://github.com/microsoft/agent-framework) is an open-source SDK (for .NET and Python) for building and orchestrating multi-agent systems. It provides graph-based workflow orchestration, pluggable memory, middleware, and support for multiple model providers — including GitHub Copilot via the Copilot SDK.

### When to Use It

- You need multiple specialized agents coordinating on a single task (research, analysis, implementation, review)
- You want deterministic workflow control (branching, checkpointing, human-in-the-loop) alongside LLM-driven decisions
- You are building enterprise-grade agent systems that require observability, compliance, and long-term support

### How Copilot Fits In

The Copilot SDK acts as an agent provider within the Agent Framework. Copilot-powered agents implement the same interfaces (`AIAgent` in .NET, `BaseAgent` in Python) as agents backed by Azure OpenAI, Anthropic, or other providers, which means they can participate in multi-agent workflows side by side:

```csharp
var architect = new CopilotAgent("architect", "Design a clean API for the feature");
var guardian  = new CopilotAgent("guardian", "Review the design for security issues");

var workflow = new SequentialWorkflow(architect, guardian);
var result = await workflow.RunAsync(context);
```

### Tradeoffs vs GH-AW

| | GH-AW | Microsoft Agent Framework |
|---|---|---|
| Orchestration | GitHub Actions event-driven dispatch | Graph-based workflows with branching, loops, and handoffs |
| Agents | Single coding agent per workflow step | Multiple agents with shared memory and tool access |
| Runtime | GitHub-hosted runners | Your own infrastructure (containers, cloud, on-prem) |
| Complexity | Low — Markdown + frontmatter | Higher — requires application code and infrastructure |
| Best for | Repo-centric, event-driven automation | Complex multi-agent orchestration and enterprise systems |