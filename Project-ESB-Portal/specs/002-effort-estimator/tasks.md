# Tasks: Integration Effort Estimator

**Feature Branch**: `002-effort-estimator`  
**Input**: Design documents from `/specs/002-effort-estimator/`  
**Prerequisites**: plan.md ✅, spec.md ✅

**Tests**: Tests are MANDATORY per Constitution Principle VI (Test-First Development). All calculation functions require 100% unit test coverage; main user flows require component tests.

**Organization**: Tasks are grouped by user story and phase to enable independent implementation and testing.

## Format: `- [ ] [ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Project Structure

This is a **single-project frontend-only application** using:

- **Framework**: React 18+ with TypeScript 5.3+
- **Build Tool**: Team's preferred React build tool (e.g., Create React App, Next.js, Webpack, or similar)
- **Testing**: React-compatible test runner (e.g., Jest) + React Testing Library
- **Paths**: `src/` and `tests/` at repository root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, basic configuration, and directory structure

**Duration**: ~1-2 hours

- [ ] T001 Initialize a React 18+ + TypeScript 5.3+ project using the team's chosen build tool
- [ ] T002 Install dependencies: `npm install react@^18.2.0 react-dom@^18.2.0`
- [ ] T003 Install dev dependencies for testing: `npm install -D @testing-library/react@^14.0.0 @testing-library/user-event@^14.5.0 @types/react @types/react-dom typescript@^5.3.0`
- [ ] T004 Install test runner (e.g., Jest): `npm install -D jest @types/jest ts-jest` (or equivalent for chosen build tool)
- [ ] T005 Configure tsconfig.json with strict mode: `"strict": true, "noUnusedLocals": true, "noUnusedParameters": true`
- [ ] T006 Configure test runner setup (e.g., jest.config.js with ts-jest preset)
- [ ] T007 Create directory structure: `src/components/`, `src/lib/`, `tests/unit/`, `tests/components/`
- [ ] T008 Add test scripts to package.json: `"test": "jest"` (or equivalent command)
- [ ] T009 Create .gitignore with node_modules/, dist/, build/, coverage/, .env

**Checkpoint**: Project structure ready - foundation work can begin

---

## Phase 2: Foundational (Pure Functions & Types)

**Purpose**: Core calculation logic and type definitions that ALL user stories depend on

**⚠️ CRITICAL**: No UI work can begin until this phase is complete. Use TDD: write tests first, then implement.

**Duration**: ~3-4 hours

### Type Definitions & Constants (Foundation)

- [ ] T010 [P] Create src/lib/types.ts with all TypeScript interfaces: Feature, FeatureType, RoleAllocation, RolePercentages, ValidationResult
- [ ] T011 [P] Create src/lib/constants.ts with MD_DIVISOR (5000), FEATURE_TYPES array (5 types), ROLE_PERCENTAGES map, ROLE_LABELS map

### Unit Tests for Calculation Functions (TDD - Write First) ✅

- [ ] T012 [P] Create tests/unit/calculations.test.ts with test suite structure for all 4 calculation functions
- [ ] T013 [P] Write unit tests for calculateEstimatedMD: zero costs, normal division, floor rounding (12500 → 2), large numbers (50000 → 10), fractional costs
- [ ] T014 [P] Write unit tests for calculateRoleAllocations: all 5 feature types, various MD values (0, 2, 5, 10), verify floor rounding per role, verify restMD calculation, verify supportPlusRest sum
- [ ] T015 [P] Write unit tests for calculateTotals: empty array → all zeros, single feature, multiple features with different types, verify column-wise sums
- [ ] T016 [P] Write unit tests for validateCostInput: empty string → valid/0, valid positive number, negative number → error, non-numeric → error, fractional valid

### Implement Calculation Functions (TDD - Make Tests Pass)

- [ ] T017 [P] Implement calculateEstimatedMD() in src/lib/calculations.ts: `Math.floor((projectCost + hardwareCost) / 5000)`
- [ ] T018 [P] Implement calculateRoleAllocations() in src/lib/calculations.ts: apply ROLE_PERCENTAGES with Math.floor, calculate restMD, calculate supportPlusRest
- [ ] T019 [P] Implement calculateTotals() in src/lib/calculations.ts: reduce over features array, sum each role column
- [ ] T020 [P] Implement validateCostInput() in src/lib/calculations.ts: handle empty string, validate non-negative, return ValidationResult object
- [ ] T021 Run test command and verify 100% coverage of src/lib/calculations.ts (all tests pass)

