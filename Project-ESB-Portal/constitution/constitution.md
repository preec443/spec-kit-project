<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 1.0.1 (Clarification Amendment)
Modified Principles:
  - III. Table-Based Accessible UI: Added clarification about separate input/output views
Added Sections: None
Removed Sections: None

Templates Requiring Updates:
  ✅ plan-template.md - No changes required (generic principle check remains valid)
  ✅ spec-template.md - No changes required (user stories unaffected)
  ✅ tasks-template.md - No changes required (task structure unaffected)

Follow-up TODOs: None

Bump Rationale: PATCH version (1.0.0 → 1.0.1) for clarification of existing principle without semantic change. The amendment explicitly permits separate input/output views (e.g., tabbed interface or routing) while maintaining all existing constraints (semantic tables, keyboard navigation, visual distinction). This clarifies implementation flexibility without redefining the principle's core intent.
==================
-->

<!-- PREVIOUS VERSIONS:
Version 1.0.0 (2025-11-18): Initial constitution establishment with 7 core principles
-->

# Integration Effort Estimator Constitution

## Core Principles

### I. Frontend-Only Architecture (First Version)

The Integration Effort Estimator MUST be implemented as a frontend-only Single Page Application (SPA) in its first iteration. This means:

- All state management occurs in client memory
- No backend services or APIs in version 1.0
- All data processing and calculations execute client-side
- Data persistence (if needed) uses browser storage only

**Rationale**: Minimizes initial complexity, enables rapid prototyping, and delivers core value without infrastructure dependencies. Backend integration can be added in future versions when scale or persistence requirements justify the complexity.

### II. Pure Function Business Logic (NON-NEGOTIABLE)

All business rules for manday (MD) calculations MUST be implemented as pure functions without side effects. This requires:

- A dedicated calculation module (e.g., `src/calculations/` or `src/lib/calculations.ts`)
- Functions accept inputs as parameters and return outputs without modifying external state
- No direct DOM manipulation, API calls, or I/O operations within calculation functions
- Each calculation function is independently testable with deterministic results

**Rationale**: Pure functions enable comprehensive unit testing, predictable behavior, easier debugging, and future refactoring without fear of breaking hidden dependencies.

### III. Table-Based Accessible UI

User interface MUST prioritize simplicity and accessibility using table-based layouts for inputs and outputs. Requirements:

- Input forms use HTML tables or CSS Grid for structured data entry
- Output displays use semantic HTML tables with proper headers
- All interactive elements are keyboard-navigable
- ARIA labels and roles where appropriate
- Clear visual distinction between input and output sections

The application MAY present separate views for input and output (e.g., an Input page and an Output page) as long as it remains a single-page React application. Each view MUST continue to use semantic tables for tabular data, preserve keyboard navigability, and maintain a clear visual distinction between input controls and calculated outputs.

**Rationale**: Tables naturally represent tabular data (integration features, costs, role-based efforts), are familiar to business users, and provide inherent structure that aids accessibility.

### IV. Input Validation and Handling

All currency and numeric input fields MUST enforce non-negative number validation:

- Empty fields are treated as zero (0) in calculations
- Negative values trigger immediate validation errors with clear messaging
- Invalid input (non-numeric characters) displays inline error feedback
- Validation occurs on blur or form submission, not character-by-character
- Error messages specify the exact issue (e.g., "Hardware cost cannot be negative")

**Rationale**: Prevents calculation errors, improves data quality, and provides clear user feedback. Treating empty as zero reduces friction for optional fields.

### V. Integer Manday Estimation with Round-Down

All estimated efforts (mandays/MD) MUST be integers calculated using round-down behavior:

- The core formula: `MD = floor((Σ Project Cost (THB) + Σ Hardware Cost (THB)) / 5000)`
- JavaScript `Math.floor()` or equivalent ensures consistent rounding
- Display formatted integers without decimal places
- Document rounding behavior in UI tooltips or help text

**Rationale**: Mandays represent discrete work units (whole days); fractional days create planning ambiguity. Round-down is conservative and aligns with typical effort estimation practices.

