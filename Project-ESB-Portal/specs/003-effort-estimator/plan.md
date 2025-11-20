# Implementation Plan: Integration Effort Estimator

**Branch**: `003-effort-estimator` | **Date**: November 20, 2025 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/003-effort-estimator/spec.md`

## Summary

The Integration Effort Estimator is a frontend-only React + TypeScript single-page application that calculates manday (MD) estimates for integration projects based on feature types and costs. Users input project details and integration features (with costs), then view a role-based effort breakdown. The application enforces pure function calculation logic, integer manday estimation with round-down behavior, and table-based accessible UI patterns.

**Core Formula**: `MD = Math.ceil((projectCost + hardwareCost) / 5000)`  
**Role Allocation**: Feature-type-dependent percentages (30-70%) distributed as `Math.floor(totalMD * percentage)`  
**Technical Approach**: Client-side state management with separate calculation module; two-view SPA (Input → Output); 100% calculation unit test coverage.

## Technical Context

**Language/Version**: TypeScript 5+ (strict mode enabled)  
**Primary Dependencies**: React 18+  
**Build Tool**: Use the team's preferred React build tool (tool-agnostic design)  
**Storage**: N/A (frontend-only; all state in client memory; browser storage optional for future versions)  
**Testing**: Jest (unit tests for calculations) + React Testing Library (component tests)  
**Target Platform**: Modern web browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)  
**Project Type**: Web (single-page application)  
**Performance Goals**:

- Calculation execution: <100ms for 20 features
- UI responsiveness: <50ms for user interactions
- Initial page load: <2 seconds on 3G connection
  **Constraints**:
- No backend services or APIs in v1.0
- All calculations must use pure functions (no side effects)
- Input validation: non-negative currency values; empty fields treated as 0
- Manday values must be integers (round-down for role allocation, round-up for total MD)
  **Scale/Scope**:
- Support up to 20 integration features per project
- 5 feature types with distinct role allocation rules
- 4 roles (SA, QA, BE, Support) with dynamic selection
- Target: <3 minutes to complete estimation for 5 features

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

- [x] **Frontend-Only Architecture**: ✅ No backend services planned; all state management in client memory; calculations execute client-side
- [x] **Pure Function Business Logic**: ✅ Dedicated `src/lib/calculations.ts` module with pure functions for all MD calculations and role allocations
- [x] **Table-Based UI**: ✅ HTML tables for input feature grid and output manday breakdown; CSS Grid for role selection layout
- [x] **Input Validation**: ✅ Currency inputs validate non-negative; empty fields default to 0 in calculations; inline error feedback
- [x] **Integer MD with Round-Down**: ✅ Total MD uses `Math.ceil((costs)/5000)`; role MD uses `Math.floor(totalMD * percentage)`
- [x] **Test-First Development**: ✅ Unit tests for all calculation functions (100% coverage); component test for input→output flow
- [x] **Documentation Separation**: ✅ Constitution (`.specify/memory/constitution.md`), spec (`spec.md`), plan (`plan.md`), tasks (`tasks.md`)

**Status**: ✅ PASSED - All constitution principles satisfied

## Project Structure

### Documentation (this feature)

```text
specs/003-effort-estimator/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output: No unknowns to resolve (spec fully defined)
├── data-model.md        # Phase 1 output: Entity definitions and state model
├── quickstart.md        # Phase 1 output: Setup and run instructions
├── contracts/           # Phase 1 output: TypeScript interfaces and types
│   └── README.md        # Interface documentation
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/
├── lib/
│   ├── calculations.ts       # Pure functions: MD calculation, role allocation
│   └── calculations.test.ts  # Unit tests for calculations (100% coverage)
├── components/
│   ├── App.tsx               # Root component with view state management
│   ├── InputView.tsx         # Input page: project name, features table, role selection
│   ├── OutputView.tsx        # Output page: manday breakdown table
│   ├── FeatureRow.tsx        # Single feature row in input table
│   └── MandayTable.tsx       # Output table component
├── types/
│   ├── Feature.ts            # Feature interface (type, costs)
│   ├── Role.ts               # Role enum and selection state
│   └── MandayAllocation.ts   # Calculated allocation interface
├── utils/
│   └── validation.ts         # Input validation helpers
└── App.css                   # Styles (table layouts, grey backgrounds)

