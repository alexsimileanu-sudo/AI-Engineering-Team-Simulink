# GitHub Copilot Agent Adaptation

This folder contains a GitHub Copilot-friendly adaptation of the Cursor-based MATLAB/Simulink AI engineering team operating model, extended to support **Model-Based Design projects that combine Simulink/Stateflow models with manually written C/C++ code** (S-Functions, Legacy Code Tool, custom code, hand-coded drivers/algorithms integrated with generated code).

This folder is self-contained and structured exactly as it needs to be copied into a target project: everything under `.github/` is read natively by GitHub Copilot in VS Code, and `workflows/`, `checklists/`, `templates/` are supporting assets shared with the [`../cursor`](../cursor/README.md) adaptation.

## Target Environment

- VS Code with GitHub Copilot Chat
- MATLAB/Simulink, Stateflow, Embedded Coder
- Hand-written C/C++ integrated via S-Functions, Legacy Code Tool, or custom code
- MIL/SIL/PIL testing

## Repository Contents

- `.github/copilot-instructions.md` — shared global engineering rules for all roles, including the model↔C/C++ interface contract rules
- `.github/chatmodes/*.chatmode.md` — one GitHub Copilot chat mode per role
- `workflows/*.md` — operational procedures
- `checklists/*.md` — practical engineering checklists
- `templates/*.md` — reusable reporting/planning templates

## Install Into a Real Project

Copy this folder's contents into your project root:

1. Copy `.github/copilot-instructions.md` to `<project-root>/.github/copilot-instructions.md`.
2. Copy `.github/chatmodes/` to `<project-root>/.github/chatmodes/`.
3. Copy `workflows/`, `checklists/`, and `templates/` into your project root (recommended, shared with the Cursor adaptation).

### Minimal install

- Required: `.github/copilot-instructions.md`, `.github/chatmodes/*.chatmode.md`
- Optional but recommended: `workflows/`, `checklists/`, `templates/`

## Roles / Chat Modes

- `project-lead` → `project-lead.chatmode.md`
- `matlab-expert` → `matlab-expert.chatmode.md`
- `requirements-engineer` → `requirements-engineer.chatmode.md`
- `system-architect` → `system-architect.chatmode.md` *(new — System Composer architecture: components, ports, interfaces, allocation, stereotypes)*
- `simulink-developer` → `simulink-developer.chatmode.md`
- `stateflow-specialist` → `stateflow-specialist.chatmode.md`
- `c-code-integrator` → `c-code-integrator.chatmode.md` *(hand-written C/C++ and model↔C integration artifacts: S-Functions, Legacy Code Tool, custom code)*
- `build-release-engineer` → `build-release-engineer.chatmode.md` *(new — code generation/AUTOSAR config, build automation, project/toolchain settings)*
- `test-engineer` → `test-engineer.chatmode.md` *(extended with SIL/PIL and back-to-back model-vs-C testing)*
- `safety-compliance-engineer` → `safety-compliance-engineer.chatmode.md` *(new — Model Advisor/standards-compliance findings: MISRA, ISO 26262, ISO 25119, DO-178C, AUTOSAR, etc.)*
- `code-reviewer` → `code-reviewer.chatmode.md` *(extended with MISRA-C/C++ and interface-contract review)*
- `documentation-agent` → `documentation-agent.chatmode.md`

## Default Workflow

User request
→ Project Lead plan (classifies change as architecture-, model-, C/C++-, build/config-, or compliance-level, or a combination)
→ Requirements Engineer analysis
→ MATLAB Expert investigation (models, architecture, **and** C/C++ sources/headers/build specs)
→ System Architect / Simulink Developer / Stateflow Specialist / C/C++ Integration Engineer / Build & Release Engineer implementation
→ Test Engineer validation (MIL/SIL/PIL as applicable)
→ Safety & Compliance Engineer standards check (when applicable)
→ Code Reviewer review
→ Documentation Agent final documentation

## Recommended Model Usage

Each chat mode file's `model:` frontmatter field pins that role to a specific model. VS Code uses this model automatically whenever the chat mode is selected; no manual model-picker step is needed.

The assignment below was derived from a role-by-role match of each agent's actual cognitive workload (planning vs. investigation vs. implementation vs. strict review vs. writing) against GitHub's published task-based model guidance and per-token pricing (as of 2026-09-01; see [Models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing) and [Model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)).

