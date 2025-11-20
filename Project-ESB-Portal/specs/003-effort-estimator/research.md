# Research: Integration Effort Estimator

**Feature Branch**: `003-effort-estimator`  
**Date**: November 20, 2025  
**Status**: No research required

## Overview

This document records research conducted during Phase 0 of the `/speckit.plan` workflow. The purpose is to resolve any `NEEDS CLARIFICATION` markers from the specification and document technical decisions for dependencies, patterns, and integration approaches.

## Research Status

**Status**: ✅ Complete (No unknowns to resolve)

**Rationale**: The feature specification (`spec.md`) contains no `NEEDS CLARIFICATION` markers. All requirements are fully defined:

- **Calculation formulas**: Explicitly specified with exact rounding behavior
  - Total MD: `ROUNDUP((projectCost + hardwareCost) / 5000)` → `Math.ceil()`
  - Role MD: `ROUNDDOWN(totalMD * percentage)` → `Math.floor()`
- **Role allocation rules**: Complete mapping for all 5 feature types (FR-019 to FR-023)
  - Use Case / API: SA 30%, QA 0%, BE 0%, Support 70%
  - Topic / Event: SA 30%, QA 0%, BE 0%, Support 70%
  - Data Synchronization: SA 30%, QA 20%, BE 20%, Support 30%
  - Batch / File: SA 30%, QA 20%, BE 20%, Support 30%
  - ESB Feature: SA 20%, QA 30%, BE 20%, Support 30%
- **Validation rules**: Explicit constraints on all inputs
  - Project name: Required (non-empty)
  - Feature type: Required (must be selected)
  - Costs: Non-negative numbers (empty treated as 0)
  - Roles: At least one must remain selected
- **UI structure**: Detailed component requirements
  - Input page: Project name input, features table, role selection checkboxes
  - Output page: Manday breakdown table with conditional columns
  - Total row styling: Grey background for Support and Rest MD cells

## Technical Decisions

### Technology Stack

**Decision**: React 18+ with TypeScript 5+ (strict mode)  
**Rationale**: Mandated by constitution (v1.0.1) and user requirements. TypeScript strict mode enforces type safety for calculation functions.

**Alternatives Considered**: None (constitution mandates React + TypeScript)

---

**Decision**: Use the team's preferred React build tool (tool-agnostic)  
**Rationale**: Per user requirements, do not assume Vite, CRA, or Next.js. Plan remains flexible for team's existing build toolchain.

**Alternatives Considered**: None (explicitly requested to remain tool-agnostic)

---

**Decision**: Jest + React Testing Library for testing  
**Rationale**: Constitution mandates test-first development with 100% calculation coverage. Jest is de-facto standard for React unit tests; React Testing Library aligns with accessibility principles (semantic queries).

**Alternatives Considered**: Vitest (too coupled to Vite), Testing Library alone (needs Jest for mocking)

---

### Architecture Patterns

**Decision**: Pure function calculation module (`src/lib/calculations.ts`)  
**Rationale**: Constitution Principle II (NON-NEGOTIABLE) requires pure functions for all business logic. This enables deterministic testing and prevents side effects.

**Alternatives Considered**: Class-based calculator (rejected: adds unnecessary complexity), inline calculations in components (rejected: violates constitution)

---

**Decision**: React `useState` for state management (no Redux/MobX)  
**Rationale**: Simple two-view application with straightforward state (project name, features array, role selection). Lifting state to App component is sufficient.

**Alternatives Considered**: Context API (overkill for single parent-child relationship), Redux (violates constitution's simplicity principle for v1.0)

---

**Decision**: HTML `<table>` elements for tabular data  
**Rationale**: Constitution Principle III mandates table-based UI. Semantic HTML tables provide built-in accessibility (screen readers, keyboard navigation) and familiar structure for business users.

**Alternatives Considered**: CSS Grid with divs (rejected: loses semantic meaning), third-party table library (rejected: unnecessary dependency)

---

### Validation Strategy

**Decision**: Inline validation on blur + form-level validation on submit  
**Rationale**: Provides immediate feedback without disrupting typing flow. Form-level validation catches incomplete data before view transition.

**Alternatives Considered**: Real-time validation on every keystroke (rejected: too aggressive for cost inputs), validation only on submit (rejected: poor UX)

---

### Data Flow

**Decision**: Two-view SPA with view state toggle (`'input' | 'output'`)  
**Rationale**: Constitution allows separate input/output views within SPA (v1.0.1 clarification). Simple boolean state controls which view renders; all data preserved during view switches.

**Alternatives Considered**: Multi-page app with routing (rejected: unnecessary complexity for two views), modal for output (rejected: harder to print/share)

---

## Dependencies

**Core Dependencies**:

- `react@^18.0.0` - UI framework (mandated)
- `react-dom@^18.0.0` - React DOM bindings
- `typescript@^5.0.0` - Type system (mandated)

**Dev Dependencies**:

- `@testing-library/react@^14.0.0` - Component testing
- `@testing-library/jest-dom@^6.0.0` - Custom Jest matchers
- `jest@^29.0.0` - Test runner
- `@types/react@^18.0.0` - React TypeScript definitions
- `@types/jest@^29.0.0` - Jest TypeScript definitions

**Build Tool**: Team's preferred React build tool (not specified in this plan)

---

## Best Practices Applied

### Pure Functions (Constitution Principle II)

- All calculation functions are exported from `src/lib/calculations.ts`
- Functions accept parameters and return values without modifying external state
- No DOM manipulation, API calls, or I/O within calculation module
- Each function has JSDoc comments documenting inputs, outputs, and formulas

### Test-First Development (Constitution Principle VI)

- Unit tests written for all calculation functions before implementation
- 100% coverage requirement for `calculations.ts` module
- Component test covers full user flow (input → output)
- Test data includes edge cases from spec (zero costs, decimals, single role)

### Accessibility (Constitution Principle III)

- Semantic HTML elements (`<table>`, `<label>`, `<input>`)
- ARIA labels where appropriate (`aria-label` on icon buttons)
- Keyboard navigation support (Tab, Enter, Space)
- Focus management (first input on mount, first error on validation failure)

### Input Validation (Constitution Principle IV)

- Non-negative number validation on all cost inputs
- Empty fields treated as 0 in calculations
- Inline error messages with `role="alert"` for screen readers
- Validation prevents navigation to output until data is valid

---

## Integration Patterns

**N/A**: This is a frontend-only application with no external integrations in v1.0. Future versions may integrate with:

- Backend API for project persistence
- Authentication service for multi-user support
- Export functionality (PDF/Excel) for sharing estimates

---

## Performance Considerations

**Calculation Performance**:

- Target: <100ms for 20 features (constitution constraint)
- All calculations are synchronous (no async overhead)
- Pure functions enable memoization if needed (not required for 20 features)

**Rendering Performance**:

- Target: <50ms for UI updates (constitution constraint)
- Conditional rendering for output columns (only selected roles)
- React's virtual DOM handles table updates efficiently for 20 rows

**Initial Load**:

- Target: <2 seconds on 3G (constitution constraint)
- Minimal bundle size (no heavy dependencies)
- Build tool should enable code splitting if bundle exceeds threshold

---

## Open Questions

**None**: All requirements fully specified.

---

## References

- Feature Specification: `specs/003-effort-estimator/spec.md`
- Constitution: `.specify/memory/constitution.md` (v1.0.1)
- React Documentation: https://react.dev/
- TypeScript Handbook: https://www.typescriptlang.org/docs/
- React Testing Library: https://testing-library.com/react
