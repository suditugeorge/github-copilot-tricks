# GitHub Copilot Tricks

Curated collection of custom **GitHub Copilot agent modes**, **reusable prompts**, and **skills** for VS Code — install them directly via badge links or browse the catalog with the [Awesome Coding Assistants](https://github.com/jlacube/awesome-coding-assistants-vscode) extension.

[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[Available Agents](#available-agents) • [Available Prompts](#available-prompts) • [Available Skills](#available-skills) • [Quick Start](#quick-start) • [How It Works](#how-it-works)

## Available Agents

| Agent | Description | Install |
| --- | --- | --- |
| **[React 19 Plan](.github/agents/react-19-plan.agent.md)** | Strategic planning agent that researches your codebase, clarifies requirements, and creates detailed implementation plans using modern React 19 patterns. Hands off to the implementation agent when ready. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/agent?url=vscode%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fagents%2Freact-19-plan.agent.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/agent?url=vscode-insiders%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fagents%2Freact-19-plan.agent.md) |
| **[Expert React Frontend Engineer](.github/agents/expert-react-frontend-engineer.agent.md)** | World-class React 19.2 implementation agent with deep knowledge of modern hooks, Server Components, Actions, TypeScript, and performance optimization. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/agent?url=vscode%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fagents%2Fexpert-react-frontend-engineer.agent.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/agent?url=vscode-insiders%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fagents%2Fexpert-react-frontend-engineer.agent.md) |

## Quick Start

### Option A: One-click install

Click the **VS Code Install** badge next to any agent or prompt above. It will open VS Code and install the file directly.

### Option B: Browse via Awesome Coding Assistants

Install the [Awesome Coding Assistants](https://marketplace.visualstudio.com/items?itemName=jlacube.awesome-coding-assistants) extension, then add this repository as a custom source in your VS Code settings:

```jsonc
// settings.json
"awesome-coding-assistants.sources": [
  {
    "url": "https://github.com/suditugeorge/github-copilot-tricks",
    "name": "GitHub Copilot Tricks",
    "branch": "main"
  }
]
```

Open the **Awesome Coding Assistants** view from the Activity Bar to browse, preview, and install agents and prompts from a catalog UI.

### Option C: Manual install

1. Copy the desired `.agent.md` file from the [`.github/agents/`](.github/agents/) folder (or `.prompt.md` from [`.github/prompts/`](.github/prompts/))
2. Place it in your project's `.github/agents/` (or `.github/prompts/`) directory, or in `~/.github/` for global availability
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

## Available Prompts

| Prompt | Description | Install |
| --- | --- | --- |
| **[PR Description Generator (RO)](.github/prompts/generate-pull-request-description.prompt.md)** | Generates a PR title and description in Romanian with emoticons, focused on business impact. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fgenerate-pull-request-description.prompt.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fgenerate-pull-request-description.prompt.md) |
| **[PR Description Generator (EN)](.github/prompts/generate-pull-request-description-en.prompt.md)** | Generates a PR title and description in English with emoticons, focused on business impact. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fgenerate-pull-request-description-en.prompt.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fgenerate-pull-request-description-en.prompt.md) |
| **[PR Code Review Pipeline](.github/prompts/pr-code-review.prompt.md)** | Multi-step code review pipeline for pull requests: runs a general review + security review and saves results to a `code-review/` folder. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fpr-code-review.prompt.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/prompt?url=vscode-insiders%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2Fpr-code-review.prompt.md) |

## Available Skills

| Skill | Description | Install |
| --- | --- | --- |
| **[Swagger / OpenAPI Documentation](.github/skills/swagger-documentation/SKILL.md)** | Guidelines and examples for writing OpenAPI/Swagger documentation — covers Resource schemas, FormRequest parameter mapping, middleware headers, and Laravel validator-to-OpenAPI type conversion. | [![Install in VS Code](https://img.shields.io/badge/VS_Code-Install-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/skill?url=vscode%3Achat-skill%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fskills%2Fswagger-documentation%2FSKILL.md) [![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://aka.ms/awesome-copilot/install/skill?url=vscode-insiders%3Achat-skill%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fskills%2Fswagger-documentation%2FSKILL.md) |

## Project Structure

```text
.github/
├── agents/
│   ├── react-19-plan.agent.md                       # Planning agent for React 19 projects
│   └── expert-react-frontend-engineer.agent.md      # Implementation agent for React 19
├── prompts/
│   ├── generate-pull-request-description.prompt.md   # PR title/description generator (Romanian)
│   ├── generate-pull-request-description-en.prompt.md # PR title/description generator (English)
│   └── pr-code-review.prompt.md                      # Multi-step PR code review pipeline
├── skills/
│   └── swagger-documentation/
│       └── SKILL.md                                  # Swagger/OpenAPI documentation skill
└── copilot-instructions.md                           # Project guidelines for Copilot
```

## Resources

- [VS Code Custom Chat Modes](https://code.visualstudio.com/docs/copilot/chat/chat-modes)
- [Awesome Coding Assistants](https://github.com/jlacube/awesome-coding-assistants-vscode) — VS Code extension to browse, install, and manage AI coding assistant customizations
- [awesome-copilot](https://github.com/github/awesome-copilot) — community collection of Copilot agents and prompts
- [React 19 Documentation](https://react.dev/blog/2024/12/05/react-19)
