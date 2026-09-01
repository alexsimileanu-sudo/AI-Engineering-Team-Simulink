# Cursor MATLAB/Simulink AI Engineering Team

A reusable AI agent team framework for MATLAB/Simulink automotive embedded development in Cursor IDE.  
It provides role-based agents, strict engineering boundaries, anti-hallucination rules, workflows, checklists, and templates for daily ECU development, including projects that combine Simulink/Stateflow models with manually written C/C++ code.

This folder is self-contained and structured exactly as it needs to be copied into a target project. The sibling [`../github_copilot`](../github_copilot/README.md) folder provides the same operating model adapted for GitHub Copilot in VS Code.

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
- **System Architect**: System Composer architecture implementation only (components, ports, interfaces, allocation, stereotypes)
- **Simulink Developer**: Simulink implementation only
- **Stateflow Specialist**: Stateflow implementation only
- **C/C++ Integration Engineer**: hand-written C/C++ and model↔C integration artifacts only (S-Functions, Legacy Code Tool, custom code)
- **Build & Release Engineer**: code generation/AUTOSAR config, build automation, project/toolchain settings only
- **Test Engineer**: test artifacts only (MIL/SIL/PIL)
- **Safety & Compliance Engineer**: standards-compliance/Model Advisor findings only (MISRA, ISO 26262, ISO 25119, DO-178C, AUTOSAR, etc.); does not implement fixes
- **Code Reviewer**: strict review and challenge assumptions (includes MISRA-C/C++)
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

### C/C++ Integration Engineer
“C/C++ Integration Engineer: Implement the approved change to `torque_arbitration.c`/`.h` and update the matching Legacy Code Tool spec and S-Function block so the model interface stays consistent. Do not modify model logic or generated code.”

### System Architect
“System Architect: Add the new `ArbitrationOutput` port and interface to the `TorqueArbitration` architecture component per requirement REQ-TRQ-011, and update the interface dictionary and allocation set. Do not implement the component's internal Simulink logic.”

### Build & Release Engineer
“Build & Release Engineer: Update the Embedded Coder configuration to add the new AUTOSAR RTE mapping for `ArbitrationOutput` and rebuild. Report the actual build result. Do not modify model logic or hand-written C.”

### Test Engineer
“Test Engineer: Create/modify MIL and SIL test artifacts for acceptance criteria AC1..AC6. Do not change production model logic.”

### Safety & Compliance Engineer
“Safety & Compliance Engineer: Run the MISRA-C and ISO 26262 Model Advisor checks against the arbitration subsystem and `torque_arbitration.c`. Report findings with rule IDs; do not apply fixes.”

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
5. Focused implementation by System Architect / Simulink / Stateflow / C/C++ Integration / Build & Release agents
6. Test Engineer updates validation matrix and tests
7. Safety & Compliance Engineer runs standards checks when applicable
8. Code Reviewer verifies traceability and impact containment
9. Documentation Agent publishes validation and compliance narrative

## Recommended Model Usage

Model binding is set in each `.cursor/agents/*.md` file's `model:` frontmatter field (Cursor Project Rules in `.cursor/rules/*.mdc` have no model field — they only carry persona/boundaries text). Invoke a role by its exact subagent name (`/project-lead`, `/matlab-expert`, etc.) to guarantee the bound model is used; natural-language or automatic delegation is not guaranteed to pick the intended subagent.

The assignment below was derived from a role-by-role match of each agent's actual cognitive workload (planning vs. investigation vs. implementation vs. strict review vs. writing) against GitHub's published task-based model guidance and per-token pricing (as of 2026-09-01; see [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) and [Model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)). Cursor exposes overlapping model families but its own naming/availability can differ — verify each slug against Cursor's model picker before relying on it.

