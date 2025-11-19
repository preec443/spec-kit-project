# Implementation Plan: Integration Effort Estimator

**Branch**: `002-effort-estimator` | **Date**: 2025-11-19 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/002-effort-estimator/spec.md`

## Summary

The Integration Effort Estimator is a frontend-only React/TypeScript single-page application that calculates mandays (MD) from integration feature costs and displays role-based effort distribution across separate Input and Output views. Users input project details and multiple features with costs in the Input view, select which roles are in scope via checkboxes, then navigate to the Output view to see calculated MD allocated across System Analyst, Quality Assurance, Back-End Developer, and Client Support Operation roles. All calculations use pure functions with Math.floor() for consistent integer rounding. The application maintains state across view transitions and enforces input validation with inline error feedback.

## Technical Context

**Language/Version**: TypeScript 5.3+ with strict mode enabled  
**Primary Dependencies**: React 18+ application built with the team's preferred React build tool (e.g., Create React App, Next.js, or Webpack)  
**Storage**: In-memory React state only; no persistence in v1.0  
**Testing**: Jest + React Testing Library (or any React-compatible testing framework)  
**Target Platform**: Modern web browsers (Chrome, Firefox, Safari, Edge from last 2 years)  
**Project Type**: Frontend-only single-page application  
**Performance Goals**: <100ms calculation time for 20 features; <2s initial page load on 3G  
**Constraints**: No blocking operations >50ms; frontend-only (no backend/database); accessible keyboard navigation  
**Scale/Scope**: Up to 50 integration features per project; single-user session-based state

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

- [x] **Frontend-Only Architecture**: Verify no backend services planned for v1.0 ✅ React SPA with in-memory state only
- [x] **Pure Function Business Logic**: Calculation module uses pure functions only (no side effects) ✅ `src/lib/calculations.ts` with exported pure functions
- [x] **Table-Based UI**: Design uses table/grid layouts for inputs and outputs ✅ HTML tables for feature input rows and output manday table
- [x] **Input Validation**: All currency fields have non-negative validation; empty treated as 0 ✅ Validation logic in form handlers with inline error display
- [x] **Integer MD with Round-Down**: Formula uses `Math.floor()` for manday calculations ✅ All calculation functions use Math.floor()
- [x] **Test-First Development**: Unit tests for all calculations + component test for main flow ✅ Jest unit tests + React Testing Library component tests planned
- [x] **Documentation Separation**: Constitution, spec, plan, and tasks are distinct artifacts ✅ This plan is separate from spec.md

## Project Structure

### Documentation (this feature)

```text
specs/002-effort-estimator/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (if needed)
├── data-model.md        # Phase 1 output (if needed)
├── quickstart.md        # Phase 1 output (development guide)
├── contracts/           # Phase 1 output (API contracts)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```text
src/
├── components/
│   ├── App.tsx                    # Root component, manages application state and view routing
│   ├── InputView.tsx              # Input page container
│   ├── OutputView.tsx             # Output page container
│   ├── ProjectForm.tsx            # Project name input
│   ├── RoleSelector.tsx           # Role visibility checkboxes with validation
│   ├── FeatureTable.tsx           # Feature input table with add/delete
│   ├── FeatureRow.tsx             # Single feature row (type, costs, estimated MD)
│   └── MandayTable.tsx            # Output table with role-based MD and totals
├── lib/
│   ├── calculations.ts            # Pure functions for all MD and role allocation calculations
│   ├── types.ts                   # TypeScript interfaces (Feature, RoleAllocation, etc.)
│   └── constants.ts               # Feature types, role percentages, role labels
├── App.css                        # Global styles
├── main.tsx                       # React app entry point
└── index.html                     # HTML shell

tests/
├── unit/
│   └── calculations.test.ts       # Unit tests for all calculation functions (100% coverage)
└── components/
    └── App.test.tsx               # Integration test: input → output flow

