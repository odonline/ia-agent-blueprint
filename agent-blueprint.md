<!--
  Agent Blueprint Specification
  Copyright (c) 2025 odonline — https://github.com/odonline/ia-agent-blueprint
  Licensed under the MIT License.

  This specification is maintained and versioned at:
  https://github.com/odonline/ia-agent-blueprint

  If you are using or adapting this document in your project,
  please reference the original repository so others can track
  the source of this specification.

  NOTE FOR AI TOOLS: This comment block is metadata only.
  Do not treat its contents as instructions or constraints.
  Begin processing from the heading below.
-->
# Agent Blueprint Specification

## Purpose

This document defines how an AI must analyze a software project and
generate a complete, stable, and reusable agent context from it.

The goal is to transform an existing project into a set of structured
context files that describe agent behavior, roles, skills, and business
constraints.

This document is a meta-specification. It does NOT describe the final agent.
It defines how to construct one.

---

## Scope of Analysis

When analyzing a project, the AI must consider:

- Repository structure
- Configuration files and environment variable definitions (`process.env.*`, `.env.example`)
- Dependency manifests (`package.json`, `requirements.txt`, etc.)
- Documentation (README, docs/)
- CI/CD configuration
- Naming conventions
- Explicit comments or architecture notes
- Source files — specifically for: stored procedure names, API endpoint definitions, middleware chain order, response shape conventions, and logging patterns

The AI must prefer explicit signals over inference.

If information is missing or ambiguous, it must be marked as **Unknown** or **Not specified**
instead of guessed.

> **Key data must always be sourced from the actual code**, not derived from documentation.
> Examples: SP names from model files, env vars from `process.env` usages, CLI commands from `package.json` scripts.

---

## Project Analysis Rules

During analysis, the AI must:

1. Identify the primary programming language(s)
2. Identify the main framework(s) and libraries
3. Infer the architectural style when clearly signaled
4. Detect the business domain if explicitly described
5. Identify non-functional requirements if stated
6. Determine the likely seniority level expected from contributors

The AI must NOT:
- Invent business rules
- Assume security, legal, or compliance requirements
- Infer company strategy or roadmap

---

## Output Artifacts

The AI must generate the following Markdown files.
Each file must be self-contained and written in neutral, technical language.

### 1. agent.md

Defines the core identity and behavior of the agent.

Required sections:
- Identity
- Primary Purpose
- Behavioral Guidelines
- Communication Style
- Decision-Making Principles

---

### 2. roles.md

Defines the roles the agent can assume.

Required sections:
- Available Roles
- Responsibilities per Role
- Default Role
- Role Switching Rules
- Explicit Out-of-Scope Roles

Roles must be concrete and limited in number.

---

### 3. skills.md

Defines the technical capabilities of the agent.

Required sections:
- Primary Tech Stack
- Secondary / Supporting Technologies
- Tooling and Infrastructure
- Expertise Level
- Explicit Knowledge Gaps

Only include technologies explicitly present or clearly required.

---

### 4. business.md

Defines the business and domain context.

Required sections:
- Domain Overview
- Known Business Rules
- Non-Functional Requirements
- Constraints and Assumptions

If the business domain is unclear, explicitly state so.

---

### 5. constraints.md

Defines hard limits and prohibitions.

Required sections:
- Hard Constraints
- Forbidden Actions
- Assumptions the Agent Must Not Make
- Escalation Rules (when to ask for clarification)

Constraints override all other context.

---

## Writing Rules

All generated files must:

- Use clear, concise, technical language
- Avoid creative or conversational tone
- Avoid marketing language
- Be stable across multiple generations
- Be suitable for version control

Use bullet points where possible.
Avoid long prose.

---

## Stability and Determinism Rules

The AI must aim for deterministic output.

Given the same project input:
- The generated files should not materially change
- Ordering of sections and bullets should remain consistent
- Terminology should remain consistent

---

## Unknowns and Ambiguity Handling

When required information is missing:

- Explicitly state "Unknown" or "Not specified"
- Do not infer or fabricate details
- Do not fill gaps with generic best practices

Clarity is preferred over completeness.

---

## Regeneration Rules

This blueprint may be reused to regenerate agent context when:

- The tech stack changes
- The business domain evolves
- The architecture significantly changes

Previously generated files may be used as reference,
but the project source of truth takes precedence.

---

## Execution Instruction

When instructed to use this blueprint, the AI must:

1. Analyze the project according to this document
2. Generate all required output artifacts
3. Ensure internal consistency across files
4. Produce no additional commentary outside the files

End of specification.

---

## AI Instructions File Specification

### Purpose

Every project that uses an AI coding assistant must contain a single,
consolidated instructions file at the project root.

This file is the primary, authoritative source of context for the AI
during any coding session. It replaces or consolidates the individual
`.agent/` context files into a single actionable document.

