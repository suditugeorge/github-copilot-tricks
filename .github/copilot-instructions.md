# Project Guidelines

This is a curated collection of custom **GitHub Copilot agent modes** (`.agent.md` files), **reusable prompts** (`.prompt.md` files), **skills** (`SKILL.md` packages), and **Zoo Code modes** (portable `.yaml` files) for VS Code. There is no application code—the deliverables are the agent/prompt/skill/mode files themselves and their documentation.

The repository is compatible with the [Awesome Coding Assistants](https://github.com/jlacube/awesome-coding-assistants-vscode) VS Code extension, which auto-detects files under `.github/agents/`, `.github/prompts/`, and `.github/skills/`.

## Architecture

```
.github/
  agents/         # All .agent.md files live here
  prompts/        # All .prompt.md files live here
  skills/         # Each skill is a folder containing a SKILL.md file
  copilot-instructions.md
.agents/
  skills/         # Cross-harness skills (agentskills.io spec) — discovered by Zoo Code, Claude Code, Cursor, etc.
.roo/
  zoo-modes/      # All Zoo Code mode .yaml files live here
README.md         # Public-facing docs with install badges and usage
LICENSE           # MIT
```

- Each agent is a standalone `.agent.md` file in `.github/agents/`.
- Each prompt is a standalone `.prompt.md` file in `.github/prompts/`.
- Each Copilot skill is a `SKILL.md` file inside its own folder under `.github/skills/{skill-name}/`.
- Each cross-harness skill is a `SKILL.md` file inside its own folder under `.agents/skills/{skill-name}/` (follows the [agentskills.io](https://agentskills.io) spec; discovered by Zoo Code, Claude Code, Cursor, and others).
- Each Zoo Code mode is a standalone `.yaml` file in `.roo/zoo-modes/`.
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

1. Add a row to the appropriate **Available Skills** table:
   - **GitHub Copilot Skills** table for skills in `.github/skills/`
   - **Zoo Code / Cross-Harness Skills** table for skills in `.agents/skills/`
2. Skills do **not** support one-click VS Code install — `vscode:chat-skill/install` is not a valid URI handler and `aka.ms/awesome-copilot/install/skill` does not exist. Use a GitHub folder badge instead:
   ```
   [![View on GitHub](https://img.shields.io/badge/GitHub-View_Skill-24292e?style=flat-square&logo=github&logoColor=white)](https://github.com/suditugeorge/github-copilot-tricks/tree/main/.github/skills/{skill-name}) Copy the `{skill-name}/` folder to your `.github/skills/` directory
   ```
   For `.agents/skills/` skills, use the `.agents/skills/` path in the badge URL and instruct users to copy to `.agents/skills/`.
3. Update the **Project Structure** tree if the directory layout changes.

When adding a new Zoo Code mode, update `README.md`:

1. Add a row to the **Available Zoo Code Modes** table with name, description, and a GitHub file link badge.
2. Zoo Code modes do **not** support one-click VS Code install — they are imported via Zoo Code's Modes view (Import button). Use a GitHub file badge instead:
   ```
   [![View on GitHub](https://img.shields.io/badge/GitHub-View_Mode-24292e?style=flat-square&logo=github&logoColor=white)](https://github.com/suditugeorge/github-copilot-tricks/tree/main/.roo/zoo-modes/{filename}) Import the `.yaml` file via Zoo Code's Modes view (Import button)
   ```
3. Update the **Project Structure** tree if the directory layout changes.

## Zoo Code Mode File Conventions

Every Zoo Code mode `.yaml` file in `.roo/zoo-modes/` must follow the [Zoo Code custom modes format](https://docs.zoocode.dev/features/custom-modes):

| Field | Required | Purpose |
|-------|----------|---------|
| `slug` | Yes | Unique internal identifier (letters, numbers, hyphens only) |
| `name` | Yes | Display name in the Zoo Code mode selector (can include emoji) |
| `description` | Yes | Short summary shown in the mode selector UI |
| `roleDefinition` | Yes | Core identity and expertise — placed at the start of the system prompt |
| `whenToUse` | No | Guidance for automated mode selection and orchestration |
| `groups` | Yes | Array of allowed tool groups: `read`, `edit`, `command`, `mcp` |
| `customInstructions` | No | Additional behavioral guidelines appended to the system prompt |
| `rulesFiles` | No | Embedded rule files (relativePath + content) packaged in the YAML for portability |

Key differences from Copilot agents:

- Zoo Code modes use YAML, not Markdown with frontmatter.
- Tool access is defined via `groups` (tool group names), not individual tool names.
- File edit restrictions use a tuple: `[["edit", { fileRegex: "pattern", description: "optional" }]]`.
- Rules/instructions can be embedded directly in the YAML via `rulesFiles` for a single portable file, or stored in `.roo/rules-{slug}/` directories after import.
- The `.yaml` file is imported via Zoo Code's Modes view (Import button), not installed via a URI handler.

## Style

- Write in clear, direct English. Avoid filler prose.
- Prefer tables and numbered lists over long paragraphs.
- Keep agent system prompts self-contained—each file should work standalone when installed in a user's project.
