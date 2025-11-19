# Implementation Plan: Integration Effort Estimator

**Branch**: `001-effort-estimator` | **Date**: 2025-11-18 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-effort-estimator/spec.md`

## Summary

The Integration Effort Estimator is a frontend-only React/TypeScript web application that calculates mandays (MD) from integration feature costs and displays role-based effort distribution. Users input project name and multiple feature rows (type, project cost, hardware cost), and the system instantly calculates estimated MD using `floor((costs) / 5000)` and distributes effort across SA/QA/BE/Support roles based on feature-type-specific percentages. All calculations use pure functions with Math.floor() for consistent rounding. The UI displays both input forms and output tables using accessible table layouts with real-time reactivity.

## Technical Context

**Language/Version**: TypeScript 5.3+ with strict mode enabled  
**Primary Dependencies**: React 18+, Vite 5+ (build tool)  
**Storage**: N/A (in-memory React state only; no persistence in v1.0)  
**Testing**: Vitest (unit tests) + React Testing Library (component tests)  
**Target Platform**: Modern web browsers (Chrome, Firefox, Safari, Edge from last 2 years)  
**Project Type**: Frontend-only web application (single-page app)  
**Performance Goals**: <100ms calculation time for 20 features; <2s initial page load on 3G  
**Constraints**: No blocking operations >50ms; frontend-only (no backend/database); accessible keyboard navigation  
**Scale/Scope**: Up to 50 integration features per project; single-user session-based state

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

- [x] **Frontend-Only Architecture**: Verify no backend services planned for v1.0 ✅ React SPA with in-memory state only
- [x] **Pure Function Business Logic**: Calculation module uses pure functions only (no side effects) ✅ `src/lib/calculations.ts` with exported pure functions
- [x] **Table-Based UI**: Design uses table/grid layouts for inputs and outputs ✅ HTML tables for input rows and output table
- [x] **Input Validation**: All currency fields have non-negative validation; empty treated as 0 ✅ Validation logic in form handlers
- [x] **Integer MD with Round-Down**: Formula uses `Math.floor()` for manday calculations ✅ All calculation functions use Math.floor()
- [x] **Test-First Development**: Unit tests for all calculations + component test for main flow ✅ Vitest unit tests + RTL component tests planned
- [x] **Documentation Separation**: Constitution, spec, plan, and tasks are distinct artifacts ✅ This plan is separate from spec.md

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/
├── components/
│   ├── App.tsx                    # Root component, manages project state
│   ├── ProjectForm.tsx            # Project name input
│   ├── RoleSelector.tsx           # Role visibility checkboxes
│   ├── FeatureInputTable.tsx      # Table of feature input rows
│   ├── FeatureRow.tsx             # Single feature input row (type, costs)
│   ├── OutputView.tsx             # Output page wrapper with navigation
│   └── MandayTable.tsx            # Output table with role-based MD
├── lib/
│   ├── calculations.ts            # Pure functions for all MD calculations
│   ├── types.ts                   # TypeScript interfaces (Feature, RoleAllocation, etc.)
│   └── constants.ts               # Feature types, role percentages config
├── App.css                        # Global styles (or CSS modules per component)
├── main.tsx                       # React app entry point
└── vite-env.d.ts                  # Vite type declarations

tests/
├── unit/
│   └── calculations.test.ts       # Unit tests for all calculation functions
└── components/
    └── App.test.tsx               # Integration test: input changes → table updates

public/
└── index.html                     # HTML shell

package.json                       # Dependencies: react, typescript, vite, vitest, @testing-library/react
tsconfig.json                      # TypeScript strict mode config
vite.config.ts                     # Vite build config + Vitest setup
```

**Structure Decision**: Frontend-only web application using Vite for fast development and build. All business logic isolated in `src/lib/calculations.ts` as pure functions (Constitution Principle II). Components in `src/components/` follow single-responsibility principle. Tests co-located by type (unit vs component) for clarity. No backend directories since this is v1.0 frontend-only (Constitution Principle I).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations. All principles are satisfied:

- Frontend-only architecture with no backend
- Pure functions for all calculations
- Table-based accessible UI
- Complete input validation
- Math.floor() for all rounding
- Comprehensive test coverage planned
- Documentation properly separated

