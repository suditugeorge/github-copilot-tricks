<div align="center">

# GitHub Copilot Tricks

Curated collection of custom **GitHub Copilot agent modes** for VS Code — drop them into your project and supercharge your workflow.

[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[Available Agents](#available-agents) • [Quick Start](#quick-start) • [How It Works](#how-it-works)

</div>

## Available Agents

| Agent | Description | Install |
| --- | --- | --- |
| **React 19 Plan** | Strategic planning agent that researches your codebase, clarifies requirements, and creates detailed implementation plans using modern React 19 patterns. Hands off to the implementation agent when ready. | [Install](https://github.com/suditugeorge/github-copilot-tricks/blob/main/agents/react-19-plan.agent.md) |
| **Expert React Frontend Engineer** | World-class React 19.2 implementation agent with deep knowledge of modern hooks, Server Components, Actions, TypeScript, and performance optimization. | [Install](https://github.com/suditugeorge/github-copilot-tricks/blob/main/agents/expert-react-frontend-engineer.agent.md) |

## Quick Start

### One-click install

Click the **Install** link next to any agent above. GitHub will show an "Install in VS Code" button that adds the agent directly to your editor.

### Manual install

1. Copy the desired `.agent.md` file from the [`agents/`](agents/) folder
2. Place it in your project's `.github/` directory, or in `~/.github/` for global availability
3. Open GitHub Copilot Chat in VS Code and select the agent from the mode picker

> [!TIP]
> Place agents in `~/.github/agents/` to make them available across all your projects without copying files into each repo.

## How It Works

These are [custom agent modes](https://code.visualstudio.com/docs/copilot/chat/chat-modes) for GitHub Copilot in VS Code. Each `.agent.md` file defines:

- **Name & description** — shown in the Copilot mode picker
- **Tools** — which VS Code and Copilot tools the agent can access
- **System prompt** — domain expertise, workflow rules, and knowledge that guide the agent's behavior

### React 19 Workflow

The two React agents are designed to work together:

```text
React 19 Plan ──► researches codebase, clarifies requirements, creates plan
       │
       ├─ "Start Implementation" ──► Expert React Frontend Engineer (builds it)
       └─ "Open in Editor" ──► exports plan as a prompt file for refinement
```

The **Plan** agent never writes code — it focuses entirely on research and planning. Once you approve the plan, handoff buttons route directly to the **Expert React Frontend Engineer** for implementation.

### Model Compatibility

Both agents are designed to work across model tiers:

- **Stronger models** (Claude Opus, GPT-5) — leverage the full architecture decision guides and nuanced trade-off analysis
- **Lighter models** (Claude Sonnet, GPT-5 Mini) — follow the structured checklists, tables, and fill-in-the-blank templates mechanically

All instructions use explicit rules, numbered steps, and tabular formats to minimize ambiguity regardless of model capability.

## Project Structure

```text
agents/
├── react-19-plan.agent.md                  # Planning agent for React 19 projects
└── expert-react-frontend-engineer.agent.md # Implementation agent for React 19
```

## Resources

- [VS Code Custom Chat Modes](https://code.visualstudio.com/docs/copilot/chat/chat-modes)
- [awesome-copilot](https://github.com/github/awesome-copilot) — community collection of Copilot agents and prompts
- [React 19 Documentation](https://react.dev/blog/2024/12/05/react-19)