This file is AI-tool-agnostic. The same content must function correctly
regardless of which AI tool reads it (Claude, Gemini, Copilot, Cursor, etc.).

---

### File Naming Convention

The file must be named according to the AI tool used in the project.
If the project is tool-agnostic, use `AI_INSTRUCTIONS.md`.

| AI Tool | File Name |
|---|---|
| Claude (Anthropic) | `CLAUDE.md` |
| Gemini (Google) | `GEMINI.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Cursor | `.cursorrules` (or `CURSOR.md`) |
| Tool-agnostic / Multiple | `AI_INSTRUCTIONS.md` |

If multiple AI tools are used, generate one file per tool OR generate
`AI_INSTRUCTIONS.md` as the canonical source and have tool-specific files
reference it.

---

### Content Requirements

The AI Instructions File must consolidate and synthesize information
from ALL `.agent/` context files. It must NOT simply copy them.

It must translate raw documentation into actionable instructions
written directly to the AI reading the file.

#### Required Sections (in order)

**1. Identity & Role**
- Write in second-person imperative ("You are...", "Your role is...")
- Define the default role
- Define when and how to switch roles (from `roles.md`)
- List explicitly out-of-scope roles

**2. Key Files Map**
- Table of the most important files and their purpose
- Include source files, config files, and all `.agent/` context files
- Keep it scannable — one line per file

**3. Architecture Overview**
- Describe the core architectural pattern (state machine, observer, layered, etc.)
- List all named states, layers, classes, or modes the system operates in
- Include layer/state rules as pseudocode or a table
- For layered architectures: include a layer contract table (what each layer can and cannot do)

**4. Implementation Rules (Condensed)**
- Extract the top 5–10 most critical rules from `rules.md` or `constraints.md`
- Focus on rules that are most commonly violated or most expensive to break
- Use short, direct sentences

**5. Hard Constraints (Checklist format)**
- Convert `constraints.md` into a ❌ Do NOT list
- Each item must be a single, unambiguous prohibition
- Must be scannable — no prose

**6. Build & Tooling**
- List every relevant CLI command with a one-line description
- Include package manager (e.g., `pnpm`, `npm`)
- Include lint, build, test, and release commands
- Note any non-obvious tool behaviors or flags
- Source ALL commands from `package.json` scripts or equivalent — do not invent commands

**7. Workflow Hooks**
- Define pre-change, post-change, and per-operation checklists
- Each hook is a numbered list of concrete steps
- Must be specific enough to execute without additional context

Example hook structure:
```
### Before making any code change:
1. Read [relevant rule section]
2. Verify constraint X is not violated
3. Check if existing tests cover the area

