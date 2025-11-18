/speckit.plan
Create a technical implementation plan for the "Integration Effort Estimator"
based on the existing specification.

Technical constraints:

- Frontend-only React application with TypeScript.
- Use Create React App for bootstrapping, whichever Spec Kit prefers.
- No backend or database in the first version; keep all data in React state.
- Organize code into:
  - `components/` for React components like ProjectForm and MandayTable.
  - `lib/` or `utils/` for pure calculation helpers, e.g. mdCalculator.ts.
- Use a simple CSS approach (CSS modules, Tailwind, or plain CSS) but avoid
  large UI frameworks to keep the codebase small.
- Include a clear mapping from the specification:
  - For each Integration feature row, map spec fields to component props/state.
  - For each formula, define a corresponding pure function in the calculation module.
- Add a section about testing:
  - Unit tests for all calculation logic.
  - At least one integration test where changing inputs re-renders the table.
- Plan should describe file structure, key components, and how MD formulas are wired
  from inputs to outputs.