---

## Architecture & Components

### Component Hierarchy

```
App
├── ProjectForm (project name input)
├── RoleSelector (role checkboxes)
├── FeatureInputTable
│   └── FeatureRow[] (one per feature)
│       ├── Feature type dropdown
│       ├── Project cost input
│       ├── Hardware cost input
│       ├── Estimated MD display (calculated)
│       └── Delete button
└── OutputView
    └── MandayTable (output table with totals and dynamic role columns)
```

### State Management

**App.tsx** manages all state in a single React component:

```typescript
interface Feature {
  id: string; // UUID for React keys
  type: FeatureType | ""; // Empty string = not selected
  projectCost: number; // Default 0
  hardwareCost: number; // Default 0
}

type RoleKey = "sa" | "qa" | "be" | "support";

interface AppState {
  projectName: string;
  features: Feature[];
  selectedRoles: RoleKey[]; // at least one role, default ['sa','qa','be','support']
  activeView: "input" | "output"; // default 'input'
}
```

**Implementation Notes**:

- `selectedRoles` is updated by the RoleSelector component via callbacks.
- `activeView` is toggled by navigation buttons (e.g., 'View Output' and 'Back to Input').

**Data Flow**:

1. User types in input fields → onChange handlers update state
2. State change triggers re-render
3. Components call pure functions from `calculations.ts` to derive outputs
4. Derived data (estimated MD, role allocations) calculated on each render

No external state management library needed (per Constitution: avoid complexity unless justified).

### Component Details

#### 1. App.tsx (Root Component)

**Responsibilities**:

- Manage project name and features array state
- Provide handlers: `addFeature()`, `updateFeature(id, field, value)`, `deleteFeature(id)`
- Pass state and handlers down to child components
- Orchestrate layout: header, project form, input table, output table

**Props**: None (root component)

**State**:

```typescript
const [projectName, setProjectName] = useState<string>("");
const [features, setFeatures] = useState<Feature[]>([]);
```

---

#### 2. ProjectForm.tsx

**Responsibilities**:

- Render project name input field
- Basic validation (required field indicator)

**Props**:

```typescript
interface ProjectFormProps {
  projectName: string;
  onProjectNameChange: (name: string) => void;
}
```

**Implementation Notes**:

- Single text input with label "Project Name"
- Mark as required with asterisk or ARIA
- No complex validation (name can be any string)

---

#### 3. FeatureInputTable.tsx

**Responsibilities**:

- Render table with header row (Feature Type, Project Cost, Hardware Cost, Estimated MD, Actions)
- Map features array to FeatureRow components
- Display "Add Feature" button below table

**Props**:

```typescript
interface FeatureInputTableProps {
  features: Feature[];
  onAddFeature: () => void;
  onUpdateFeature: (id: string, field: keyof Feature, value: any) => void;
  onDeleteFeature: (id: string) => void;
}
```

**Implementation Notes**:

- Use semantic HTML `<table>` with `<thead>` and `<tbody>`
- Table headers include column labels with units (e.g., "Project Cost (THB)")
- "Add Feature" button creates empty feature row with generated UUID

---

#### 4. FeatureRow.tsx

**Responsibilities**:

- Render single feature input row (one `<tr>` element)
- Handle validation errors inline
- Calculate and display Estimated MD using `calculateEstimatedMD()` from calculations module

**Props**:

```typescript
interface FeatureRowProps {
  feature: Feature;
  onUpdate: (field: keyof Feature, value: any) => void;
  onDelete: () => void;
}
```

**Validation Logic** (per Constitution Principle IV):

- Empty cost fields → treat as 0 (no error)
- Negative costs → display inline error message, highlight field
- Non-numeric input → display inline error message
- Validation triggers on blur, not per-character

**Estimated MD Display**:

```typescript
const estimatedMD = calculateEstimatedMD(
  feature.projectCost || 0,
  feature.hardwareCost || 0
);
```

---

#### 5. MandayTable.tsx

**Responsibilities**:

- Render output table with role-based MD allocations
- One row per feature + Total summary row
- Columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD
- Dynamically show/hide role columns based on selected roles

**Props**:

```typescript
interface MandayTableProps {
  features: Feature[];
  selectedRoles: RoleKey[]; // controls which role columns are visible
}
```