package.json                       # Dependencies: react, typescript, testing framework
tsconfig.json                      # TypeScript strict mode config
```

**Structure Decision**: Frontend-only web application using the team's preferred React build tool. All business logic isolated in `src/lib/calculations.ts` as pure functions (Constitution Principle II). Components separated by view responsibility: InputView and OutputView as top-level containers, with shared child components. Tests co-located by type (unit vs component) for clarity. No backend directories since this is v1.0 frontend-only (Constitution Principle I).

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations. All principles are satisfied:

- Frontend-only architecture with no backend
- Pure functions for all calculations
- Table-based accessible UI with separate Input/Output views (Constitution v1.0.1)
- Complete input validation with inline errors
- Math.floor() for all rounding
- Comprehensive test coverage planned
- Documentation properly separated

---

## Architecture & Components

### Component Hierarchy

```
App
├── InputView (when activeView === 'input')
│   ├── ProjectForm (project name input)
│   ├── RoleSelector (4 role checkboxes with validation)
│   ├── FeatureTable
│   │   ├── FeatureRow[] (one per feature)
│   │   │   ├── Feature type dropdown
│   │   │   ├── Project cost input
│   │   │   ├── Hardware cost input
│   │   │   ├── Estimated MD display (calculated)
│   │   │   └── Delete button
│   │   └── Add Feature button
│   └── View Output button
│
└── OutputView (when activeView === 'output')
    ├── Back to Input button
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

type FeatureType =
  | "Use Case / API"
  | "Topic / Event"
  | "Data Synchronization"
  | "Batch / File"
  | "ESB Feature";

type RoleKey = "sa" | "qa" | "be" | "support";

