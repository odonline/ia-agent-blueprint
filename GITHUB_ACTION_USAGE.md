# Using Agent Blueprint as a GitHub Action

There are **three ways** to automate the Agent Blueprint in your CI/CD pipeline.
Choose the one that best fits your setup.

---

## Option 1: Reusable Action (Recommended)

Use the published GitHub Action in your own workflow. This is the simplest way
to integrate Agent Blueprint into any repository.

### Quick Setup

1. Add `agent-blueprint.md` to your repository root
2. Set up your API key as a repository secret
3. Create the workflow file below

### Workflow Example

Create `.github/workflows/generate-agent-context.yml` in your repository:

```yaml
name: "Generate Agent Context"

on:
  # Run when agent-blueprint.md is added or changed
  push:
    branches: [main]
    paths:
      - "agent-blueprint.md"

  # Allow manual trigger
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate Agent Context Files
        uses: odonline/ia-agent-blueprint@v1
        with:
          ai_provider: "openai"           # or "anthropic"
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
          ai_instructions_file: "AI_INSTRUCTIONS.md"
          create_pr: "true"
```

### Available Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `ai_provider` | Yes | `openai` | `openai` or `anthropic` |
| `openai_api_key` | If OpenAI | — | Your OpenAI API key |
| `anthropic_api_key` | If Anthropic | — | Your Anthropic API key |
| `ai_instructions_file` | No | `AI_INSTRUCTIONS.md` | Output filename |
| `blueprint_path` | No | `agent-blueprint.md` | Path to blueprint |
| `model` | No | Auto | Model override (e.g., `gpt-4o`) |
| `create_pr` | No | `true` | Create PR or commit directly |

### Outputs

| Output | Description |
|---|---|
| `files_generated` | `true` if files were generated successfully |
| `pr_number` | PR number (if `create_pr` is `true`) |

### On First Commit Example

To run when the repository is first set up:

```yaml
name: "Initialize Agent Context"

on:
  push:
    branches: [main]

jobs:
  check-and-generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check if agent context exists
        id: check
        run: |
          if [ -d ".agent" ]; then
            echo "exists=true" >> $GITHUB_OUTPUT
          else
            echo "exists=false" >> $GITHUB_OUTPUT
          fi

      - name: Generate Agent Context
        if: steps.check.outputs.exists == 'false'
        uses: odonline/ia-agent-blueprint@v1
        with:
          ai_provider: "openai"
          openai_api_key: ${{ secrets.OPENAI_API_KEY }}
```

---

## Option 2: GitHub Copilot Agent

If your organization has **GitHub Copilot Enterprise or Business**, you can use
the Copilot coding agent directly. No API keys needed.

### How it works

1. A workflow creates a GitHub Issue with the blueprint prompt
2. The issue is assigned to `@copilot`
3. Copilot analyzes the project and creates a PR

### Setup

Copy `examples/workflows/agent-blueprint-copilot.yml` from this repository into your repo's `.github/workflows/` directory.

Then trigger it manually from the Actions tab, or create an issue manually:

```markdown
Title: 🤖 Generate Agent Context Files via Blueprint

Body:
Read the agent-blueprint.md file at the root of this repository.
Analyze this project following the blueprint's instructions and generate
all required output artifacts into a .agent/ directory and an
AI_INSTRUCTIONS.md file at the root.

Assignees: copilot
```

---

## Option 3: GitHub Agentic Workflows (Preview)

GitHub Agentic Workflows (launched February 2026, technical preview) allow you
to define AI agent tasks directly in Markdown.

### Setup

1. Install the GitHub CLI extension:
   ```bash
   gh extension install github/gh-aw
   ```

2. Copy `examples/workflows/agent-blueprint-agentic.md` from this repository into your repo's `.github/workflows/` directory

3. Compile the workflow:
   ```bash
   gh aw compile
   ```

4. Commit both the `.md` and the generated `.lock.yml` files

5. Set the `GITHUB_COPILOT_TOKEN` secret in your repository

### How it works

The Markdown file contains natural language instructions for the AI agent.
When triggered, the agent reads the instructions and executes them within
a sandboxed GitHub Actions environment. Write actions (like creating a PR)
go through the "safe outputs" review system.

---

## Comparison

| Feature | Reusable Action | Copilot Agent | Agentic Workflow |
|---|---|---|---|
| API Key Required | ✅ Yes (OpenAI/Anthropic) | ❌ No | ⚠️ Copilot Token |
| Copilot Plan Required | ❌ No | ✅ Enterprise/Business | ✅ Yes |
| Trigger Options | Push, Manual | Manual, Issue | Push, Manual |
| Customization | High | Medium | High |
| Availability | ✅ GA | ✅ GA | ⚠️ Preview |
| Output Format | PR or Direct Commit | PR | PR |
| Cost | LLM API usage | Copilot plan | Copilot plan |

---

## Secrets Setup

### OpenAI
1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Create a new API key
3. In your repo: Settings → Secrets → Actions → New repository secret
4. Name: `OPENAI_API_KEY`, Value: your API key

### Anthropic
1. Go to [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)
2. Create a new API key
3. In your repo: Settings → Secrets → Actions → New repository secret
4. Name: `ANTHROPIC_API_KEY`, Value: your API key

### Copilot Token (for Agentic Workflows)
1. Go to GitHub Settings → Developer Settings → Personal Access Tokens
2. Create a new token with `copilot` scope
3. In your repo: Settings → Secrets → Actions → New repository secret
4. Name: `GITHUB_COPILOT_TOKEN`, Value: your token