**Implementation Notes**:

- MandayTable MUST always display 'Feature', 'Rest MD', and 'Support + Rest MD' columns.
- It MUST conditionally render 'SA (MD)', 'QA (MD)', 'BE (MD)', and 'Support (MD)' columns depending on whether each role key is present in selectedRoles.

**Calculation Flow**:

```typescript
// For each feature:
const estimatedMD = calculateEstimatedMD(projectCost, hardwareCost);
const roleAllocations = calculateRoleAllocations(feature.type, estimatedMD);
// roleAllocations = { sa, qa, be, support, restMD, supportPlusRest }

// Total row:
const totals = calculateTotals(features);
```

**Implementation Notes**:

- Use semantic HTML `<table>` with proper headers
- Feature column shows feature type name (e.g., "Use Case / API")
- All MD values displayed as integers (no decimals)
- Total row uses bold or different styling for distinction
- Empty state: if no features, show message "No features added yet"

---

#### 6. RoleSelector.tsx

**Responsibilities**:

- Render four checkboxes for System Analyst, Quality Assurance, Back-End Developer, and Client Support Operation.
- Ensure at least one checkbox remains selected by preventing the user from unchecking the last selected role and showing an inline helper message when that happens.

**Props**:

```typescript
interface RoleSelectorProps {
  selectedRoles: RoleKey[];
  onChange: (nextRoles: RoleKey[]) => void;
}
```

**Implementation Notes**:

- All four checkboxes MUST be checked by default when the application loads (per FR-021).
- When user attempts to uncheck the last remaining role, prevent the action and display inline message: "At least one role must be selected." (per FR-022)
- Use semantic HTML with `<input type="checkbox">` and associated `<label>` elements.
- Apply ARIA attributes for accessibility.

---

#### 7. OutputView.tsx

**Responsibilities**:

- Render the MandayTable component based on the current features and selectedRoles.
- Provide a 'Back to Input' button that sets activeView back to 'input'.

**Props**:

```typescript
interface OutputViewProps {
  features: Feature[];
  selectedRoles: RoleKey[];
  onBackToInput: () => void;
}
```

**Implementation Notes**:

- OutputView is a wrapper component that contains navigation controls and the MandayTable.
- The 'Back to Input' button MUST be keyboard accessible and clearly labeled (per FR-024).
- OutputView receives selectedRoles from App state and passes to MandayTable for column filtering.

---

## Pure Calculation Functions (src/lib/calculations.ts)

All functions are pure (no side effects, deterministic output from inputs). Each function has JSDoc comments documenting formula and behavior.

### Function Specifications

#### 1. `calculateEstimatedMD(projectCost: number, hardwareCost: number): number`

**Formula**: `MD = Math.floor((projectCost + hardwareCost) / 5000)`

**Purpose**: Calculate total estimated mandays for a feature (FR-007)

**Test Cases**:

- `(10000, 0)` → `2`
- `(12500, 0)` → `2` (floor rounds down)
- `(0, 0)` → `0`
- `(50000000, 0)` → `10000` (large numbers)
- `(12345.67, 0)` → `2` (fractional costs)

---

#### 2. `calculateRoleAllocations(featureType: FeatureType, estimatedMD: number): RoleAllocation`

**Purpose**: Calculate SA, QA, BE, Support, Rest MD, and Support+Rest MD based on feature type (FR-009 to FR-011, FR-016 to FR-020)

**Returns**:

```typescript
interface RoleAllocation {
  sa: number;
  qa: number;
  be: number;
  support: number;
  restMD: number;
  supportPlusRest: number;
}
```

**Percentages by Type** (from constants.ts):

```typescript
const ROLE_PERCENTAGES: Record<FeatureType, RolePercentages> = {
  "Use Case / API": { sa: 0.3, qa: 0.0, be: 0.0, support: 0.7 },
  "Topic / Event": { sa: 0.3, qa: 0.0, be: 0.0, support: 0.7 },
  "Data Synchronization": { sa: 0.3, qa: 0.2, be: 0.2, support: 0.3 },
  "Batch / File": { sa: 0.3, qa: 0.2, be: 0.2, support: 0.3 },
  "ESB Feature": { sa: 0.2, qa: 0.3, be: 0.2, support: 0.3 },
};
```