tests/
└── integration/
    └── App.test.tsx          # E2E test: input feature → view output
```

**Structure Decision**: Single web application (frontend-only React SPA). The `src/lib/` directory isolates pure calculation logic from UI components, enabling independent unit testing. The `src/components/` directory follows a view-based hierarchy (App → InputView/OutputView → child components). The `src/types/` directory centralizes TypeScript interfaces for type safety across components and calculations.

## Technical Approach

### Component Hierarchy

```text
App (root component, manages view state)
├── InputView (visible when viewState === 'input')
│   ├── ProjectNameInput (controlled text input)
│   ├── FeaturesTable (dynamic feature rows)
│   │   └── FeatureRow[] (type dropdown, cost inputs, remove button)
│   ├── AddFeatureButton (adds new row)
│   ├── RoleSelection (checkboxes for 4 roles)
│   └── ViewOutputButton (validates & switches to output)
└── OutputView (visible when viewState === 'output')
    ├── ProjectHeader (displays project name)
    ├── MandayTable (manday breakdown)
    │   ├── TableHeader (dynamic columns based on selected roles)
    │   ├── FeatureRow[] (one per feature, calculated MD allocations)
    │   └── TotalRow (summed totals, grey cells for Support/Rest MD)
    └── BackToInputButton (switches to input view)
```

### State Model

**App Component State**:

```typescript
{
  viewState: 'input' | 'output',
  projectName: string,
  features: Feature[],
  selectedRoles: RoleSelection
}
```

**Feature Interface** (`src/types/Feature.ts`):

```typescript
interface Feature {
  id: string; // UUID for React keys
  type: FeatureType; // Enum: UseCase, Topic, DataSync, Batch, ESB
  projectCost: number; // THB (non-negative, default 0)
  hardwareCost: number; // THB (non-negative, default 0)
}

enum FeatureType {
  UseCase = "Use Case / API",
  Topic = "Topic / Event",
  DataSync = "Data Synchronization",
  Batch = "Batch / File",
  ESB = "ESB Feature",
}
```

**RoleSelection Interface** (`src/types/Role.ts`):

```typescript
interface RoleSelection {
  systemAnalyst: boolean;
  qualityAssurance: boolean;
  backendDeveloper: boolean;
  clientSupport: boolean;
}

// All roles default to true on mount
```

**MandayAllocation Interface** (`src/types/MandayAllocation.ts`):

```typescript
interface MandayAllocation {
  featureId: string;
  featureName: string;
  totalMD: number;
  systemAnalyst?: number; // Included only if role selected
  qualityAssurance?: number;
  backendDeveloper?: number;
  clientSupport?: number;
  restMD: number; // Always calculated
  supportPlusRest: number; // Always calculated
}
```

### Calculation Module (`src/lib/calculations.ts`)

**Pure Functions** (exported, fully unit tested):

```typescript
/**
 * Calculate total mandays for a feature using round-up formula
 * @param projectCost - Project cost in THB
 * @param hardwareCost - Hardware cost in THB
 * @returns Total MD as integer (ceil of costs/5000)
 */
export function calculateTotalMD(
  projectCost: number,
  hardwareCost: number
): number {
  return Math.ceil((projectCost + hardwareCost) / 5000);
}

/**
 * Get role allocation percentages for a feature type
 * @param featureType - Type of integration feature
 * @returns Object with SA, QA, BE, Support percentages (0.0-1.0)
 */
