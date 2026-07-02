# Workflow: Requirement Change

## Objective
Implement requirement deltas while preserving system integrity and traceability.

## Steps

1. **Change Intake**
   - Capture old vs new requirement text and revision references.

2. **Impact Planning (Project Lead)**
   - Identify impacted components and required role handoffs.

3. **Requirement Decomposition (Requirements Engineer)**
   - Convert delta into testable acceptance criteria.
   - Identify conflicts/dependencies.

4. **Baseline Investigation (MATLAB Expert)**
   - Inspect where prior requirement is currently realized.

5. **Implementation (Simulink Developer / Stateflow Specialist)**
   - Apply scoped design updates aligned to approved criteria.

6. **Validation Update (Test Engineer)**
   - Update verification matrix and MIL/SIL assets.
   - Execute tests and gather results.

7. **Independent Review (Code Reviewer)**
   - Confirm traceability REQ delta → implementation → tests.

8. **Final Records (Documentation Agent)**
   - Publish validation report and Jira/status update.

## Mandatory Requirement-Change Rules
- No requirement interpretation by implementation agents without Requirements Engineer output.
- Traceability must be explicit and reviewable.