**Algorithm**:

1. For each role, calculate: `roleMD = Math.floor(estimatedMD * percentage)`
2. Calculate: `restMD = estimatedMD - (sa + qa + be + support)`
3. Calculate: `supportPlusRest = support + restMD`

**Test Cases**:

- `('Use Case / API', 2)` → `{ sa: 0, qa: 0, be: 0, support: 1, restMD: 1, supportPlusRest: 2 }`
- `('Data Synchronization', 10)` → `{ sa: 3, qa: 2, be: 2, support: 3, restMD: 0, supportPlusRest: 3 }`
- `('ESB Feature', 5)` → `{ sa: 1, qa: 1, be: 1, support: 1, restMD: 1, supportPlusRest: 2 }`

---

#### 3. `calculateTotals(features: Feature[]): RoleAllocation`

**Purpose**: Sum all role MD values across features for the Total row (FR-014)

**Algorithm**:

1. For each feature, calculate `calculateRoleAllocations(feature.type, estimatedMD)`
2. Sum each role column: `total.sa = sum(all features' sa)`, etc.

**Test Cases**:

- Empty array → all zeros
- Multiple features → correct sums for each column

---

#### 4. `validateCostInput(value: string): { valid: boolean; error?: string; numericValue: number }`

**Purpose**: Validate cost input fields (FR-005, FR-006)

**Rules**:

- Empty string → `{ valid: true, error: undefined, numericValue: 0 }`
- Valid non-negative number → `{ valid: true, error: undefined, numericValue: parsed }`
- Negative number → `{ valid: false, error: 'Cost cannot be negative', numericValue: 0 }`
- Non-numeric → `{ valid: false, error: 'Must be a valid number', numericValue: 0 }`

**Test Cases**:

- `''` → valid, 0
- `'1000'` → valid, 1000
- `'-500'` → invalid, error message
- `'abc'` → invalid, error message

---

## Type Definitions (src/lib/types.ts)

```typescript
export type FeatureType =
  | "Use Case / API"
  | "Topic / Event"
  | "Data Synchronization"
  | "Batch / File"
  | "ESB Feature";

export interface Feature {
  id: string;
  type: FeatureType | "";
  projectCost: number;
  hardwareCost: number;
}

export interface RoleAllocation {
  sa: number;
  qa: number;
  be: number;
  support: number;
  restMD: number;
  supportPlusRest: number;
}

export interface RolePercentages {
  sa: number;
  qa: number;
  be: number;
  support: number;
}

export interface ValidationResult {
  valid: boolean;
  error?: string;
  numericValue: number;
}
```

---

## Constants (src/lib/constants.ts)

```typescript
export const MD_DIVISOR = 5000;

export const FEATURE_TYPES: FeatureType[] = [
  "Use Case / API",
  "Topic / Event",
  "Data Synchronization",
  "Batch / File",
  "ESB Feature",
];

export const ROLE_PERCENTAGES: Record<FeatureType, RolePercentages> = {
  "Use Case / API": { sa: 0.3, qa: 0.0, be: 0.0, support: 0.7 },
  "Topic / Event": { sa: 0.3, qa: 0.0, be: 0.0, support: 0.7 },
  "Data Synchronization": { sa: 0.3, qa: 0.2, be: 0.2, support: 0.3 },
  "Batch / File": { sa: 0.3, qa: 0.2, be: 0.2, support: 0.3 },
  "ESB Feature": { sa: 0.2, qa: 0.3, be: 0.2, support: 0.3 },
};

export const ROLE_LABELS = {
  sa: "System Analyst",
  qa: "Quality Assurance",
  be: "Back-End Developer",
  support: "Client Support Operation",
};
```

---

## Testing Strategy

### Unit Tests (tests/unit/calculations.test.ts)

**Coverage**: 100% of all functions in `src/lib/calculations.ts` (Constitution Principle VI)

**Framework**: Vitest with describe/it/expect assertions

**Test Structure**:

