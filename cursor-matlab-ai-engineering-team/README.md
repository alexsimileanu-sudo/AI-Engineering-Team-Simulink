# Cursor MATLAB/Simulink AI Engineering Team

A reusable AI agent team framework for MATLAB/Simulink automotive embedded development in Cursor IDE.  
It provides role-based agents, strict engineering boundaries, anti-hallucination rules, workflows, checklists, and templates for daily ECU development.

## Target Environment

- Cursor IDE
- MATLAB Agentic Toolkit through MCP
- MATLAB/Simulink
- Stateflow
- Embedded Coder
- AUTOSAR Classic
- MIL/SIL testing
- Automotive ECU software development

## Repository Contents

- `AGENTS.md` — team operating model
- `.cursor/rules/*.mdc` — Cursor Project Rules per agent (persona/boundaries, auto-attached by description; carry no model binding)
- `.cursor/agents/*.md` — Cursor Subagents per agent (same persona, plus the actual per-role model binding — this is the only Cursor mechanism that supports `model:`)
- `workflows/*.md` — operational procedures
- `checklists/*.md` — practical engineering checklists
- `templates/*.md` — reusable reporting/planning templates

## Install Into a Real Project

1. Copy `AGENTS.md` into your project root.
2. Copy `.cursor/rules/` into your project root at:
   - `<project-root>/.cursor/rules/`
3. Copy `.cursor/agents/` into your project root at:
   - `<project-root>/.cursor/agents/`
4. Copy `workflows/`, `checklists/`, and `templates/` into your project root (recommended).

### Minimal install

- Required:
  - `AGENTS.md`
  - `.cursor/rules/*.mdc`
  - `.cursor/agents/*.md` (required for model binding to actually take effect)
- Optional but recommended:
  - `workflows/`
  - `checklists/`
  - `templates/`

## Where to Copy AGENTS.md

Copy to:
- `<your-matlab-project-root>/AGENTS.md`

## Where to Copy .cursor/rules and .cursor/agents

Copy to:
- `<your-matlab-project-root>/.cursor/rules/`
- `<your-matlab-project-root>/.cursor/agents/`

## How to Use Each Agent in Cursor Chat

Use explicit role invocation in prompts:

- **Project Lead**: planning only
- **MATLAB Expert**: investigation only
- **Requirements Engineer**: requirements analysis only
- **Simulink Developer**: Simulink implementation only
- **Stateflow Specialist**: Stateflow implementation only
- **Test Engineer**: test artifacts only
- **Code Reviewer**: strict review and challenge assumptions
- **Documentation Agent**: commit/Jira/reports/docs only

## Example Prompts by Agent

### Project Lead
“Project Lead: Create a feature execution plan for torque arbitration enhancement. Split tasks by agent, identify risks, dependencies, and validation gates. Do not implement.”

### MATLAB Expert
“MATLAB Expert: Investigate current signal flow for torque request from input interfaces through arbitration outputs. Report exact model paths, data dictionaries, bus objects, and callback dependencies. No edits.”

### Requirements Engineer
“Requirements Engineer: Analyze requirement set REQ-TRQ-001..010. Extract acceptance criteria, ambiguity, and testable conditions. No design assumptions.”

### Simulink Developer
“Simulink Developer: Implement the approved Simulink-only changes from plan section 4 using existing interfaces only. Do not modify Stateflow charts.”

### Stateflow Specialist
“Stateflow Specialist: Implement approved state transition updates in `ModeManager` chart according to requirement delta RCR-22. Do not modify non-Stateflow logic.”

### Test Engineer
“Test Engineer: Create/modify MIL and SIL test artifacts for acceptance criteria AC1..AC6. Do not change production model logic.”

### Code Reviewer
“Code Reviewer: Perform strict review for safety, interface integrity, requirement traceability, regression risk, and validation evidence. Challenge unsupported assumptions.”

### Documentation Agent
“Documentation Agent: Generate commit message, Jira update, and validation report using only actual changed files and actual test results.”

## Recommended Workflow: Feature Development

1. User request
2. Project Lead plan
3. Requirements Engineer analysis
4. MATLAB Expert investigation
5. Simulink Developer / Stateflow Specialist implementation
6. Test Engineer validation
7. Code Reviewer review
8. Documentation Agent final docs

## Recommended Workflow: Bug Fixing

1. User reports issue + failure evidence
2. Project Lead triage plan
3. Requirements Engineer impact statement (if requirement affected)
4. MATLAB Expert root-cause investigation
5. Targeted fix by Simulink Developer/Stateflow Specialist
6. Regression MIL/SIL by Test Engineer
7. Code Reviewer gate
8. Documentation Agent summary and release/Jira note

## Recommended Workflow: Requirement Change

1. Requirement delta received
2. Project Lead impact planning
3. Requirements Engineer decomposition to acceptance criteria
4. MATLAB Expert baseline investigation
5. Focused implementation by Simulink/Stateflow agents
6. Test Engineer updates validation matrix and tests
7. Code Reviewer verifies traceability and impact containment
8. Documentation Agent publishes validation and compliance narrative

## Recommended Model Usage

Model binding is set in each `.cursor/agents/*.md` file's `model:` frontmatter field (Cursor Project Rules in `.cursor/rules/*.mdc` have no model field — they only carry persona/boundaries text). Invoke a role by its exact subagent name (`/project-lead`, `/matlab-expert`, etc.) to guarantee the bound model is used; natural-language or automatic delegation is not guaranteed to pick the intended subagent.

- **Claude Sonnet 5** (`claude-sonnet-5`):
  - Project Lead
  - MATLAB Expert
  - Simulink Developer
  - Stateflow Specialist
  - Test Engineer (default)
- **Gemini 3.5 Flash** (`gemini-3.5-flash`):
  - Requirements Engineer
  - Documentation Agent
- **GPT-5.5** (`gpt-5.5`):
  - Code Reviewer
- **Claude Opus 4.8** (`claude-opus-4-8`), on-demand escalation only:
  - `test-engineer-gap-analysis` — invoke explicitly when Test Engineer coverage is ambiguous or contested; not used automatically.

Exact model ID strings should be double-checked against Cursor's own model picker/autocomplete in your account, since Cursor's published docs mix dot- and dash-separated slugs across model families (e.g. `gpt-5.5` vs `claude-opus-4-8`) and don't publish a single canonical list.

## Engineering Safety Principles (Always On)

- Never invent signals, bus objects, parameters, requirements, or test outcomes.
- Always inspect before editing.
- Never modify unrelated models/artifacts.
- Preserve interfaces and architecture unless explicitly approved.
- Always report assumptions, changed files, and validation status.
- Never claim tests passed unless they were actually executed.
