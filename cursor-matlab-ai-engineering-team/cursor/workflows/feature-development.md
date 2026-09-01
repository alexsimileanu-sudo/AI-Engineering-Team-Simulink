# Workflow: Feature Development

## Objective
Deliver a new feature with controlled scope, traceability, and validated behavior for automotive ECU software.

## Steps

1. **Intake (User → Project Lead)**
   - Capture feature objective, constraints, target ECU behavior, and deadline.

2. **Planning (Project Lead)**
   - Create role-based execution plan.
   - Define dependencies, risks, and validation gates.

3. **Requirement Analysis (Requirements Engineer)**
   - Decompose requirements into acceptance criteria.
   - Flag ambiguity and missing requirement details.

4. **Baseline Investigation (MATLAB Expert)**
   - Inspect current model/charts/data/test baseline.
   - Identify exact implementation touchpoints.

5. **Implementation (Simulink Developer / Stateflow Specialist)**
   - Implement only approved scope.
   - Preserve interfaces unless explicitly approved to change.

6. **Validation (Test Engineer)**
   - Update MIL/SIL test artifacts.
   - Execute tests and collect evidence.

7. **Review (Code Reviewer)**
   - Perform strict quality and safety review.
   - Challenge assumptions and unsupported claims.

8. **Closure (Documentation Agent)**
   - Prepare commit message, Jira update, and validation report from real artifacts.

## Mandatory Gates
- No implementation before investigation + requirement analysis.
- No closure without review and validation status.