| Role | Model | Why |
|---|---|---|
| Project Lead | `Claude Sonnet 5 (copilot)` | Frequent, general-purpose planning/orchestration with no tool-heavy work; "general-purpose coding and agent tasks" tier gives solid reasoning at moderate cost ($2/$10 per 1M input/output tokens). |
| MATLAB Expert | `GPT-5.4 mini (copilot)` | Investigation is pure tool-driven codebase/model exploration (grep-style search across models/data dictionaries) — GitHub specifically calls out this tier as excelling at grep-style codebase exploration, at roughly a third of Sonnet 5's output cost ($0.75/$4.50), for a role invoked on nearly every task. |
| System Architect | `Claude Sonnet 5 (copilot)` | Architecture edits (interface dictionary, allocation, stereotypes) need the same agentic-tool-use + reasoning profile as component-level implementation roles. |
| Simulink Developer | `Claude Sonnet 5 (copilot)` | Implementation needs reliable agentic tool use (block/port edits) plus solid reasoning to preserve interfaces; "Versatile" tier is the right cost/quality point. |
| Stateflow Specialist | `Claude Sonnet 5 (copilot)` | Same implementation profile as Simulink Developer; state-machine edits don't need Opus/Sol-tier reasoning when scope is already pre-approved and minimal. |
| C/C++ Integration Engineer | `GPT-5.3-Codex (copilot)` | Hand-written C/C++ carries the highest single-defect cost (memory safety, MISRA-C, interface contracts) of any implementation role — GitHub positions Codex specifically for "complex engineering tasks like features, tests, debugging, refactors," justifying the premium over Sonnet 5 ($1.75/$14 vs $2/$10) here specifically. |
| Build & Release Engineer | `GPT-5.6 Terra (copilot)` | Build/codegen/toolchain configuration is well-defined, tool-heavy, moderately repetitive work — GitHub positions Terra as the "balanced all-round choice for everyday interactive and agentic coding," a good middle tier between the mini and premium models. |
| Test Engineer | `Claude Sonnet 5 (copilot)` | Needs dependable, evidence-grounded reasoning (never claim an unexecuted test passed) plus agentic tool use for MIL/SIL/PIL harnesses; same tier as the other implementation roles. |
| Safety & Compliance Engineer | `Claude Opus 5 (copilot)` | Standards/safety-case findings (ISO 26262 etc.) carry certification-level consequences and are checked far less often than every-task roles, justifying the top reasoning tier ($5/$25). |
| Code Reviewer | `GPT-5.6 Sol (copilot)` | Lowest invocation frequency (one gate per change) but the highest reasoning bar — Sol is GitHub's highest-reasoning-ceiling model for "complex reasoning over large codebases," priced below Opus/GPT-5.5 tiers, making the review gate affordable to run at high scrutiny. |
| Requirements Engineer | `Gemini 3.6 Flash (copilot)` | Pure language decomposition (requirements → acceptance criteria), no code edits, no heavy tool use — a lightweight/versatile model is sufficient and meaningfully cheaper ($0.75/$3.75). |
| Documentation Agent | `Gemini 3.6 Flash (copilot)` | Same profile as Requirements Engineer: summarization/writing from already-verified evidence, no implementation. |

Notes on currency of this mapping (check before relying on it long-term):
- Several superseded models retire on **2026-09-01 itself**: Claude Sonnet 4.5/4.6 → Claude Sonnet 5, Claude Opus 4.5/4.6 → Claude Opus 5, Gemini 3.1 Pro → Gemini 3.6 Flash. Re-check GitHub's [model retirement history](https://docs.github.com/en/copilot/reference/ai-models/supported-models#model-retirement-history) periodically.
- `GPT-5.6 Sol`'s listed pricing ($2/$10 per 1M input/output tokens) is a 50%-off promotion active through **2026-09-03**; afterward it reverts to $4/$20, still cheaper than GPT-5.5 ($5/$30) or Claude Opus 5 ($5/$25) at a comparable reasoning tier.
- The `model:` value must match the exact qualified name shown in your own VS Code model picker (format `Model Name (vendor)`, e.g. `Claude Sonnet 4.5 (copilot)` or `GPT-5 (copilot)`) — double-check and update the strings above against what your Copilot subscription actually offers before relying on them. If a pinned model isn't available, VS Code falls back to the currently selected model in the picker; you can also supply an array (e.g. `model: ['Claude Sonnet 5 (copilot)', 'GPT-5.5 (copilot)']`) to give an ordered fallback list.

Note: `.chatmode.md` is the legacy custom chat mode format. VS Code now calls this primitive "custom agents" and recommends the `.agent.md` extension in `.github/agents/`; existing `.chatmode.md` files (including these) continue to work, and the `model:` field is supported in both.

## Use

This folder is intentionally kept as a sibling of `../cursor` so the repository can support both Cursor and GitHub Copilot workflows from the same source without either overwriting the other. Both role sets are aligned; the `c-code-integrator` role, interface-contract rules, and per-role model assignments exist in both.