interface AppState {
  projectName: string;
  features: Feature[];
  selectedRoles: RoleKey[]; // at least one role, default ['sa','qa','be','support']
  activeView: "input" | "output"; // default 'input'
}
```

**Implementation Notes**:

- `selectedRoles` is updated by the RoleSelector component via callbacks
- `activeView` is toggled by navigation buttons ('View Output' and 'Back to Input')
- All state persists across view transitions (no data loss when navigating)

**Data Flow**:

1. User types in input fields → onChange handlers update state
2. State change triggers re-render
3. Components call pure functions from `calculations.ts` to derive outputs
4. Derived data (estimated MD, role allocations, totals) calculated on each render
5. View transitions update `activeView` without clearing feature data

No external state management library needed (per Constitution: avoid complexity unless justified).

### Component Details

#### 1. App.tsx (Root Component)

**Responsibilities**:

- Manage application state: projectName, features, selectedRoles, activeView
- Provide handlers: `addFeature()`, `updateFeature(id, field, value)`, `deleteFeature(id)`
- Provide handlers: `updateSelectedRoles(roles)`, `setActiveView(view)`
- Conditionally render InputView or OutputView based on activeView
- Pass state and handlers down to child components

**Props**: None (root component)

**State**:

```typescript
const [projectName, setProjectName] = useState<string>("");
const [features, setFeatures] = useState<Feature[]>([]);
const [selectedRoles, setSelectedRoles] = useState<RoleKey[]>([
  "sa",
  "qa",
  "be",
  "support",
]);
const [activeView, setActiveView] = useState<"input" | "output">("input");
```

---

#### 2. InputView.tsx (Input Page Container)

**Responsibilities**:

- Render Input page layout
- Compose ProjectForm, RoleSelector, FeatureTable components
- Provide "View Output" button

**Props**:

```typescript
interface InputViewProps {
  projectName: string;
  onProjectNameChange: (name: string) => void;
  features: Feature[];
  selectedRoles: RoleKey[];
  onAddFeature: () => void;
  onUpdateFeature: (id: string, field: keyof Feature, value: any) => void;
  onDeleteFeature: (id: string) => void;
  onUpdateSelectedRoles: (roles: RoleKey[]) => void;
  onNavigateToOutput: () => void;
}
```

**Implementation Notes**:

- Simple layout component wrapping child components
- "View Output" button calls `onNavigateToOutput()`

---

#### 3. OutputView.tsx (Output Page Container)

**Responsibilities**:

- Render Output page layout
- Compose MandayTable component
- Provide "Back to Input" button

**Props**:

```typescript
interface OutputViewProps {
  features: Feature[];
  selectedRoles: RoleKey[];
  onNavigateToInput: () => void;
}
```

**Implementation Notes**:

- "Back to Input" button calls `onNavigateToInput()`
- MandayTable receives features and selectedRoles for rendering

---

#### 4. ProjectForm.tsx

**Responsibilities**:

- Render project name input field
- Display required field indicator

**Props**:

```typescript
interface ProjectFormProps {
  projectName: string;
  onProjectNameChange: (name: string) => void;
}
```

**Implementation Notes**:

- Single text input with label "Project Name"
- Mark as required with asterisk or ARIA attribute
- No complex validation (name can be any string)

---

#### 5. RoleSelector.tsx

**Responsibilities**:

- Render four checkboxes for System Analyst, Quality Assurance, Back-End Developer, and Client Support Operation
- Ensure at least one checkbox remains selected by preventing the user from unchecking the last selected role
- Display inline helper message when user attempts to uncheck last role

**Props**:

```typescript
interface RoleSelectorProps {
  selectedRoles: RoleKey[];
  onChange: (nextRoles: RoleKey[]) => void;
}
```

**Implementation Notes**:

- All four checkboxes MUST be checked by default (handled in App.tsx initial state)
- When user attempts to uncheck the last remaining role, prevent the action and display inline message: "At least one role must be selected." (per FR-012)
- Use semantic HTML with `<input type="checkbox">` and associated `<label>` elements
- Apply ARIA attributes for accessibility

**Validation Logic**:

```typescript
const handleRoleToggle = (role: RoleKey) => {
  const isSelected = selectedRoles.includes(role);
  if (isSelected && selectedRoles.length === 1) {
    // Prevent unchecking last role, show message
    setErrorMessage("At least one role must be selected.");
    return;
  }
  // Update roles...
};
```

---

#### 6. FeatureTable.tsx

**Responsibilities**:

- Render table with header row (Feature Type, Project Cost (THB), Hardware Cost (THB), Estimated MD, Actions)
- Map features array to FeatureRow components
- Display "Add Feature" button below table

**Props**:

```typescript
interface FeatureTableProps {
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
- Empty state: if no features, show message "No features added yet. Click 'Add Feature' to begin."

---

#### 7. FeatureRow.tsx

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
- Negative costs → display inline error message, highlight field with red border
- Non-numeric input → display inline error message
- Validation triggers on blur, not per-character

**Estimated MD Display**:

```typescript
const estimatedMD = calculateEstimatedMD(
  feature.projectCost || 0,
  feature.hardwareCost || 0
);
```

**Implementation Notes**:

- If feature.type is empty string, display "—" or "Select type" instead of MD value
- Delete button removes feature from list

---

#### 8. MandayTable.tsx

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

- MandayTable MUST always display 'Feature', 'Rest MD', and 'Support + Rest MD' columns
- It MUST conditionally render 'SA (MD)', 'QA (MD)', 'BE (MD)', and 'Support (MD)' columns depending on whether each role key is present in selectedRoles
- Use semantic HTML `<table>` with proper headers
- Feature column shows feature type name (e.g., "Use Case / API")
- All MD values displayed as integers (no decimals)
- Total row uses bold or different styling for distinction
- Empty state: if no features, show message "No features added yet. Return to Input to add features."

**Calculation Flow**:

```typescript
// For each feature:
const estimatedMD = calculateEstimatedMD(projectCost, hardwareCost);
const roleAllocations = calculateRoleAllocations(feature.type, estimatedMD);
// roleAllocations = { sa, qa, be, support, restMD, supportPlusRest }

// Total row:
const totals = calculateTotals(features);
```

---

## Pure Calculation Functions (src/lib/calculations.ts)

All functions are pure (no side effects, deterministic output from inputs). Each function has JSDoc comments documenting formula and behavior.

### Function Specifications

#### 1. `calculateEstimatedMD(projectCost: number, hardwareCost: number): number`

**Formula**: `MD = Math.floor((projectCost + hardwareCost) / 5000)`

**Purpose**: Calculate total estimated mandays for a feature (FR-007)

**Test Cases**:

- `(10000, 0)` → `2`
- `(12500, 0)` → `2` (floor rounds down from 2.5)
- `(0, 0)` → `0`
- `(50000, 0)` → `10`
- `(12345, 0)` → `2` (fractional costs)

---

#### 2. `calculateRoleAllocations(featureType: FeatureType, estimatedMD: number): RoleAllocation`

**Purpose**: Calculate SA, QA, BE, Support, Rest MD, and Support+Rest MD based on feature type (FR-020 to FR-023)

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

**Purpose**: Sum all role MD values across features for the Total row (FR-025)

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

**Framework**: Jest (or any React-compatible test runner) with describe/it/expect assertions

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

**Framework**: Jest + React Testing Library (or equivalent)

**Test Scenarios** (aligned with User Stories):

#### Test 1: Input and Calculate Features (User Story 1)

```typescript
it("calculates and displays MD for features in Input view", async () => {
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
  expect(screen.getByText(/2/i)).toBeInTheDocument(); // MD = 2
});
```

#### Test 2: Role Selection Validation (User Story 2)

```typescript
it("prevents unchecking the last remaining role", async () => {
  render(<App />);

  // Uncheck 3 roles, leaving only one
  const qaCheckbox = screen.getByLabelText(/quality assurance/i);
  await userEvent.click(qaCheckbox);
  // ... uncheck BE and Support

  // Try to uncheck last role (SA)
  const saCheckbox = screen.getByLabelText(/system analyst/i);
  await userEvent.click(saCheckbox);

  // Verify checkbox still checked and error message shown
  expect(saCheckbox).toBeChecked();
  expect(
    screen.getByText(/at least one role must be selected/i)
  ).toBeInTheDocument();
});
```

#### Test 3: Navigation and Output View (User Story 3)

```typescript
it("navigates to Output view and displays role allocations", async () => {
  render(<App />);

  // Add feature with data
  // ... (setup feature input)

  // Navigate to Output
  const viewOutputButton = screen.getByText(/view output/i);
  await userEvent.click(viewOutputButton);

  // Verify Output view displayed
  expect(screen.getByRole("table")).toBeInTheDocument();
  expect(screen.getByText(/back to input/i)).toBeInTheDocument();

  // Verify role allocations displayed
  // ... check table cells for expected values
});
```

#### Test 4: Input Validation (User Story 4)

```typescript
it("shows error for negative cost", async () => {
  render(<App />);

  // Add feature
  // ... (setup)

  // Enter negative cost
  const projectCostInput = screen.getByLabelText(/project cost/i);
  await userEvent.type(projectCostInput, "-500");
  await userEvent.tab(); // Trigger blur

  // Verify error message appears
  expect(screen.getByText(/cost cannot be negative/i)).toBeInTheDocument();
});
```

#### Test 5: State Preservation Across Views

```typescript
it("preserves feature data when navigating between views", async () => {
  render(<App />);

  // Add features with data
  // ... (setup multiple features)

  // Navigate to Output
  await userEvent.click(screen.getByText(/view output/i));

  // Navigate back to Input
  await userEvent.click(screen.getByText(/back to input/i));

  // Verify all data preserved
  // ... check that feature rows still exist with correct values
});
```

---

## Implementation Order (Phased Approach)

### Phase 0: Project Setup

1. Initialize React TypeScript project using the team's preferred build tool
2. Install dependencies: react, typescript, testing framework, @testing-library/react
3. Configure tsconfig.json with strict mode
4. Configure test framework setup
5. Create directory structure: src/{components,lib}, tests/{unit,components}

### Phase 1: Pure Functions + Unit Tests (TDD)

1. Create `src/lib/types.ts` with all interfaces
2. Create `src/lib/constants.ts` with feature types and percentages
3. Write unit tests in `tests/unit/calculations.test.ts` (RED phase)
4. Implement `calculateEstimatedMD()` → tests pass (GREEN)
5. Implement `calculateRoleAllocations()` → tests pass (GREEN)
6. Implement `calculateTotals()` → tests pass (GREEN)
7. Implement `validateCostInput()` → tests pass (GREEN)
8. Verify 100% coverage: run test coverage command

### Phase 2: Input View Components (User Story 1 - Basic Input)

1. Create `App.tsx` with state management
2. Create `InputView.tsx` layout component
3. Create `ProjectForm.tsx` (simple text input)
4. Create `FeatureTable.tsx` with "Add Feature" button
5. Create `FeatureRow.tsx` with dropdown and cost inputs (no validation yet)
6. Wire up data flow: input → state → calculations → display
7. Display Estimated MD in each FeatureRow
8. Manual test: Enter features, verify MD calculation displays correctly

### Phase 3: Role Selection (User Story 2)

1. Create `RoleSelector.tsx` with four checkboxes
2. Add selectedRoles state to App.tsx (default all checked)
3. Implement "at least one role" validation logic
4. Display inline error message when trying to uncheck last role
5. Test role selection behavior

### Phase 4: Output View (User Story 3)

1. Create `OutputView.tsx` layout component
2. Create `MandayTable.tsx` showing calculated results
3. Implement dynamic column visibility based on selectedRoles
4. Implement Total row logic in `MandayTable.tsx`
5. Add activeView state to App.tsx
6. Add "View Output" button in InputView
7. Add "Back to Input" button in OutputView
8. Wire up view navigation
9. Test state preservation across view transitions

### Phase 5: Input Validation (User Story 4)

1. Add validation logic to `FeatureRow.tsx` using `validateCostInput()`
2. Display inline error messages
3. Highlight invalid fields with CSS (red border)
4. Write component test for validation
5. Add empty state handling ("No features added yet")
6. Test all validation scenarios

### Phase 6: Component Tests & Integration

1. Write `tests/components/App.test.tsx` with Test 1 (input and calculate)
2. Write Test 2 (role selection validation)
3. Write Test 3 (navigation and output view)
4. Write Test 4 (input validation)
5. Write Test 5 (state preservation)
6. Ensure all tests pass → validates end-to-end integration

### Phase 7: Accessibility & Styling

1. Add ARIA labels to all form fields
2. Ensure keyboard navigation works (tab order, enter to submit)
3. Add CSS for table styling (headers, borders, total row bold)
4. Test with keyboard-only navigation
5. Verify table semantics (`<table>`, `<thead>`, `<th scope="col">`)
6. Add focus indicators for all interactive elements
7. Test with screen reader (VoiceOver/NVDA)

### Phase 8: Polish & Performance

1. Add performance benchmarks for 20 and 50 features
2. Optimize rendering if needed (React.memo, useMemo)
3. Add responsive styles (if time permits)
4. Cross-browser testing
5. Build production bundle and verify performance
6. Final QA and bug fixes

---

## Mapping Spec to Implementation

### Functional Requirements → Code Mapping

| Requirement                      | Implementation                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------- |
| FR-001: Project name input       | `ProjectForm.tsx` renders `<input type="text" required>`                        |
| FR-002: Add multiple features    | `App.tsx` state: `features: Feature[]`; `addFeature()` handler                  |
| FR-003: Feature type dropdown    | `FeatureRow.tsx` renders `<select>` with 5 options from `FEATURE_TYPES`         |
| FR-004: Cost input fields        | `FeatureRow.tsx` renders two `<input type="number">` fields                     |
| FR-005: Empty = 0                | `validateCostInput('')` returns `numericValue: 0`                               |
| FR-006: Non-negative validation  | `validateCostInput()` checks `< 0`; `FeatureRow.tsx` displays error             |
| FR-007: MD formula               | `calculateEstimatedMD()` implements `Math.floor((c1 + c2) / 5000)`              |
| FR-008: Math.floor() usage       | All calculation functions use `Math.floor()`                                    |
| FR-009: Role checkboxes          | `RoleSelector.tsx` renders 4 checkboxes                                         |
| FR-010: All roles default        | `App.tsx` initial state: `['sa','qa','be','support']`                           |
| FR-011: At least one role        | `RoleSelector.tsx` prevents unchecking last role                                |
| FR-012: Validation message       | `RoleSelector.tsx` displays "At least one role must be selected."               |
| FR-013: Role controls visibility | `MandayTable.tsx` conditionally renders columns based on `selectedRoles`        |
| FR-014: View navigation          | `App.tsx` manages `activeView` state; buttons toggle between views              |
| FR-015: Manday table columns     | `MandayTable.tsx` renders 7-column `<table>`                                    |
| FR-016: Conditional columns      | `MandayTable.tsx` checks `selectedRoles.includes(role)` before rendering column |
| FR-017: One row per feature      | `MandayTable.tsx` maps `features.map((f) => <tr>...)`                           |
| FR-018: Total row                | `MandayTable.tsx` calls `calculateTotals()` and renders footer row              |
| FR-019: Preserve data            | `App.tsx` maintains all state across view transitions                           |
| FR-020: Type percentages         | `ROLE_PERCENTAGES` constant defines all 5 types' allocations                    |
| FR-021: Role MD calculation      | `calculateRoleAllocations()` applies `Math.floor(E * percentage)`               |
| FR-022: Rest MD formula          | `calculateRoleAllocations()` computes `E - (sa + qa + be + support)`            |
| FR-023: Support + Rest MD        | `calculateRoleAllocations()` computes `support + restMD`                        |
| FR-024: Integer display          | All components display MD values without decimals                               |
| FR-025: Total row sums           | `calculateTotals()` sums each column across features                            |

### User Stories → Components Mapping

| User Story                   | Primary Components                               | Key Functions                                 |
| ---------------------------- | ------------------------------------------------ | --------------------------------------------- |
| US1: Calculate features      | `App`, `InputView`, `FeatureRow`, `FeatureTable` | `calculateEstimatedMD`                        |
| US2: Control role visibility | `RoleSelector`, `App`                            | (UI state management)                         |
| US3: View role distribution  | `OutputView`, `MandayTable`, `App`               | `calculateRoleAllocations`, `calculateTotals` |
| US4: Handle invalid input    | `FeatureRow` (error display)                     | `validateCostInput`                           |

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
    "typescript": "^5.3.0"
  }
}
```

**Note**: Additional build tool and testing framework dependencies should be added based on your chosen toolchain (e.g., CRA + Jest, Webpack + Jest, etc.).

**Rationale**:

- **React 18+**: Per Constitution technical constraints
- **TypeScript 5+**: Strict mode support, latest type features
- **React Testing Library**: Standard for React component testing
- **Build Tool Flexibility**: Choose any standard React build tool based on team preference
- **Testing Framework Flexibility**: Choose any React-compatible test runner (Jest) based on build tool selection
- **No CSS framework**: Keep codebase small; use CSS modules or plain CSS

---

## Acceptance Checklist

Before marking implementation complete, verify:

- [ ] All 25 functional requirements (FR-001 to FR-025) implemented
- [ ] All 7 constitution principles satisfied (Constitution Check passes)
- [ ] All unit tests pass with 100% coverage of calculation functions
- [ ] Component test(s) pass for all 4 user stories
- [ ] All 4 user stories (US1, US2, US3, US4) implemented and testable
- [ ] Input validation works per FR-006 (negative/invalid inputs show errors)
- [ ] Empty cost fields treated as 0 per FR-005
- [ ] Math.floor() used consistently per FR-008
- [ ] Output table displays dynamic columns per FR-016
- [ ] Role selection enforces "at least one" constraint per FR-011
- [ ] Total row sums correctly per FR-018
- [ ] View navigation preserves all data per FR-019
- [ ] Performance: 20 features calculate in <100ms per SC-003
- [ ] Accessibility: keyboard navigation works, ARIA labels present
- [ ] Table semantics: proper `<table>`, `<th>`, `<td>` structure
- [ ] TypeScript compiles with no errors (strict mode)
- [ ] No linting errors

---

## Next Steps

1. Run `/speckit.tasks` to generate detailed task breakdown from this plan
2. Execute Phase 0 (project setup) to create repository structure
3. Follow TDD workflow: write tests → implement → verify (Phase 1)
4. Build components incrementally (Phases 2-5)
5. Add component tests and validate integration (Phase 6)
6. Polish accessibility and styling (Phase 7)
7. Performance optimization and final QA (Phase 8)