### VI. Test-First Development (NON-NEGOTIABLE)

Testing is mandatory and follows strict coverage requirements:

- **Unit tests**: 100% coverage of all calculation functions (pure functions in calculation module)
- **Component tests**: At least one end-to-end test covering the main input/output flow
- Tests MUST be written before or alongside implementation
- All tests MUST pass before code review/merge
- Use a recognized testing framework (e.g., Jest)

**Rationale**: Calculation accuracy is critical for business decisions. Pure functions make unit testing straightforward; component tests verify integration. Test-first prevents regression and documents expected behavior.

### VII. Clear Documentation Separation

Project documentation MUST maintain clear boundaries between four artifact types:

- **Constitution** (`.specify/memory/constitution.md`): Non-negotiable principles and governance
- **Specifications** (`/specs/###-feature/spec.md`): User requirements, acceptance criteria, functional specs
- **Implementation Plan** (`/specs/###-feature/plan.md`): Technical approach, architecture, structure
- **Task List** (`/specs/###-feature/tasks.md`): Actionable work items with dependencies

**Rationale**: Separation of concerns enables independent review/approval of principles, requirements, design, and execution. Each artifact serves distinct stakeholders (governance reviewers, product owners, architects, developers).

## Technical Constraints

**Technology Stack** (Version 1.0):

- **Frontend Framework**: React 18+
- **Language**: TypeScript 5+ with strict mode enabled
- **Build Tool**: Create React App
- **Testing**: Jest (unit tests) + React Testing Library (component tests)

**Forbidden in Version 1.0**:

- Backend services (Node.js/Express, databases, APIs)
- External API calls or third-party data sources
- Complex state management libraries (Redux, MobX) unless justified by complexity
- CSS frameworks that compromise accessibility (must audit any framework used)

**Calculation Module Requirements**:

- Located at `src/calculations/` or `src/lib/calculations.ts`
- Exports named functions (e.g., `calculateTotalMD`, `calculateRoleEffort`)
- Each function has JSDoc comments documenting inputs, outputs, and formula
- Co-located test file (e.g., `calculations.test.ts`) in same directory or `tests/unit/`

## Quality Standards

### Testing Gates

Code MUST NOT be merged without:

1. All unit tests passing (100% coverage of calculation functions)
2. Component test(s) passing for main user flow
3. No TypeScript compilation errors
4. No linting errors (ESLint with TypeScript rules)

### Code Quality

- **TypeScript**: Strict mode enabled, no `any` types without explicit justification
- **Naming**: Clear, descriptive names (e.g., `calculateMandays` not `calcMD`)
- **Functions**: Single responsibility, prefer smaller functions over large ones
- **Comments**: Explain _why_ not _what_; calculation formulas MUST be documented

### Performance

- Initial page load: <2 seconds on 3G connection
- Calculation execution: <100ms for typical inputs (10-20 integration features)
- UI responsiveness: No blocking operations >50ms; use Web Workers if needed for large datasets

## Governance

This Constitution supersedes all other development practices and guidelines for the Integration Effort Estimator project. All decisions, code reviews, and feature implementations MUST align with the principles defined above.

**Amendment Process**:

1. Proposed changes require written justification with impact analysis
2. Major version bump for principle removal or redefinition
3. Minor version bump for new principles or expanded guidance
4. Patch version bump for clarifications or wording improvements
5. All amendments documented in Sync Impact Report at top of constitution file

**Compliance**:

- Constitution Check gate in `plan-template.md` MUST verify principle adherence
- Code reviews MUST validate pure function patterns, input validation, test coverage
- Any principle violation requires explicit approval and documentation in plan's Complexity Tracking section

**Versioning Policy**:

- Constitution follows semantic versioning: MAJOR.MINOR.PATCH
- Version updates captured in Sync Impact Report
- Templates (plan, spec, tasks) updated within same commit as constitution amendments

**Version**: 1.0.1 | **Ratified**: 2025-11-18 | **Last Amended**: 2025-11-19