```typescript
describe("calculateEstimatedMD", () => {
  it("calculates MD using floor((costs) / 5000)", () => {
    expect(calculateEstimatedMD(10000, 0)).toBe(2);
  });

  it("rounds down fractional results", () => {
    expect(calculateEstimatedMD(12500, 0)).toBe(2);
  });

  it("handles zero costs", () => {
    expect(calculateEstimatedMD(0, 0)).toBe(0);
  });

  // ... more test cases covering edge cases
});

describe("calculateRoleAllocations", () => {
  it("calculates Use Case / API percentages correctly", () => {
    const result = calculateRoleAllocations("Use Case / API", 2);
    expect(result).toEqual({
      sa: 0,
      qa: 0,
      be: 0,
      support: 1,
      restMD: 1,
      supportPlusRest: 2,
    });
  });

  // ... test all 5 feature types with various MD values
});

describe("validateCostInput", () => {
  it("treats empty string as 0", () => {
    const result = validateCostInput("");
    expect(result.valid).toBe(true);
    expect(result.numericValue).toBe(0);
  });

  it("rejects negative numbers", () => {
    const result = validateCostInput("-500");
    expect(result.valid).toBe(false);
    expect(result.error).toContain("negative");
  });

  // ... more validation test cases
});
```

**Test Data**: Include edge cases from spec.md (zero costs, large numbers, fractional costs, rounding edge cases)

---

### Component Tests (tests/components/App.test.tsx)

**Coverage**: At least one end-to-end test covering main user flow (Constitution Principle VI)

**Framework**: Vitest + React Testing Library

**Test Scenarios** (aligned with User Stories):

#### Test 1: Calculate Single Feature (User Story 1)

```typescript
it("calculates and displays MD for a single feature", async () => {
  render(<App />);

  // Enter project name
  const projectNameInput = screen.getByLabelText(/project name/i);
  await userEvent.type(projectNameInput, "API Integration Project");

  // Add a feature
  const addButton = screen.getByText(/add feature/i);
  await userEvent.click(addButton);

  // Select feature type
  const typeDropdown = screen.getByRole("combobox", { name: /feature type/i });
  await userEvent.selectOptions(typeDropdown, "Use Case / API");

  // Enter costs
  const projectCostInput = screen.getByLabelText(/project cost/i);
  await userEvent.type(projectCostInput, "10000");

  // Verify estimated MD appears
  expect(screen.getByText(/2 MD/i)).toBeInTheDocument();

  // Verify output table shows correct role allocations
  expect(screen.getByRole("table", { name: /manday/i })).toBeInTheDocument();
  // Check table cells for SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2
});
```

#### Test 2: Real-time Reactivity (User Story 1, Scenario 2)

```typescript
it("updates output table when costs change", async () => {
  // Setup: render app, add feature with 10000 cost
  // Action: change project cost to 15000
  // Assert: estimated MD updates to 3, table recalculates
});
```

#### Test 3: Multiple Features with Totals (User Story 2)

```typescript
it("displays total row summing all features", async () => {
  // Add 3 features with different types and costs
  // Verify output table has 3 feature rows + 1 total row
  // Verify total row sums match expected values
});
```

#### Test 4: Input Validation (User Story 3)

```typescript
it("shows error for negative cost", async () => {
  // Enter -500 in hardware cost
  // Blur field
  // Verify error message appears
});
```

---

## Implementation Order (Phased Approach)

### Phase 0: Project Setup

1. Initialize Vite React TypeScript project
2. Install dependencies: react, typescript, vite, vitest, @testing-library/react
3. Configure tsconfig.json with strict mode
4. Configure vite.config.ts with Vitest plugin
5. Create directory structure: src/{components,lib}, tests/{unit,components}

### Phase 1: Pure Functions + Unit Tests (TDD)

1. Create `src/lib/types.ts` with all interfaces
2. Create `src/lib/constants.ts` with feature types and percentages
3. Write unit tests in `tests/unit/calculations.test.ts` (RED phase)
4. Implement `calculateEstimatedMD()` → tests pass (GREEN)
5. Implement `calculateRoleAllocations()` → tests pass (GREEN)
6. Implement `calculateTotals()` → tests pass (GREEN)
7. Implement `validateCostInput()` → tests pass (GREEN)
8. Verify 100% coverage: `npm run test:coverage`

### Phase 2: Basic Components (User Story 1 - MVP)

