/speckit.specify
Create a NEW feature spec called "002-effort-estimator".

This feature is an Integration Effort Estimator web application.

High-level behavior:

- Input page:

  - Project name (required text field).
  - Integration Features table, where each row has:
    - Feature Type (Use Case / API, Topic / Event, Data Synchronization, Batch / File, ESB Feature)
    - Summation of Project cost (THB)
    - Summation of Hardware cost (THB)
  - Estimated efforts (MD) per feature is calculated as:
    MD = ROUNDDOWN( (Project cost + Hardware cost) / 5000 )
  - Role Selection section with checkboxes:
    - System Analyst
    - Quality Assurance
    - Back-End Developer
    - Client Support Operation
      All roles are checked by default, at least one role must remain selected.

- Output page:
  - Separate view (within the same React SPA) that shows a Manday table.
  - Columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD.
  - Only columns for the selected roles are visible; Rest MD and Support + Rest MD are always visible.
  - One row per feature + a Total summary row.

Role allocation rules:

- Use Case / API:
  SA 30%, QA 0%, BE 0%, Support 70%
- Topic / Event:
  SA 30%, QA 0%, BE 0%, Support 70%
- Data Synchronization:
  SA 30%, QA 20%, BE 20%, Support 30%
- Batch / File:
  SA 30%, QA 20%, BE 20%, Support 30%
- ESB Feature:
  SA 20%, QA 30%, BE 20%, Support 30%

For each feature, compute:

- For each role: roleMD = ROUNDDOWN(E \* percentage)
- Rest MD = E - (SA + QA + BE + Support)
- Support + Rest MD = Support + Rest MD

IMPORTANT TECH CONSTRAINTS:

- Frontend-only React + TypeScript SPA.
- Do NOT mention or assume any specific build tool (no Vite, no CRA, no Next.js by name).
- Testing should be described generically (example: Jest + React Testing Library), but keep it tool-agnostic if possible.

Please:

- Create a new folder under specs/ named exactly `002-effort-estimator`
  (if your scripts choose another folder name, still make it clear in the generated spec.md
  that the Feature Branch name is `002-effort-estimator`).
- Write the specification into specs/002-effort-estimator/spec.md.
