/speckit.constitution
Create a constitution for a project called "Integration Effort Estimator".
The goal is a small web app that calculates Mandays (MD) from integration
features and shows a role-based effort table.

Principles:

- Frontend-only SPA using React and TypeScript in the first version.
- All business rules for MD calculations are implemented as pure functions
  (no side effects) in a dedicated module, so they can be tested easily.
- Use simple, accessible table-based UI for inputs and outputs.
- All currency input fields are non-negative numbers; treat empty as 0
  and show validation errors for negative or invalid values.
- Estimated efforts (MD) are always integers and use round-down behavior.
- The main formula for Estimated efforts (MD) is:
  (Summation of Project cost (THB) + Summation of Hardware cost (THB)) / 5000
- Do not add a backend in the first iteration; keep state in client memory.
- Require unit tests for all calculation functions and at least one
  component test for the main input/output flow.
- Documentation should clearly separate: constitution, specifications,
  implementation plan, and task list.