**Checkpoint**: All pure functions tested and working - UI components can now use them safely

---

## Phase 3: User Story 1 - Calculate Effort for Integration Features (Priority: P1) 🎯 MVP

**Goal**: User enters project name and features with costs, sees calculated MD in Input view

**Independent Test**: Enter project name "API Integration", add one "Use Case / API" feature with costs 10000/0, verify Estimated MD displays as 2

**Duration**: ~4-6 hours

### Component Tests for US1 (TDD - Write First) ✅

- [ ] T022 Create tests/components/App.test.tsx with React Testing Library setup
- [ ] T023 [US1] Write component test: "renders empty app with project name input and add feature button"
- [ ] T024 [US1] Write component test: "calculates and displays MD for single feature" (full US1 Scenario 1 flow)
- [ ] T025 [US1] Write component test: "updates MD when costs change" (US1 real-time reactivity)

### Basic Component Structure

- [ ] T026 [P] [US1] Create src/components/App.tsx with state: projectName, features (Feature[]), selectedRoles (['sa','qa','be','support']), activeView ('input')
- [ ] T027 [P] [US1] Create src/components/InputView.tsx with props: projectName, features, selectedRoles, handlers (onProjectNameChange, onAddFeature, onUpdateFeature, onDeleteFeature, onUpdateSelectedRoles, onNavigateToOutput)
- [ ] T028 [P] [US1] Create src/components/OutputView.tsx with props: features, selectedRoles, onNavigateToInput
- [ ] T029 [P] [US1] Create src/components/ProjectForm.tsx with props: projectName, onProjectNameChange
- [ ] T030 [P] [US1] Create src/components/FeatureTable.tsx with props: features, onAddFeature, onUpdateFeature, onDeleteFeature
- [ ] T031 [P] [US1] Create src/components/FeatureRow.tsx with props: feature, onUpdate, onDelete

### Implement ProjectForm Component

- [ ] T032 [US1] Implement ProjectForm.tsx: render text input with label "Project Name", bind value to projectName prop, call onProjectNameChange on input change
- [ ] T033 [US1] Add required indicator (\*) to project name label in ProjectForm.tsx
- [ ] T034 [US1] Add ARIA label to project name input: `aria-label="Project name (required)"`

### Implement FeatureTable Component

- [ ] T035 [US1] Implement FeatureTable.tsx: render semantic HTML `<table>` with `<thead>` containing column headers: Feature Type, Project Cost (THB), Hardware Cost (THB), Estimated MD, Actions
- [ ] T036 [US1] Add table header row with `<th scope="col">` for each column in FeatureTable.tsx
- [ ] T037 [US1] Map features array to FeatureRow components in `<tbody>` of FeatureTable.tsx
- [ ] T038 [US1] Add "Add Feature" button below table in FeatureTable.tsx that calls onAddFeature()
- [ ] T039 [US1] Add empty state message in FeatureTable.tsx: "No features added yet. Click 'Add Feature' to begin."

### Implement FeatureRow Component (Single Feature Input)

- [ ] T040 [US1] Implement FeatureRow.tsx: render `<tr>` with 5 `<td>` cells for type dropdown, project cost input, hardware cost input, estimated MD display, delete button
- [ ] T041 [US1] Add feature type dropdown in FeatureRow.tsx: `<select>` with options from FEATURE_TYPES constant, bind to feature.type, call onUpdate('type', value)
- [ ] T042 [US1] Add project cost input in FeatureRow.tsx: `<input type="number">`, bind to feature.projectCost, call onUpdate('projectCost', numericValue)
- [ ] T043 [US1] Add hardware cost input in FeatureRow.tsx: `<input type="number">`, bind to feature.hardwareCost, call onUpdate('hardwareCost', numericValue)
- [ ] T044 [US1] Calculate and display estimatedMD in FeatureRow.tsx: call calculateEstimatedMD(feature.projectCost || 0, feature.hardwareCost || 0), display result with "MD" suffix or as integer
- [ ] T045 [US1] Add delete button in FeatureRow.tsx: `<button>` with "Delete" text or icon, calls onDelete()
- [ ] T046 [US1] Add ARIA labels to all inputs in FeatureRow.tsx: "Feature type", "Project cost in Thai Baht", "Hardware cost in Thai Baht"

