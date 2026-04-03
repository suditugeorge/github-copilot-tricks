---
name: generate-pull-request-description
description: "Generates a PR title and description in Romanian with emoticons, focused on business impact."
agent: "agent"
model: "GPT-4.1"
tools:
  - vscode
  - execute
  - read
  - agent
  - search
  - github.vscode-pull-request-github/issue_fetch
  - github.vscode-pull-request-github/labels_fetch
  - github.vscode-pull-request-github/notification_fetch
  - github.vscode-pull-request-github/doSearch
  - github.vscode-pull-request-github/activePullRequest
  - github.vscode-pull-request-github/pullRequestStatusChecks
  - github.vscode-pull-request-github/openPullRequest
---

# Pull Request Title & Description Generator

Generate a PR title and description for PR number: `${input:pullRequestNumber:Enter the Pull Request number (e.g., 79)}`

You MUST produce the final output in a **single response**. Do NOT stop early, do NOT output intermediate checkpoints or file lists. Your only visible output is the final PR title and description.

---

## Analysis Steps (internal — do NOT output any of this)

### 1. Retrieve the PR

- If the PR number matches the active PR, use `activePullRequest`. Otherwise use `openPullRequest` or `issue_fetch`.
- Extract changed files and commit messages. **Skip files under `vendor/`.**

### 2. Read Changed Files

For each non-vendor file, read its **full content** from the workspace (not just the diff). Internally determine its architectural role, what changed, and the business meaning.

### 3. Cross-Cutting Analysis

Internally trace the end-to-end flow, identify business impact, motivation, behavioral delta, and any breaking changes. Do NOT output any of this.

---

## Output Format

### ✅ Titlu Pull Request

A single line in **Romanian** that:

- Opens with a relevant emoticon
- Captures the business essence of the change
- Does **not** mention file names, class names, or method names

### ✅ Descriere Pull Request

Write in **Romanian** with emoticons. Keep the description **concise and focused** — cover the essentials without padding. Target a length that a reviewer can read in under 2 minutes.

Structure using these sections in order:

#### 📝 Rezumat

2–4 sentences: What was the problem or need? Why was this change necessary? What is the business value? This replaces a long context section — be direct.

#### ✏️ Ce s-a schimbat

For each functional change, describe **what it does** and **what business rule it implements**. Describe behavior, NOT code structure. Do NOT mention file names or class names. Use a bulleted list if there are multiple changes.

#### ↔️ Comportament înainte și după

Concrete before/after comparisons:

- **Înainte:** [what the system did or did not do]
- **După:** [what the system does now]

One pair per meaningful behavioral change. Keep it factual.

#### ⚠️ Note pentru reviewer *(opțional — only if there are breaking changes, risks, or non-obvious decisions)*

Brief notes on breaking changes, edge cases, or important design decisions. Omit this section entirely if there is nothing noteworthy.

---

## Constraints

- 🇷🇴 **Romanian only** — English only for technical terms with no Romanian equivalent.
- 😀 **Emoticons** — in section headers and the title.
- 🚫 **No file/class names** — the reviewer sees these in the "Changes" tab.
- ✅ **Evidence-based** — every claim must come from the code or commits. Do not invent context.
- 🚫 **No intermediate output** — no file lists, checkpoints, or analysis. Only the final title and description.
- ✂️ **Concise** — be thorough but not verbose. Every sentence must add value.
