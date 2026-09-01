# MATLAB/Simulink AI Engineering Team

A reusable AI agent team framework for MATLAB/Simulink automotive embedded development, including projects that combine Simulink/Stateflow models with manually written C/C++ code (S-Functions, Legacy Code Tool, custom code). The same 12-role operating model is provided for two AI coding assistants, each in its own self-contained, ready-to-copy folder.

## Structure

- [`cursor/`](cursor/README.md) — Cursor IDE adaptation: `AGENTS.md`, `.cursor/rules/*.mdc`, `.cursor/agents/*.md`, `workflows/`, `checklists/`, `templates/`.
- [`github_copilot/`](github_copilot/README.md) — GitHub Copilot (VS Code) adaptation: `.github/copilot-instructions.md`, `.github/chatmodes/*.chatmode.md`, `workflows/`, `checklists/`, `templates/`.

Each folder is independent and can be copied as-is into a target project root — see the README inside each folder for install steps.

## Team Roles (both adaptations)

1. Project Lead
2. MATLAB Expert
3. Requirements Engineer
4. System Architect *(System Composer: components, ports, interfaces, allocation, stereotypes, requirement links)*
5. Simulink Developer
6. Stateflow Specialist
7. C/C++ Integration Engineer *(hand-written C/C++ and model↔C integration artifacts: S-Functions, Legacy Code Tool, custom code)*
8. Build & Release Engineer *(Embedded Coder/AUTOSAR code generation, build automation, project/toolchain configuration)*
9. Test Engineer *(MIL/SIL/PIL)*
10. Safety & Compliance Engineer *(Model Advisor/standards-compliance findings: MISRA, ISO 26262, ISO 25119, DO-178C, DO-254, IEC 61508, IEC 62304, EN 50128, CERT C/CWE, AUTOSAR)*
11. Code Reviewer *(includes MISRA-C/C++ review)*
12. Documentation Agent

This covers the full MBD lifecycle end to end: system architecture → requirements → component implementation (model and hand-written C/C++) → build/codegen → verification (MIL/SIL/PIL) → standards/safety compliance → review → documentation.

## Choosing a Folder

- Using Cursor IDE → copy from [`cursor/`](cursor/README.md).
- Using GitHub Copilot in VS Code → copy from [`github_copilot/`](github_copilot/README.md).
- Using both tools on the same project → copy both folders' contents; the role definitions and anti-hallucination/interface-contract rules are kept in sync between them.