### Wire Up App Component (Data Flow for US1)

- [ ] T047 [US1] Implement addFeature() handler in App.tsx: push new feature with `{ id: crypto.randomUUID(), type: '', projectCost: 0, hardwareCost: 0 }` to features array
- [ ] T048 [US1] Implement updateFeature(id, field, value) handler in App.tsx: use immutable update pattern `features.map(f => f.id === id ? { ...f, [field]: value } : f)`
- [ ] T049 [US1] Implement deleteFeature(id) handler in App.tsx: filter features array `features.filter(f => f.id !== id)`
- [ ] T050 [US1] Implement InputView layout in InputView.tsx: render ProjectForm, FeatureTable, and placeholder for RoleSelector (to be added in Phase 4)
- [ ] T051 [US1] Conditionally render InputView in App.tsx when activeView === 'input'

### CSS Styling for US1 (Basic Layout)

- [ ] T052 [US1] Create src/App.css with basic table styles: border-collapse, cell padding, header background color
- [ ] T053 [US1] Add input field styles in App.css: consistent width, padding, border
- [ ] T054 [US1] Add button styles in App.css: padding, background color, hover state
- [ ] T055 [US1] Import App.css in App.tsx

### Manual Testing & Bug Fixes for US1

- [ ] T056 [US1] Run dev server command (e.g., `npm start`), open in browser, manually test US1 Scenario 1 (add feature, see MD calculation)
- [ ] T057 [US1] Manually test US1 Scenario 2 (change costs, verify MD updates)
- [ ] T058 [US1] Manually test US1 Scenario 3 (12500 cost → MD = 2)
- [ ] T059 [US1] Fix any bugs discovered during manual testing
- [ ] T060 [US1] Run test command and verify all US1 component tests pass

**Checkpoint**: User Story 1 complete - MVP functional! Users can input features and see MD calculations.

---

## Phase 4: User Story 2 - Control Role Visibility (Priority: P1)

**Goal**: Users can select which roles are relevant via checkboxes, with at least one role always selected

**Independent Test**: All roles checked by default; uncheck QA and BE; try to uncheck last remaining role (SA) → checkbox stays checked, inline message appears

**Duration**: ~2-3 hours

### Component Tests for US2 (TDD - Write First) ✅

- [ ] T061 [US2] Write component test in tests/components/App.test.tsx: "displays four role checkboxes all checked by default"
- [ ] T062 [US2] Write component test: "allows unchecking roles except the last one" (US2 Scenario 2)
- [ ] T063 [US2] Write component test: "shows inline message when trying to uncheck last role"

### Implement RoleSelector Component

- [ ] T064 [P] [US2] Create src/components/RoleSelector.tsx with props: selectedRoles, onChange
- [ ] T065 [US2] Implement RoleSelector.tsx: render four checkboxes for SA, QA, BE, Support using ROLE_LABELS from constants
- [ ] T066 [US2] Add checkbox state logic in RoleSelector.tsx: each checkbox checked if role key is in selectedRoles array
- [ ] T067 [US2] Implement "at least one role" validation in RoleSelector.tsx: when user clicks checkbox, check if unchecking would result in zero selected roles; if yes, prevent and show message
- [ ] T068 [US2] Display inline error message in RoleSelector.tsx when validation fails: "At least one role must be selected."
- [ ] T069 [US2] Add ARIA attributes to checkboxes in RoleSelector.tsx for accessibility

### Wire Up RoleSelector in App

- [ ] T070 [US2] Implement updateSelectedRoles(roles) handler in App.tsx: update selectedRoles state
- [ ] T071 [US2] Add RoleSelector component to InputView.tsx layout, pass selectedRoles and onUpdateSelectedRoles handler
- [ ] T072 [US2] Verify default state in App.tsx: `useState<RoleKey[]>(['sa', 'qa', 'be', 'support'])`

### Manual Testing & Bug Fixes for US2

- [ ] T073 [US2] Manually test US2 Scenario 1: uncheck QA and BE, verify checkboxes update correctly
- [ ] T074 [US2] Manually test US2 Scenario 2: with only SA selected, try to uncheck it, verify checkbox stays checked and message appears
- [ ] T075 [US2] Manually test US2 Scenario 3: check all four roles, verify all checkboxes can be checked
- [ ] T076 [US2] Run test command and verify all US2 component tests pass

