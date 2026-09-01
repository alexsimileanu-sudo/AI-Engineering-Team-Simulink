# Workflow: Bug Fix

## Objective
Resolve a defect with minimal safe change and validated regression containment.

## Steps

1. **Bug Intake**
   - Record symptom, trigger conditions, observed behavior, expected behavior.
   - Attach logs, test failures, model references.

2. **Triage Plan (Project Lead)**
   - Define containment scope and fast-risk assessment.

3. **Requirement Impact (Requirements Engineer, if needed)**
   - Determine whether requirement interpretation is incorrect or incomplete.

4. **Root-Cause Investigation (MATLAB Expert)**
   - Trace defect path in model/charts/scripts.
   - Identify exact faulty logic and affected interfaces.

5. **Fix Implementation (Simulink Developer / Stateflow Specialist)**
   - Apply minimal targeted fix only.
   - Avoid architectural churn unless required.

6. **Regression Validation (Test Engineer)**
   - Add/adjust tests reproducing the bug.
   - Execute MIL/SIL regression suite subset or full set as needed.

7. **Strict Review (Code Reviewer)**
   - Verify root-cause correctness and absence of collateral impact.

8. **Documentation (Documentation Agent)**
   - Summarize root cause, fix scope, validation results, residual risk.

## Mandatory Bug-Fix Rules
- No broad refactor inside bug fix unless separately approved.
- No “fixed” claim without reproduced-before / verified-after evidence.