export function getRolePercentages(featureType: FeatureType): {
  systemAnalyst: number;
  qualityAssurance: number;
  backendDeveloper: number;
  clientSupport: number;
} {
  // Maps feature types to allocation rules (FR-019 to FR-023)
  const percentageMap = {
    [FeatureType.UseCase]: {
      systemAnalyst: 0.3,
      qualityAssurance: 0,
      backendDeveloper: 0,
      clientSupport: 0.7,
    },
    [FeatureType.Topic]: {
      systemAnalyst: 0.3,
      qualityAssurance: 0,
      backendDeveloper: 0,
      clientSupport: 0.7,
    },
    [FeatureType.DataSync]: {
      systemAnalyst: 0.3,
      qualityAssurance: 0.2,
      backendDeveloper: 0.2,
      clientSupport: 0.3,
    },
    [FeatureType.Batch]: {
      systemAnalyst: 0.3,
      qualityAssurance: 0.2,
      backendDeveloper: 0.2,
      clientSupport: 0.3,
    },
    [FeatureType.ESB]: {
      systemAnalyst: 0.2,
      qualityAssurance: 0.3,
      backendDeveloper: 0.2,
      clientSupport: 0.3,
    },
  };
  return percentageMap[featureType];
}

/**
 * Calculate role-specific MD allocation using round-down
 * @param totalMD - Total mandays for the feature
 * @param percentage - Role allocation percentage (0.0-1.0)
 * @returns Integer MD allocated to role
 */
export function calculateRoleMD(totalMD: number, percentage: number): number {
  return Math.floor(totalMD * percentage);
}

/**
 * Calculate Rest MD (unallocated effort)
 * @param totalMD - Total mandays for the feature
 * @param allocatedRoles - Object with MD values for selected roles
 * @returns Rest MD as integer
 */
export function calculateRestMD(
  totalMD: number,
  allocatedRoles: Partial<Record<string, number>>
): number {
  const sum = Object.values(allocatedRoles).reduce(
    (acc, val) => acc + (val || 0),
    0
  );
  return totalMD - sum;
}

/**
 * Calculate complete manday allocation for a feature
 * @param feature - Feature with type and costs
 * @param selectedRoles - Boolean flags for which roles to include
 * @returns MandayAllocation object with all calculated values
 */
export function calculateFeatureAllocation(
  feature: Feature,
  selectedRoles: RoleSelection
): MandayAllocation {
  const totalMD = calculateTotalMD(feature.projectCost, feature.hardwareCost);
  const percentages = getRolePercentages(feature.type);

  const allocation: MandayAllocation = {
    featureId: feature.id,
    featureName: feature.type,
    totalMD,
  };

  const allocatedMD: Record<string, number> = {};

  if (selectedRoles.systemAnalyst) {
    allocation.systemAnalyst = calculateRoleMD(
      totalMD,
      percentages.systemAnalyst
    );
    allocatedMD.systemAnalyst = allocation.systemAnalyst;
  }
  if (selectedRoles.qualityAssurance) {
    allocation.qualityAssurance = calculateRoleMD(
      totalMD,
      percentages.qualityAssurance
    );
    allocatedMD.qualityAssurance = allocation.qualityAssurance;
  }
  if (selectedRoles.backendDeveloper) {
    allocation.backendDeveloper = calculateRoleMD(
      totalMD,
      percentages.backendDeveloper
    );
    allocatedMD.backendDeveloper = allocation.backendDeveloper;
  }
  if (selectedRoles.clientSupport) {
    allocation.clientSupport = calculateRoleMD(
      totalMD,
      percentages.clientSupport
    );
    allocatedMD.clientSupport = allocation.clientSupport;
  }

  allocation.restMD = calculateRestMD(totalMD, allocatedMD);
  allocation.supportPlusRest =
    (allocation.clientSupport || 0) + allocation.restMD;

  return allocation;
}
```

### Validation Strategy (`src/utils/validation.ts`)

```typescript
/**
 * Validate project name is not empty
 */
export function validateProjectName(name: string): {
  valid: boolean;
  error?: string;
} {
  if (!name || name.trim().length === 0) {
    return { valid: false, error: "Project name is required" };
  }
  return { valid: true };
}

/**
 * Validate currency input (non-negative number)
 */
export function validateCost(value: string): {
  valid: boolean;
  error?: string;
  parsed: number;
} {
  const parsed = parseFloat(value) || 0; // Empty treated as 0
  if (parsed < 0) {
    return { valid: false, error: "Cost cannot be negative", parsed: 0 };
  }
  return { valid: true, parsed };
}

/**
 * Validate feature has type selected
 */