1. Create `App.tsx` with state management
2. Create `ProjectForm.tsx` (simple text input)
3. Create `FeatureInputTable.tsx` with "Add Feature" button
4. Create `FeatureRow.tsx` with dropdown and cost inputs (no validation yet)
5. Create `MandayTable.tsx` showing calculated results
6. Wire up data flow: input → state → calculations → output
7. Manual test: Enter one feature, verify calculations display correctly

### Phase 3: Component Test (User Story 1 validation)

1. Write `tests/components/App.test.tsx` with Test 1 (single feature flow)
2. Ensure test passes → validates end-to-end integration
3. Add Test 2 (real-time reactivity)

### Phase 4: Multiple Features (User Story 2)

1. Enhance `FeatureInputTable.tsx` to handle array of features
2. Add delete button to `FeatureRow.tsx`
3. Implement Total row logic in `MandayTable.tsx`
4. Write component test (Test 3) for multiple features

### Phase 5: Validation & Polish (User Story 3)

1. Add validation logic to `FeatureRow.tsx` using `validateCostInput()`
2. Display inline error messages
3. Highlight invalid fields with CSS
4. Write component test (Test 4) for validation
5. Add empty state handling ("No features added yet")

### Phase 6: Accessibility & Styling

1. Add ARIA labels to all form fields
2. Ensure keyboard navigation works (tab order, enter to submit)
3. Add CSS for table styling (headers, borders, total row bold)
4. Test with keyboard-only navigation
5. Verify table semantics (`<table>`, `<thead>`, `<th scope="col">`)

### Phase 7: User Story 4 - Role Selection and Output Navigation

**Purpose**: Add role selection checkboxes and separate Output view after US1–US3 are complete.

**Implementation Steps**:

1. Add `selectedRoles: RoleKey[]` and `activeView: 'input' | 'output'` to App state
2. Create RoleSelector component with four checkboxes and validation logic
3. Create OutputView wrapper component with 'Back to Input' button
4. Update MandayTable to accept `selectedRoles` prop and conditionally render role columns
5. Add navigation buttons in App: 'View Output' (shown in input view) and wire to OutputView
6. Write component tests ensuring:
   - Role checkboxes control visible columns in output table
   - At least one role must remain selected at all times
   - Navigation between Input and Output views preserves all state (features, costs, project name)
   - Output table updates immediately when roles are toggled
7. Update App to conditionally render either input components or OutputView based on activeView state

**Testing Requirements**:

- Test US4 Scenario 1: Uncheck a role (e.g., QA) and verify column hidden in Output view
- Test US4 Scenario 2: Attempt to uncheck last role and verify prevention + message
- Test US4 Scenario 3: Navigate Input → Output → Input with data, verify state preserved

---

## Mapping Spec to Implementation

### Functional Requirements → Code Mapping

| Requirement                     | Implementation                                                           |
| ------------------------------- | ------------------------------------------------------------------------ |
| FR-001: Project name input      | `ProjectForm.tsx` renders `<input type="text" required>`                 |
| FR-002: Add multiple features   | `App.tsx` state: `features: Feature[]`; `addFeature()` handler           |
| FR-003: Feature type dropdown   | `FeatureRow.tsx` renders `<select>` with 5 options from `FEATURE_TYPES`  |
| FR-004: Cost input fields       | `FeatureRow.tsx` renders two `<input type="number">` fields              |
| FR-005: Empty = 0               | `validateCostInput('')` returns `numericValue: 0`                        |
| FR-006: Non-negative validation | `validateCostInput()` checks `< 0`; `FeatureRow.tsx` displays error      |
| FR-007: MD formula              | `calculateEstimatedMD()` implements `Math.floor((c1 + c2) / 5000)`       |
| FR-008: Math.floor() usage      | All calculation functions use `Math.floor()`                             |
| FR-009: Role percentages        | `calculateRoleAllocations()` applies `ROLE_PERCENTAGES` config           |
| FR-010: Rest MD formula         | `calculateRoleAllocations()` computes `E - (sa + qa + be + support)`     |
| FR-011: Support + Rest MD       | `calculateRoleAllocations()` computes `support + restMD`                 |
| FR-012: Output table columns    | `MandayTable.tsx` renders 7-column `<table>`                             |
| FR-013: One row per feature     | `MandayTable.tsx` maps `features.map((f) => <tr>...)`                    |
| FR-014: Total row               | `MandayTable.tsx` calls `calculateTotals()` and renders footer row       |
| FR-015: Real-time reactivity    | React re-renders on state change; calculations run each render           |
| FR-016-020: Type percentages    | `ROLE_PERCENTAGES` constant defines all 5 types' allocations             |
| FR-021: Role selection control  | `RoleSelector.tsx` renders 4 checkboxes, all selected by default         |
| FR-022: At least one role       | `RoleSelector.tsx` prevents unchecking last role, shows inline message   |
| FR-023: Respect role selection  | `MandayTable.tsx` conditionally renders columns based on `selectedRoles` |
| FR-024: Input/Output navigation | `App.tsx` manages `activeView` state; buttons toggle between views       |
| FR-025: Calculations unchanged  | Role selection only affects `MandayTable` display, not calculation logic |