| Role | Model | Why |
|---|---|---|
| Project Lead | `claude-sonnet-5` | Frequent, general-purpose planning/orchestration with no tool-heavy work; "general-purpose coding and agent tasks" tier gives solid reasoning at moderate cost. |
| MATLAB Expert | `gpt-5.4-mini` | Investigation is pure tool-driven codebase/model exploration (grep-style search across models/data dictionaries) — the mini tier is specifically called out as excelling at this, at roughly a third of Sonnet 5's output cost, for a role invoked on nearly every task. |
| System Architect | `claude-sonnet-5` | Architecture edits (interface dictionary, allocation, stereotypes) need the same agentic-tool-use + reasoning profile as component-level implementation roles. |
| Simulink Developer | `claude-sonnet-5` | Implementation work needs reliable agentic tool use (block/port edits via MCP) plus solid reasoning to preserve interfaces; general-purpose "Versatile" tier is the right cost/quality point. |
| Stateflow Specialist | `claude-sonnet-5` | Same implementation profile as Simulink Developer; state-machine edits don't need Opus/Sol-tier reasoning to stay correct when scope is already pre-approved and minimal. |
| C/C++ Integration Engineer | `gpt-5.3-codex` | Hand-written C/C++ carries the highest single-defect cost (memory safety, MISRA-C, interface contracts) of any implementation role — a coding-specialized agentic model justifies its premium over Sonnet 5 here specifically. |
| Build & Release Engineer | `gpt-5.6-terra` | Build/codegen/toolchain configuration is well-defined, tool-heavy, moderately repetitive work — GitHub positions Terra as the "balanced all-round choice for everyday interactive and agentic coding," a good middle tier between the mini and premium models. |
| Test Engineer | `claude-sonnet-5` | Needs dependable, evidence-grounded reasoning (never claim an unexecuted test passed) plus agentic tool use for MIL/SIL/PIL harnesses; same tier as the other implementation roles. |
| Safety & Compliance Engineer | `claude-opus-5` | Standards/safety-case findings (ISO 26262 etc.) carry certification-level consequences and are checked far less often than every-task roles, justifying the top reasoning tier. |
| Code Reviewer | `gpt-5.6-sol` | Lowest invocation frequency (one gate per change) but the highest reasoning bar — Sol is GitHub's highest-reasoning-ceiling model for "complex reasoning over large codebases," priced below Opus/GPT-5.5 tiers, making the review gate affordable to run at high scrutiny. |
| Requirements Engineer | `gemini-3.6-flash` | Pure language decomposition (requirements → acceptance criteria), no code edits, no heavy tool use — a lightweight/versatile model is sufficient and meaningfully cheaper. |
| Documentation Agent | `gemini-3.6-flash` | Same profile as Requirements Engineer: summarization/writing from already-verified evidence, no implementation. |
| `test-engineer-gap-analysis` (on-demand escalation) | `claude-opus-5` | Only invoked explicitly for ambiguous/contested/safety-relevant coverage gaps — infrequent enough to afford the top reasoning tier. |

Notes on currency of this mapping (check before relying on it long-term):
- Several models this mapping replaces are retiring on 2026-09-01 itself: `claude-sonnet-4-5`/`claude-sonnet-4-6` → `claude-sonnet-5`, `claude-opus-4-5`/`claude-opus-4-6` → `claude-opus-5`, `gemini-3-1-pro` → `gemini-3-6-flash`. Re-check GitHub's [model retirement history](https://docs.github.com/en/copilot/reference/ai-models/supported-models#model-retirement-history) periodically.
- `gpt-5.6-sol`'s listed pricing ($2/$10 per 1M input/output tokens) is a 50%-off promotion active through **2026-09-03**; afterward it reverts to $4/$20, still cheaper than GPT-5.5 ($5/$30) or Claude Opus 5 ($5/$25) at a comparable reasoning tier.
- Exact model ID strings should be double-checked against Cursor's own model picker/autocomplete in your account, since Cursor's published docs mix dot- and dash-separated slugs across model families and don't publish a single canonical list.

## Engineering Safety Principles (Always On)

- Never invent signals, bus objects, parameters, requirements, or test outcomes.
- Always inspect before editing.
- Never modify unrelated models/artifacts.
- Preserve interfaces and architecture unless explicitly approved.
- Always report assumptions, changed files, and validation status.
- Never claim tests passed unless they were actually executed.
