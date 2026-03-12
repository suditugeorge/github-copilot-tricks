# Project Guidelines

This is a curated collection of custom **GitHub Copilot agent modes** (`.agent.md` files) for VS Code. There is no application code—the deliverables are the agent files themselves and their documentation.

## Architecture

```
agents/           # All .agent.md files live here
README.md         # Public-facing docs with install badges and usage
LICENSE           # MIT
```

- Each agent is a standalone `.agent.md` file in `agents/`.
- Agents can reference each other via `handoffs` (see the React 19 plan → implementation pair).

## Agent File Conventions

Every `.agent.md` file must include YAML frontmatter with these fields:

| Field | Required | Purpose |
|-------|----------|---------|
| `name` | Yes | Display name in the Copilot mode picker |
| `description` | Yes | One-line summary of the agent's purpose |
| `tools` | Yes | Array of VS Code / Copilot tools the agent can access |
| `argument-hint` | No | Placeholder text shown in the chat input |
| `agents` | No | Subagents the agent can invoke (e.g., `Explore`) |
| `handoffs` | No | Buttons for routing to another agent after completion |

After the frontmatter, structure the body as:

1. **Identity & role** — one paragraph establishing expertise
2. **Critical rules** — non-negotiable constraints (e.g., "never write code" for planning agents)
3. **Workflow** — numbered phases with clear entry/exit criteria
4. **Domain knowledge** — reference material the agent uses for decisions

Use explicit rules, numbered steps, and tables to minimize ambiguity across model tiers (strong and lightweight models alike).

## README Conventions

When adding a new agent, update `README.md`:

1. Add a row to the **Available Agents** table with name, description, and install badges.
2. The install badge URLs follow the pattern:
   ```
   https://aka.ms/awesome-copilot/install/agent?url=vscode%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2Fagents%2F{filename}
   ```
3. Update the **Project Structure** tree if the directory layout changes.
4. If agents form a workflow pair, document the interaction under **How It Works**.

## Style

- Write in clear, direct English. Avoid filler prose.
- Prefer tables and numbered lists over long paragraphs.
- Keep agent system prompts self-contained—each file should work standalone when installed in a user's project.