### User Stories → Components Mapping

| User Story                      | Primary Components                          | Key Functions                                      |
| ------------------------------- | ------------------------------------------- | -------------------------------------------------- |
| US1: Single feature calculation | `App`, `FeatureRow`, `MandayTable`          | `calculateEstimatedMD`, `calculateRoleAllocations` |
| US2: Multiple features          | `FeatureInputTable`, `MandayTable`          | `calculateTotals`                                  |
| US3: Validation                 | `FeatureRow` (error display)                | `validateCostInput`                                |
| US4: Role selection & Output    | `RoleSelector`, `OutputView`, `MandayTable` | (UI state management, no new calc functions)       |

---

## Performance Considerations

**Target**: <100ms calculation time for 20 features (SC-003)

**Analysis**:

- Each feature requires 2-3 pure function calls (calculateEstimatedMD + calculateRoleAllocations)
- Pure functions with simple math operations: ~0.001ms per call
- 20 features × 3 calls × 0.001ms = 0.06ms (well under 100ms)
- React re-render overhead: ~10-50ms for table with 20 rows
- **Conclusion**: No optimization needed; synchronous calculations sufficient

**If needed** (for 100+ features):

- Use `useMemo()` to cache total calculations
- Use `React.memo()` to prevent unnecessary row re-renders
- Consider Web Workers only if exceeds 50ms (per Constitution constraint)

---

## Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/user-event": "^14.5.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

**Rationale**:

- **React 18+**: Per Constitution technical constraints
- **TypeScript 5+**: Strict mode support, latest type features
- **Vite**: Fast dev server, optimal for frontend-only SPA
- **Vitest**: Native Vite integration, faster than Jest for this stack
- **React Testing Library**: Standard for React component testing
- **No CSS framework**: Keep codebase small per user request; use CSS modules or plain CSS

---

## Acceptance Checklist

Before marking implementation complete, verify:

- [ ] All 20 functional requirements (FR-001 to FR-020) implemented
- [ ] All 7 constitution principles satisfied (Constitution Check passes)
- [ ] All unit tests pass with 100% coverage of calculation functions
- [ ] Component test(s) pass for main user flow (US1 scenario)
- [ ] All 3 user stories (P1, P2, P3) implemented and testable
- [ ] Input validation works per FR-006 (negative/invalid inputs show errors)
- [ ] Empty cost fields treated as 0 per FR-005
- [ ] Math.floor() used consistently per FR-008
- [ ] Output table displays all 7 columns per FR-012
- [ ] Total row sums correctly per FR-014
- [ ] Real-time reactivity works per FR-015 (input change → immediate recalculation)
- [ ] Performance: 20 features calculate in <100ms per SC-003
- [ ] Accessibility: keyboard navigation works, ARIA labels present
- [ ] Table semantics: proper `<table>`, `<th>`, `<td>` structure
- [ ] TypeScript compiles with no errors (strict mode)
- [ ] No ESLint errors

---

## Next Steps

1. Run `/speckit.tasks` to generate detailed task breakdown from this plan
2. Execute Phase 0 (project setup) to create repository structure
3. Follow TDD workflow: write tests → implement → verify (Phase 1)
4. Build components incrementally (Phases 2-5)
5. Polish and validate against acceptance checklist (Phase 6)
