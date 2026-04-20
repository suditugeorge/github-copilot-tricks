# Project Guidelines

This is a curated collection of custom **GitHub Copilot agent modes** (`.agent.md` files), **reusable prompts** (`.prompt.md` files), and **skills** (`SKILL.md` packages) for VS Code. There is no application code—the deliverables are the agent/prompt/skill files themselves and their documentation.

The repository is compatible with the [Awesome Coding Assistants](https://github.com/jlacube/awesome-coding-assistants-vscode) VS Code extension, which auto-detects files under `.github/agents/`, `.github/prompts/`, and `.github/skills/`.

## Architecture

```
.github/
  agents/         # All .agent.md files live here
  prompts/        # All .prompt.md files live here
  skills/         # Each skill is a folder containing a SKILL.md file
  copilot-instructions.md
README.md         # Public-facing docs with install badges and usage
LICENSE           # MIT
```

- Each agent is a standalone `.agent.md` file in `.github/agents/`.
- Each prompt is a standalone `.prompt.md` file in `.github/prompts/`.
- Each skill is a `SKILL.md` file inside its own folder under `.github/skills/{skill-name}/`.
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
   https://aka.ms/awesome-copilot/install/agent?url=vscode%3Achat-agent%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fagents%2F{filename}
   ```
3. Update the **Project Structure** tree if the directory layout changes.
4. If agents form a workflow pair, document the interaction under **How It Works**.

When adding a new prompt, update `README.md`:

1. Add a row to the **Available Prompts** table with name, description, and install badges.
2. The install badge URLs follow the pattern:
   ```
   https://aka.ms/awesome-copilot/install/prompt?url=vscode%3Achat-prompt%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fprompts%2F{filename}
   ```

When adding a new skill, update `README.md`:

1. Add a row to the **Available Skills** table with name, description, and install badges.
2. The install badge URLs follow the pattern:
   ```
   https://aka.ms/awesome-copilot/install/skill?url=vscode%3Achat-skill%2Finstall%3Furl%3Dhttps%3A%2F%2Fraw.githubusercontent.com%2Fsuditugeorge%2Fgithub-copilot-tricks%2Fmain%2F.github%2Fskills%2F{skill-name}%2FSKILL.md
   ```
3. Update the **Project Structure** tree if the directory layout changes.

## Style

- Write in clear, direct English. Avoid filler prose.
- Prefer tables and numbered lists over long paragraphs.
- Keep agent system prompts self-contained—each file should work standalone when installed in a user's project.
