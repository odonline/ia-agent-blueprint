---
# =============================================================================
# Agent Blueprint — Agentic Workflow (GitHub Agentic Workflows)
# =============================================================================
# This is a GitHub Agentic Workflow file. It defines the task in natural
# language and uses an AI coding agent to execute it.
#
# REQUIREMENTS:
#   - GitHub Agentic Workflows technical preview access
#   - gh CLI with gh-aw extension installed
#   - Repository secret: GITHUB_COPILOT_TOKEN (PAT with Copilot Requests scope)
#
# SETUP:
#   1. Install: gh extension install github/gh-aw
#   2. Place this file in .github/workflows/
#   3. Run: gh aw compile
#   4. Commit both this .md file and the generated .lock.yml
#
# =============================================================================

name: Agent Blueprint — Generate Context

on:
  workflow_dispatch:
  push:
    branches: [main, master]
    paths:
      - "agent-blueprint.md"

permissions:
  contents: read

agent:
  model: copilot
  secrets:
    - GITHUB_COPILOT_TOKEN

outputs:
  pull_request:
    safe: true
---

# Agent Blueprint — Context File Generation

You are an expert software architect. Your task is to analyze this repository
and generate a complete set of AI agent context files.

## Instructions

1. **Read the specification**: Open and carefully read `agent-blueprint.md` in the repository root.
   This is your primary instruction set — follow it exactly.

2. **Analyze the project**: Examine the full project structure:
   - Source code files and their organization
   - `package.json`, `tsconfig.json`, or equivalent dependency manifests
   - `README.md` and any documentation in `docs/`
   - Configuration files (`.eslintrc`, `.prettierrc`, CI/CD configs, etc.)
   - Architecture patterns visible in the code
   - Naming conventions used throughout

3. **Generate the context files**: Create the `.agent/` directory and generate these files:
   - `.agent/agent.md` — Identity and behavioral guidelines
   - `.agent/roles.md` — Roles and responsibilities
   - `.agent/skills.md` — Technical capabilities
   - `.agent/business.md` — Domain and business context
   - `.agent/constraints.md` — Hard constraints and forbidden actions

4. **Generate the consolidated instructions file**: After all `.agent/` files are complete,
   generate `AI_INSTRUCTIONS.md` at the project root. This file must:
   - Synthesize information from ALL `.agent/` files
   - Be written in second-person imperative ("You must...", "Never...", "Always...")
   - Follow the 10-section structure defined in the blueprint specification

## Rules

- Use clear, concise, technical language
- Mark unknowns explicitly as "Unknown" or "Not specified"
- Do NOT invent business rules or assume security/compliance requirements
- Prefer explicit signals from the code over inference
- Ensure internal consistency across all generated files
- The consolidated file must be the LAST file generated

## Output

Create a pull request with all generated files. The PR title should be:
"🤖 Agent Blueprint — Generated Context Files"
