/speckit.tasks
Generate a task breakdown for feature branch `002-effort-estimator`
based on specs/002-effort-estimator/spec.md and plan.md.

Requirements:

- Frontend-only React + TypeScript.
- Do NOT mention Vite or Vitest anywhere in the tasks.
- In the setup phase, describe project creation generically:
  "Initialize a React + TypeScript project with the team's chosen build tool."
- For testing, you may mention Jest + React Testing Library as examples,
  but phrase them as "React-compatible test runner" so they are not mandatory.
- Include phases for:
  - Project setup
  - Calculation helpers (MD + role allocation)
  - Input page (project + features + role checkboxes)
  - Output page (Manday table + navigation)
  - Validation and UX polish
  - Accessibility and documentation.

Write the result into specs/002-effort-estimator/tasks.md.