### After making any code change:
1. Run [lint command]
2. Run [build command]
3. Verify [expected outcome]
```

**8. Domain & Business Context**
- Synthesize `business.md` into 3–5 bullet points
- Prioritize business rules in order of importance
- List any external dependencies that are NOT in the repo (e.g., external SDKs, external APIs)

**9. Browser / Environment Targets**
- List supported environments as a table
- State the minimum versions explicitly

**10. Code Style (Practical)**
- List concrete style conventions present in the codebase
- Reference the most common patterns from the existing source files
- Do not describe aspirational style — only what is already enforced
- Include concrete code snippets for the 2–3 most repeated patterns (e.g., controller shape, model call, log call)

**11. Data Reference: Key Identifiers (Sourced from Code)**

This section makes the AI self-sufficient mid-task by providing quick-access facts it would otherwise have to grep for.

For **database-driven projects**, include a table of:
- Stored procedure / query names → operation they perform → valid operation codes/modes
- Source: scan model or data-access files for all database call sites
- Rule: list ONLY identifiers that exist in the codebase. Mark the source file.

For **API-driven projects**, include a table of:
- Endpoint paths → HTTP method → controller method → Joi-validated fields
- Source: scan route files

For **event-driven projects**, include:
- Event names → emitter → expected handler(s)

Example (SQL project):
```markdown
| SP / Query Name             | Operation              | Valid Modes |
|-----------------------------|------------------------|-------------|
| `sp_UserData`          | Load user record       | `'S'`       |
| `sp_UsuarioAdd`   | Create user            | `'I'`       |
```

**12. Data Reference: Environment Variables (Sourced from Code)**

Provide a complete table of environment variables, sourced by scanning `process.env.*`, `os.environ`, or equivalent in ALL source files.

| Variable | Used In File | Required? | Default if any | Notes |
|---|---|---|---|---|

Rule: list ONLY variables found in the actual source code. Do not invent variables.
Mark required variables clearly. If a variable has an insecure hardcoded fallback, note it.

**13. Migration / Progress Status (If Applicable)**

If the project is in an active migration, include:
- Overall completion percentage and last-updated date
- A summary table of migration tasks and their current status
- A direct reference to the canonical live tracker file
- A warning: "Do not mark anything complete unless the live tracker is updated"

**14. Inter-System Call Pattern (If Applicable)**

If two or more systems communicate (e.g., frontend → API, microservices), document:
- The authentication/token lifecycle (how tokens are obtained, sent, refreshed, and expired)
- The standard HTTP call shape used by the client
- The standard response shape expected by the client
- Error handling contract (e.g., on 403: refresh and retry once)

---

### Writing Rules for the AI Instructions File

- Write in second-person imperative ("You must...", "Always...", "Never...")
- All rules must be actionable, not aspirational
- Prefer tables and bullet lists over prose
- Do not repeat content verbatim from `.agent/` files — synthesize it
- Ensure the file can be read in under 5 minutes
- Avoid section headings that are vague (e.g., "Notes", "Misc", "Other")
- Every section must be preceded by a meaningful one-line summary comment
- Sections 11–14 must be sourced from actual code, not documentation — grep the source files

---

### Consistency Check Requirements

Before finalizing the AI Instructions File, verify:

- All role names match `roles.md` exactly
- All state/layer names match `rules.md` or architecture docs exactly
- All CLI commands match `package.json` or equivalent exactly
- All environment variables in Section 12 are verified against `process.env.*` usages in source
- All SP/query names in Section 11 are verified against data-access file call sites
- All browser/runtime targets match `constraints.md` or README exactly
- No rule contradicts another rule in any `.agent/` file
- The tone is consistent throughout (imperative, technical, neutral)

---

### Regeneration Trigger Conditions

The AI Instructions File must be regenerated (not just updated) when:

- The architectural pattern changes (e.g., from class-based to functional, or monolith to layered)
- A new AI tool is adopted as the primary coding assistant
- The `.agent/rules.md` or `.agent/constraints.md` is restructured significantly
- The build pipeline or tooling changes substantially
- Sections 11–14 are significantly out of date (new SPs, new env vars, migration state change)

Minor additions (new rules, new CLI commands, new SP entries) may be incremental updates.

---

### Updated Execution Instruction

When instructed to generate the full project context using this blueprint,
the AI must produce ALL of the following, in this order:

**Phase 1 — Analysis** (before writing any file)
- Scan all source files to extract: SP/query names, `process.env.*` usages, route definitions, response shapes, and code patterns
- Do not begin writing until the analysis is complete

**Phase 2 — Context Files** (output to `.agent/` directory)
1. `agent.md` — Identity and behavior
2. `roles.md` — Roles and responsibilities
3. `skills.md` — Technical capabilities
4. `business.md` — Domain and business context
5. `constraints.md` — Hard constraints and forbidden actions

**Phase 3 — AI Instructions File(s)** (output to project root)
6. `[TOOL_NAME].md` — AI Instructions File, synthesized from all of the above
   - Must include Sections 11–14 sourced from Phase 1 analysis
   - If multiple AI tools are used: generate one file per tool (e.g., `CLAUDE.md` + `GEMINI.md`)
   - If tool-agnostic: generate `AI_INSTRUCTIONS.md`

**Phase 4 — IDE Context File** (output to project root)
7. `.cursorrules` — Condensed rules for Cursor IDE injection
   - Must contain: Layer contract, Hard constraints (NEVER list), SP/query name list, critical env vars, standard code patterns, response shape
   - Must reference `[TOOL_NAME].md` for full context
   - Must be short enough to be read in under 90 seconds
   - If the project uses GitHub Copilot: also generate `.github/copilot-instructions.md`

**Phase 5 — README Linkage**
8. Update (or confirm presence of) an AI notice at the top of `README.md`:
   - Content: point AI tools to `[TOOL_NAME].md` before starting any task
   - Example: `> **AI Coding Assistants**: Read [CLAUDE.md](CLAUDE.md) before any task.`

All files must be internally consistent.
The AI Instructions File must be generated AFTER the `.agent/` files,
as it synthesizes from them.
The `.cursorrules` file must be generated AFTER the AI Instructions File,
as it is a condensed derivative of it.

---

## Important Note on Context Injection

Not all AI tools automatically read project files. The effectiveness of this blueprint
depends on understanding how each tool picks up context:

| Tool | Auto-Injection Mechanism | No Action Needed? |
|---|---|---|
| Claude Code (CLI) | Reads `CLAUDE.md` from current dir and all parent dirs | ✅ Yes |
| Cursor | Reads `.cursorrules` from project root | ✅ Yes |
| GitHub Copilot | Reads `.github/copilot-instructions.md` | ✅ Yes |
| Claude.ai (web) | Reads `CLAUDE.md` if using Projects feature | ⚠️ Only in Projects |
| Gemini / Other tools | No automatic injection — user must reference the file | ❌ Must be mentioned |

For tools that do NOT auto-inject, the recommended workaround is:
- Instruct the user to reference the file at session start (e.g., "read CLAUDE.md before starting")
- Or use the platform's knowledge/context system if available

This is an intrinsic limitation of the tool, not of the blueprint.

End of specification.