export function validateFeature(feature: Feature): {
  valid: boolean;
  error?: string;
} {
  if (!feature.type) {
    return { valid: false, error: "Feature type must be selected" };
  }
  return { valid: true };
}

/**
 * Validate at least one role is selected
 */
export function validateRoleSelection(roles: RoleSelection): {
  valid: boolean;
  error?: string;
} {
  const hasAtLeastOne = Object.values(roles).some((selected) => selected);
  if (!hasAtLeastOne) {
    return { valid: false, error: "At least one role must be selected" };
  }
  return { valid: true };
}
```

### Testing Strategy

**Unit Tests** (`src/lib/calculations.test.ts`):

- ✅ 100% coverage requirement (constitution mandate)
- Test `calculateTotalMD` with various cost combinations (0, decimals, large values)
- Test `getRolePercentages` for all 5 feature types
- Test `calculateRoleMD` with round-down edge cases
- Test `calculateRestMD` for partial role selections
- Test `calculateFeatureAllocation` for each feature type + role combination

**Component Tests** (`tests/integration/App.test.tsx`):

- E2E flow: render → enter project name → add 2 features → select roles → view output
- Verify output table displays correct MD values
- Verify role column visibility based on selection
- Verify validation prevents empty project name or missing feature types
- Verify "at least one role" validation

**Test Data Examples**:

```typescript
// Test case from spec acceptance scenario
{
  feature: { type: 'Use Case / API', projectCost: 15000, hardwareCost: 10000 },
  expected: { totalMD: 5, SA: 1, QA: 0, BE: 0, Support: 3, Rest: 1 }
}
```

### UI/UX Implementation Notes

**Input View**:

- HTML `<table>` for features grid (semantic markup)
- Feature rows: `<select>` for type, `<input type="number" min="0">` for costs
- Role selection: `<input type="checkbox">` with labels, all checked by default
- Disable last role checkbox if only one remaining (prevent unchecking all)
- "View Output" button triggers validation, shows inline errors if invalid

**Output View**:

- HTML `<table>` for manday breakdown
- Dynamic columns: render only selected role columns + Rest MD + Support+Rest MD
- Total row: grey background (`background-color: #e0e0e0`) for Support and Rest MD cells, no numeric values
- "Back to Input" button preserves all entered data (return to input view without reset)

**Accessibility**:

- `<label for="...">` for all inputs
- `aria-label` on remove buttons ("Remove feature")
- `role="alert"` on validation error messages
- Keyboard navigation: Tab through inputs, Enter to submit, Space for checkboxes
- Focus management: first input focused on mount, first error focused on validation failure

## Phase 0: Research

**Status**: ✅ No research required

**Rationale**: The feature specification contains no `NEEDS CLARIFICATION` markers. All functional requirements are fully defined:

- Calculation formulas specified exactly (ROUNDUP, ROUNDDOWN)
- Role allocation percentages defined for all 5 feature types (FR-019 to FR-023)
- Validation rules explicit (non-negative, at least one role)
- UI structure detailed (input fields, output columns, styling)

The technical constraints (React, TypeScript, frontend-only) are mandated by the constitution and user requirements. No external dependencies or integration patterns require research.

**Output**: `research.md` will document this decision (no unknowns to resolve).

## Phase 1: Design

**Deliverables**:

1. **data-model.md**: Entity definitions (Feature, RoleSelection, MandayAllocation)
2. **contracts/**: TypeScript interfaces for all entities (exported from `src/types/`)
3. **quickstart.md**: Setup instructions (install, run dev server, run tests)

**Design Decisions**:

- **State Management**: React `useState` in App component (no Redux/MobX needed; simple two-view toggle)
- **Form Handling**: Controlled components with inline validation on blur
- **Table Rendering**: Conditional column rendering using `selectedRoles` object
- **Styling**: CSS classes for table layouts, grey backgrounds, responsive design (mobile-friendly tables)

**Agent Context Update**: After Phase 1 completion, run `.specify/scripts/bash/update-agent-context.sh copilot` to add:

- Technology: React 18+, TypeScript 5+
- Patterns: Pure calculation functions, controlled React forms, table-based UI
- Testing: Jest + React Testing Library