**Checkpoint**: User Story 2 complete - Role selection with validation working

---

## Phase 5: User Story 3 - View Detailed Role-Based Effort Distribution (Priority: P1)

**Goal**: Users navigate to separate Output view showing Manday table with role-based allocations and totals

**Independent Test**: Add feature with data, click "View Output", verify Manday table displays with correct role allocations, click "Back to Input", verify data preserved

**Duration**: ~4-5 hours

### Component Tests for US3 (TDD - Write First) ✅

- [ ] T077 [US3] Write component test: "navigates to Output view and displays Manday table"
- [ ] T078 [US3] Write component test: "calculates role allocations correctly for Use Case / API" (US3 Scenario 1)
- [ ] T079 [US3] Write component test: "calculates role allocations correctly for Data Synchronization" (US3 Scenario 2)
- [ ] T080 [US3] Write component test: "displays Total row summing all columns" (US3 Scenario 3)
- [ ] T081 [US3] Write component test: "conditionally displays role columns based on selectedRoles"
- [ ] T082 [US3] Write component test: "preserves feature data when navigating between views"

### Implement MandayTable Component

- [ ] T083 [P] [US3] Create src/components/MandayTable.tsx with props: features, selectedRoles
- [ ] T084 [US3] Implement MandayTable.tsx: render semantic HTML `<table>` with dynamic columns based on selectedRoles
- [ ] T085 [US3] Add table header row in MandayTable.tsx: always show "Feature", "Rest MD", "Support + Rest MD"; conditionally show "SA (MD)", "QA (MD)", "BE (MD)", "Support (MD)" based on selectedRoles
- [ ] T086 [US3] For each feature, calculate roleAllocations using calculateRoleAllocations(feature.type, calculateEstimatedMD(...)) and render one table row
- [ ] T087 [US3] Display feature type name in first column of each row (e.g., "Use Case / API")
- [ ] T088 [US3] Display all MD values as integers (no decimals) in table cells
- [ ] T089 [US3] Add `<tfoot>` section to MandayTable.tsx for Total row
- [ ] T090 [US3] Calculate totals using calculateTotals(features) and render Total row with "Total" label and summed values for each column
- [ ] T091 [US3] Style total row in App.css: bold font-weight, background color (e.g., #e3f2fd), top border to distinguish from feature rows
- [ ] T092 [US3] Add empty state to MandayTable.tsx: "No features added yet. Return to Input to add features."

### Implement OutputView Layout

- [ ] T093 [US3] Implement OutputView.tsx layout: render "Back to Input" button and MandayTable component
- [ ] T094 [US3] Add "Back to Input" button in OutputView.tsx that calls onNavigateToInput handler
- [ ] T095 [US3] Ensure button is keyboard accessible with proper ARIA label

### Add View Navigation in App

- [ ] T096 [US3] Implement setActiveView(view) handler in App.tsx: update activeView state
- [ ] T097 [US3] Add "View Output" button in InputView.tsx that calls onNavigateToOutput (which calls setActiveView('output'))
- [ ] T098 [US3] Conditionally render OutputView in App.tsx when activeView === 'output'
- [ ] T099 [US3] Pass features, selectedRoles, and onNavigateToInput handler to OutputView

### Manual Testing & Bug Fixes for US3

- [ ] T100 [US3] Manually test navigation: add features, click "View Output", verify table displays
- [ ] T101 [US3] Manually test US3 Scenario 1: "Use Case / API" with E=2 → verify SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2
- [ ] T102 [US3] Manually test US3 Scenario 2: "Data Synchronization" with E=10 → verify SA=3, QA=2, BE=2, Support=3, Rest=0, Support+Rest=3
- [ ] T103 [US3] Manually test column visibility: uncheck QA and BE in Input view, navigate to Output, verify only SA and Support columns visible
- [ ] T104 [US3] Manually test state preservation: navigate Input → Output → Input, verify all feature data unchanged
- [ ] T105 [US3] Run test command and verify all US3 component tests pass

**Checkpoint**: User Story 3 complete - Full input/output flow with navigation working

---

## Phase 6: User Story 4 - Handle Invalid Input Gracefully (Priority: P2)

**Goal**: Users receive clear validation feedback for invalid inputs; empty fields treated as zero

**Independent Test**: Leave costs empty (MD shows 0), enter negative number (error appears), enter non-numeric text (error appears), correct to valid value (error clears)

**Duration**: ~2-3 hours

### Component Tests for US4 (TDD - Write First) ✅

- [ ] T106 [US4] Write component test: "treats empty cost fields as zero" (US4 Scenario 1)
- [ ] T107 [US4] Write component test: "shows error for negative cost" (US4 Scenario 2)
- [ ] T108 [US4] Write component test: "shows error for non-numeric cost" (US4 Scenario 3)
- [ ] T109 [US4] Write component test: "clears error when invalid input corrected" (US4 Scenario 4)

### Implement Input Validation in FeatureRow

- [ ] T110 [US4] Add local state to FeatureRow.tsx: `const [projectCostError, setProjectCostError] = useState<string>(); const [hardwareCostError, setHardwareCostError] = useState<string>();`
- [ ] T111 [US4] Add onBlur handler to project cost input in FeatureRow.tsx: call validateCostInput(), set error state if invalid, call onUpdate() if valid
- [ ] T112 [US4] Add onBlur handler to hardware cost input in FeatureRow.tsx: same validation pattern as T111
- [ ] T113 [US4] Display error message below project cost input in FeatureRow.tsx: render `<span className="error">{projectCostError}</span>` if error exists
- [ ] T114 [US4] Display error message below hardware cost input in FeatureRow.tsx: same as T113
- [ ] T115 [US4] Add error styling in App.css: `.error { color: #d32f2f; font-size: 0.875rem; margin-top: 4px; }`
- [ ] T116 [US4] Highlight invalid input fields with red border in App.css: `.input-error { border-color: #d32f2f; }`
- [ ] T117 [US4] Apply error class to input fields in FeatureRow.tsx when validation fails

### Handle Empty Inputs as Zero

- [ ] T118 [US4] Verify validateCostInput() already returns numericValue: 0 for empty string (from Phase 2) - no changes needed
- [ ] T119 [US4] Test empty inputs manually: leave both costs empty, verify estimatedMD shows 0, no error

### Feature Type Validation

- [ ] T120 [US4] Add validation to FeatureRow.tsx: if feature.type is empty string, display "—" or "Select type" instead of MD value
- [ ] T121 [US4] Conditionally disable estimatedMD display in FeatureRow.tsx if type not selected

### Manual Testing & Bug Fixes for US4

- [ ] T122 [US4] Manually test US4 Scenario 1: leave costs empty, verify 0 MD, no error
- [ ] T123 [US4] Manually test US4 Scenario 2: enter "-500", verify error message "Cost cannot be negative" and red border
- [ ] T124 [US4] Manually test US4 Scenario 3: enter "abc", verify error message "Must be a valid number"
- [ ] T125 [US4] Manually test US4 Scenario 4: correct invalid input to valid value, verify error clears
- [ ] T126 [US4] Test edge case: enter negative, fix to valid, verify error clears and MD recalculates
- [ ] T127 [US4] Run test command and verify all US4 component tests pass

**Checkpoint**: User Story 4 complete - Robust input validation in place

---

## Phase 7: Accessibility & Polish

**Purpose**: Accessibility improvements, keyboard navigation, ARIA labels, and visual polish

**Duration**: ~2-3 hours

### Accessibility Improvements

- [ ] T128 [P] Add keyboard navigation support: ensure tab order is logical (project name → features table → role checkboxes → view output button)
- [ ] T129 [P] Test keyboard-only navigation: navigate entire app using only Tab/Shift+Tab/Enter/Space, verify all interactive elements reachable
- [ ] T130 [P] Add focus indicators in App.css: `input:focus, select:focus, button:focus { outline: 2px solid #1976d2; outline-offset: 2px; }`
- [ ] T131 [P] Associate error messages with inputs using `aria-describedby` in FeatureRow.tsx: `<input aria-describedby="projectCostError">` when error exists
- [ ] T132 [P] Add `aria-live="polite"` to MandayTable.tsx so screen readers announce calculation updates
- [ ] T133 [P] Test with screen reader (VoiceOver on Mac/NVDA on Windows): verify table headers, labels, and errors are announced correctly

### Performance Validation

- [ ] T134 [P] Add performance test: create 20 features with random costs, measure calculation time using `performance.now()`, verify <100ms
- [ ] T135 [P] Test with 50 features (max per assumptions), verify no UI lag or performance degradation
- [ ] T136 [P] Run Lighthouse audit in Chrome DevTools, verify Accessibility score ≥90, Performance score ≥90

### CSS Polish

- [ ] T137 [P] Enhance table styling in App.css: alternating row colors, hover effect on rows, responsive width
- [ ] T138 [P] Add container max-width and center layout in App.css: `.app-container { max-width: 1200px; margin: 0 auto; padding: 20px; }`
- [ ] T139 [P] Style total row more prominently: increase font size or add background gradient
- [ ] T140 [P] Add visual distinction between Input and Output views: different background colors or borders
- [ ] T141 [P] Improve button styling: consistent sizing, clear hover/active states, proper spacing

### View Transition Polish

- [ ] T142 [P] Add smooth transitions between Input and Output views (e.g., fade in/out or slide animation)
- [ ] T143 [P] Ensure "View Output" button is prominent and clearly labeled
- [ ] T144 [P] Ensure "Back to Input" button is easy to find in Output view

**Checkpoint**: Application is accessible, performant, and visually polished

---

## Phase 8: Documentation & Final QA

**Purpose**: Developer documentation, testing verification, and final quality assurance

**Duration**: ~2-3 hours

### Documentation

- [ ] T145 [P] Create quickstart.md in specs/002-effort-estimator/ with setup instructions (install dependencies, run dev server, run tests)
- [ ] T146 [P] Add inline JSDoc comments to all functions in src/lib/calculations.ts: document formulas, parameters, return values
- [ ] T147 [P] Create or update README.md at repository root with project overview, setup instructions, and test commands
- [ ] T148 [P] Document role percentage formulas in comments within src/lib/constants.ts

### Final QA & Bug Bash

- [ ] T149 Test all edge cases from spec.md: zero costs, large numbers (50000), fractional costs, all roles zero, maximum 50 features
- [ ] T150 Cross-browser testing: verify functionality in Chrome, Firefox, Safari, Edge
- [ ] T151 Run full test suite with coverage: run test coverage command, verify 100% coverage of src/lib/calculations.ts, verify all component tests pass
- [ ] T152 Build production bundle: run build command, verify no TypeScript errors, verify output size reasonable
- [ ] T153 Preview production build: test functionality in production mode
- [ ] T154 Run linter (if configured): fix any errors or warnings
- [ ] T155 Review Constitution Check in plan.md: verify all 7 principles satisfied
- [ ] T156 Create plan.md's Acceptance Checklist verification: mark all items as complete

**Checkpoint**: Application complete, tested, documented, and ready for deployment! 🎉

---

## Task Summary

| Phase                  | Task Count           | Estimated Duration | Dependencies |
| ---------------------- | -------------------- | ------------------ | ------------ |
| Phase 1: Setup         | T001-T009 (9 tasks)  | 1-2 hours          | None         |
| Phase 2: Foundational  | T010-T021 (12 tasks) | 3-4 hours          | Phase 1      |
| Phase 3: US1 (MVP)     | T022-T060 (39 tasks) | 4-6 hours          | Phase 2      |
| Phase 4: US2           | T061-T076 (16 tasks) | 2-3 hours          | Phase 3      |
| Phase 5: US3           | T077-T105 (29 tasks) | 4-5 hours          | Phase 3      |
| Phase 6: US4           | T106-T127 (22 tasks) | 2-3 hours          | Phase 3      |
| Phase 7: Accessibility | T128-T144 (17 tasks) | 2-3 hours          | Phases 3-6   |
| Phase 8: Documentation | T145-T156 (12 tasks) | 2-3 hours          | All phases   |
| **Total**              | **156 tasks**        | **18-27 hours**    | Sequential   |

## Dependency Graph (User Story Completion Order)

```
Phase 1 (Setup)
    ↓
Phase 2 (Foundational - Pure Functions)
    ↓
    ├─→ Phase 3 (US1 - Input Features & Calculate MD) ← MUST COMPLETE FIRST
    │       ↓
    │       ├─→ Phase 4 (US2 - Role Selection)
    │       │
    │       ├─→ Phase 5 (US3 - Output View & Navigation)
    │       │
    │       └─→ Phase 6 (US4 - Input Validation)
    │               ↓
    │               ├─→ Phase 7 (Accessibility & Polish)
    │               │
    │               └─→ Phase 8 (Documentation & QA)
```

**Critical Path**: Phase 1 → Phase 2 → Phase 3 (US1) is the MVP delivery path. Phases 4, 5, and 6 can proceed in parallel after Phase 3 completes (with Phase 5 depending on Phase 4 for role selection state).

## Parallel Execution Opportunities

### Phase 2 (After T012 tests written):

- T017, T018, T019, T020 can be implemented in parallel (different functions)

### Phase 3 (After T022-T025 tests written):

- T026-T031 (component file creation) can be done in parallel
- T032-T046 (component implementation) can be done in parallel by different developers

### Phase 7:

- T128-T133 (accessibility), T134-T136 (performance), T137-T144 (CSS) can all run in parallel

### Phase 8:

- T145-T148 (documentation) can run in parallel with T149-T156 (QA)

## Independent Testing Criteria

### US1 Independent Test

✅ Can run without US2/US3/US4 features

- Create new project in clean browser
- Enter project name "Test Project"
- Click "Add Feature"
- Select "Use Case / API" from dropdown
- Enter Project cost: 10000, Hardware cost: 0
- Verify: Estimated MD = 2 displays in feature row
- Modify Project cost to 12500
- Verify: Estimated MD updates to 2 (rounds down)

### US2 Independent Test

✅ Can run after US1, independent of US3/US4

- Complete US1 test above (add at least one feature)
- Verify: All four role checkboxes are checked by default
- Uncheck "QA" checkbox
- Uncheck "BE" checkbox
- Verify: Only SA and Support remain checked
- Try to uncheck "SA" checkbox
- Verify: Checkbox remains checked, inline message appears: "At least one role must be selected."

### US3 Independent Test

✅ Can run after US1 and US2

- Complete US1 test (add feature with data)
- Click "View Output" button
- Verify: Manday table appears with correct role allocations (SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2)
- Verify: Only selected role columns visible (from US2 state)
- Click "Back to Input" button
- Verify: Returns to Input view, all feature data preserved
- Change a cost value
- Click "View Output" again
- Verify: Updated calculations reflected in table

### US4 Independent Test

✅ Can run after US1, independent of US2/US3

- Create new project, add one feature
- Leave both cost fields empty
- Verify: Estimated MD = 0, no errors shown
- Enter Project cost: -500
- Tab away (trigger blur)
- Verify: Red error message appears: "Cost cannot be negative"
- Verify: Input field has red border
- Change to valid value: 10000
- Verify: Error clears, MD recalculates to 2

## Implementation Strategy

### Recommended MVP Scope (First Deliverable)

**Deliver Phase 1 + Phase 2 + Phase 3 (US1) first** = ~8-12 hours of work

This gives users:

- ✅ Working feature input with MD calculation
- ✅ Core calculation engine (100% tested)
- ✅ Real-time MD updates
- ✅ Basic accessible UI

Then incrementally add:

- **Iteration 2**: Phase 4 (US2) for role selection (~2-3 hours)
- **Iteration 3**: Phase 5 (US3) for Output view and navigation (~4-5 hours)
- **Iteration 4**: Phase 6 (US4) for input validation (~2-3 hours)
- **Iteration 5**: Phase 7 for accessibility & polish (~2-3 hours)
- **Iteration 6**: Phase 8 for documentation & QA (~2-3 hours)

### Test-Driven Development Flow

1. **RED**: Write test that fails (e.g., T013 - test calculateEstimatedMD before it exists)
2. **GREEN**: Write minimal code to make test pass (T017 - implement function)
3. **REFACTOR**: Clean up code while keeping tests green
4. **REPEAT**: Move to next test

### Code Review Checkpoints

- After Phase 2: Review all pure functions and unit tests (foundational, must be correct)
- After Phase 3: Review MVP functionality (US1 complete)
- After Phase 5: Review full input/output flow (US1-US3 complete)
- After Phase 8: Final review before deployment

---

**Ready to start?** Begin with Task T001 and follow the phases sequentially. Each phase builds on the previous one. Good luck! 🚀
