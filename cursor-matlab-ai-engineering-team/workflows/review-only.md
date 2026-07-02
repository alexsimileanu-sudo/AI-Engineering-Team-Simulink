# Workflow: Review-Only Task

## Objective
Assess existing changes without adding new implementation.

## Steps

1. **Scope Definition (Project Lead)**
   - Define review boundaries and expected outputs.

2. **Requirement Consistency Check (Requirements Engineer)**
   - Verify requirement mapping and acceptance coverage.

3. **Technical Evidence Check (MATLAB Expert)**
   - Validate technical claims using read-only inspection.

4. **Primary Review (Code Reviewer)**
   - Evaluate correctness, safety, interface stability, test evidence.
   - Classify findings: Blocking / Major / Minor.

5. **Documentation (Documentation Agent)**
   - Produce review report and Jira comment.

## Rules
- No model/code/test edits in review-only tasks.
- No approval based on assumptions.
