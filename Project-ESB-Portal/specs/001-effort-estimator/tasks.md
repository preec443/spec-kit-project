# Tasks: Integration Effort Estimator

**Input**: Design documents from `/specs/001-effort-estimator/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: Tests are MANDATORY per Constitution Principle VI (Test-First Development). All calculation functions require 100% unit test coverage; main user flow requires component test(s).

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `- [ ] [ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Project Structure

This is a **single-project frontend-only application** using:
- **Framework**: React 18+ with TypeScript 5.3+
- **Build Tool**: Vite 5+
- **Testing**: Vitest + React Testing Library
- **Paths**: `src/` and `tests/` at repository root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, Vite scaffolding, and basic configuration

**Duration**: ~1-2 hours

- [ ] T001 Create Vite React TypeScript project using `npm create vite@latest . -- --template react-ts`
- [ ] T002 Install dependencies: `npm install react@^18.2.0 react-dom@^18.2.0`
- [ ] T003 Install dev dependencies: `npm install -D @testing-library/react@^14.0.0 @testing-library/user-event@^14.5.0 @vitejs/plugin-react@^4.2.0 jsdom@^23.0.0 typescript@^5.3.0 vite@^5.0.0 vitest@^1.0.0`
- [ ] T004 Configure tsconfig.json with strict mode: `"strict": true, "noUnusedLocals": true, "noUnusedParameters": true`
- [ ] T005 Configure vite.config.ts with Vitest plugin: add `test: { globals: true, environment: 'jsdom', setupFiles: './src/test/setup.ts' }`
- [ ] T006 Create directory structure: `src/components/`, `src/lib/`, `tests/unit/`, `tests/components/`
- [ ] T007 Add test scripts to package.json: `"test": "vitest", "test:ui": "vitest --ui", "test:coverage": "vitest --coverage"`
- [ ] T008 Create .gitignore with node_modules/, dist/, coverage/, .env

**Checkpoint**: Project structure ready - foundation work can begin

---

## Phase 2: Foundational (Pure Functions & Types)

**Purpose**: Core calculation logic and type definitions that ALL user stories depend on

**⚠️ CRITICAL**: No UI work can begin until this phase is complete. Use TDD: write tests first, then implement.

**Duration**: ~3-4 hours

### Type Definitions & Constants (Foundation)

- [ ] T009 [P] Create src/lib/types.ts with all TypeScript interfaces: Feature, FeatureType, RoleAllocation, RolePercentages, ValidationResult, Calculation, TotalCalculation
- [ ] T010 [P] Create src/lib/constants.ts with MD_DIVISOR (5000), FEATURE_TYPES array, ROLE_PERCENTAGES map, ROLE_LABELS map

### Unit Tests for Calculation Functions (TDD - Write First) ✅

- [ ] T011 [P] Create tests/unit/calculations.test.ts with test suite structure for all 4 calculation functions
- [ ] T012 [P] Write unit tests for calculateEstimatedMD: zero costs, normal division, floor rounding (12500 → 2), large numbers (50000000 → 10000), fractional costs (12345.67 → 2)
- [ ] T013 [P] Write unit tests for calculateRoleAllocations: all 5 feature types, various MD values (0, 2, 10, 100), verify floor rounding per role, verify restMD calculation, verify supportPlusRest sum
- [ ] T014 [P] Write unit tests for calculateTotals: empty array → all zeros, single feature, multiple features with different types, verify column-wise sums
- [ ] T015 [P] Write unit tests for validateCostInput: empty string → valid/0, valid positive number, negative number → error, non-numeric → error, fractional valid

### Implement Calculation Functions (TDD - Make Tests Pass)

- [ ] T016 [P] Implement calculateEstimatedMD() in src/lib/calculations.ts: `Math.floor((projectCost + hardwareCost) / 5000)`
- [ ] T017 [P] Implement calculateRoleAllocations() in src/lib/calculations.ts: apply ROLE_PERCENTAGES with Math.floor, calculate restMD, calculate supportPlusRest
- [ ] T018 [P] Implement calculateTotals() in src/lib/calculations.ts: reduce over features array, sum each role column
- [ ] T019 [P] Implement validateCostInput() in src/lib/calculations.ts: handle empty string, validate non-negative, return ValidationResult object
- [ ] T020 Run `npm test` and verify 100% coverage of src/lib/calculations.ts (all tests pass)

**Checkpoint**: All pure functions tested and working - UI components can now use them safely

---

## Phase 3: User Story 1 - Calculate Single Feature Effort (Priority: P1) 🎯 MVP

**Goal**: User enters project name and one feature, sees calculated MD and role distribution immediately

**Independent Test**: Enter project name "API Integration Project", add one "Use Case / API" feature with costs 10000/0, verify output table shows SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2

**Duration**: ~4-6 hours

### Component Tests for US1 (TDD - Write First) ✅

- [ ] T021 Create tests/components/App.test.tsx with React Testing Library setup
- [ ] T022 [US1] Write component test: "renders empty app with project name input and add feature button"
- [ ] T023 [US1] Write component test: "calculates and displays MD for single feature" (full US1 Scenario 1 flow)
- [ ] T024 [US1] Write component test: "updates output when costs change" (US1 Scenario 2 - real-time reactivity)

### Basic Component Structure

- [ ] T025 [P] [US1] Create src/components/App.tsx with state: `const [projectName, setProjectName] = useState<string>(''); const [features, setFeatures] = useState<Feature[]>([]);`
- [ ] T026 [P] [US1] Create src/components/ProjectForm.tsx with props: projectName, onProjectNameChange
- [ ] T027 [P] [US1] Create src/components/FeatureInputTable.tsx with props: features, onAddFeature, onUpdateFeature, onDeleteFeature
- [ ] T028 [P] [US1] Create src/components/FeatureRow.tsx with props: feature, onUpdate, onDelete
- [ ] T029 [P] [US1] Create src/components/MandayTable.tsx with props: features, calculations, totals

### Implement ProjectForm Component

- [ ] T030 [US1] Implement ProjectForm.tsx: render text input with label "Project Name", bind value to projectName prop, call onProjectNameChange on input change
- [ ] T031 [US1] Add required indicator (*) to project name label in ProjectForm.tsx
- [ ] T032 [US1] Add ARIA label to project name input: `aria-label="Project name (required)"`

### Implement FeatureInputTable Component

- [ ] T033 [US1] Implement FeatureInputTable.tsx: render semantic HTML `<table>` with `<thead>` containing column headers: Feature Type, Project Cost (THB), Hardware Cost (THB), Estimated MD, Actions
- [ ] T034 [US1] Add table header row with `<th scope="col">` for each column in FeatureInputTable.tsx
- [ ] T035 [US1] Map features array to FeatureRow components in `<tbody>` of FeatureInputTable.tsx
- [ ] T036 [US1] Add "Add Feature" button below table in FeatureInputTable.tsx that calls onAddFeature()
- [ ] T037 [US1] Add empty state message in FeatureInputTable.tsx: "No features added yet. Click 'Add Feature' to begin."

### Implement FeatureRow Component (Single Feature Input)

- [ ] T038 [US1] Implement FeatureRow.tsx: render `<tr>` with 5 `<td>` cells for type dropdown, project cost input, hardware cost input, estimated MD display, delete button
- [ ] T039 [US1] Add feature type dropdown in FeatureRow.tsx: `<select>` with options from FEATURE_TYPES constant, bind to feature.type, call onUpdate('type', value)
- [ ] T040 [US1] Add project cost input in FeatureRow.tsx: `<input type="number">`, bind to feature.projectCost, call onUpdate('projectCost', numericValue)
- [ ] T041 [US1] Add hardware cost input in FeatureRow.tsx: `<input type="number">`, bind to feature.hardwareCost, call onUpdate('hardwareCost', numericValue)
- [ ] T042 [US1] Calculate and display estimatedMD in FeatureRow.tsx: call calculateEstimatedMD(feature.projectCost, feature.hardwareCost), display result with "MD" suffix
- [ ] T043 [US1] Add delete button in FeatureRow.tsx: `<button>` with "Delete" text or icon, calls onDelete()
- [ ] T044 [US1] Add ARIA labels to all inputs in FeatureRow.tsx: "Feature type", "Project cost in Thai Baht", "Hardware cost in Thai Baht"

### Implement MandayTable Component (Output Table)

- [ ] T045 [US1] Implement MandayTable.tsx: render semantic HTML `<table>` with 7 columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD
- [ ] T046 [US1] Add table header row with `<th scope="col">` for each column in MandayTable.tsx
- [ ] T047 [US1] Map calculations array to table rows in MandayTable.tsx: for each calculation, render `<tr>` with feature type label and role MD values
- [ ] T048 [US1] Display feature type name in first column of MandayTable.tsx rows (lookup feature.type from features array using calculation.featureId)
- [ ] T049 [US1] Display all role MD values as integers in MandayTable.tsx (no decimal places)
- [ ] T050 [US1] Add empty state to MandayTable.tsx: "No calculations to display. Add features above to see results."

### Wire Up App Component (Data Flow)

- [ ] T051 [US1] Implement addFeature() handler in App.tsx: push new feature with `{ id: crypto.randomUUID(), type: '', projectCost: 0, hardwareCost: 0 }` to features array
- [ ] T052 [US1] Implement updateFeature(id, field, value) handler in App.tsx: use immutable update pattern `features.map(f => f.id === id ? { ...f, [field]: value } : f)`
- [ ] T053 [US1] Implement deleteFeature(id) handler in App.tsx: filter features array `features.filter(f => f.id !== id)`
- [ ] T054 [US1] Calculate derived data in App.tsx: `const calculations = features.map(f => ({ featureId: f.id, estimatedMD: calculateEstimatedMD(...), roleAllocations: calculateRoleAllocations(f.type, estimatedMD) }))`
- [ ] T055 [US1] Calculate totals in App.tsx: `const totals = calculateTotals(calculations)`
- [ ] T056 [US1] Pass state and handlers to child components: ProjectForm, FeatureInputTable (with handlers), MandayTable (with calculations and totals)
- [ ] T057 [US1] Add basic layout structure in App.tsx: header with app title, ProjectForm section, FeatureInputTable section, MandayTable section

### CSS Styling for US1 (Basic Layout)

- [ ] T058 [US1] Create src/App.css with basic table styles: border-collapse, cell padding, header background color
- [ ] T059 [US1] Add input field styles in App.css: consistent width, padding, border
- [ ] T060 [US1] Add button styles in App.css: padding, background color, hover state
- [ ] T061 [US1] Import App.css in App.tsx

### Manual Testing & Bug Fixes for US1

- [ ] T062 [US1] Run `npm run dev`, open http://localhost:5173, manually test US1 Scenario 1 (single feature calculation)
- [ ] T063 [US1] Manually test US1 Scenario 2 (real-time updates when costs change)
- [ ] T064 [US1] Manually test US1 Scenario 3 (feature type change updates role percentages)
- [ ] T065 [US1] Fix any bugs discovered during manual testing
- [ ] T066 [US1] Run `npm test` and verify all US1 component tests pass

**Checkpoint**: User Story 1 complete - MVP functional! Users can estimate a single feature.

---

## Phase 4: User Story 2 - Manage Multiple Integration Features (Priority: P2)

**Goal**: Users can add, view, and calculate efforts for multiple features with a total summary row

**Independent Test**: Add 3 features of different types with varying costs, verify each row calculates independently, confirm total row sums all columns correctly

**Duration**: ~2-3 hours

### Component Tests for US2 (TDD - Write First) ✅

- [ ] T067 [US2] Write component test in tests/components/App.test.tsx: "displays total row summing all features" (US2 Scenario 2)
- [ ] T068 [US2] Write component test: "updates totals when feature is deleted" (US2 Scenario 3)

### Enhance FeatureInputTable for Multiple Features

- [ ] T069 [US2] Verify FeatureInputTable.tsx already maps all features in features array (from Phase 3) - no changes needed if already working
- [ ] T070 [US2] Test adding 5 features in FeatureInputTable and verify each gets unique key (feature.id)

### Implement Total Row in MandayTable

- [ ] T071 [US2] Add `<tfoot>` section to MandayTable.tsx table for total row
- [ ] T072 [US2] Render total row in MandayTable.tsx: `<tr>` with "Total" label in first cell, totals.totalEstimatedMD, totals.totalRoleAllocations.sa/qa/be/support, totals.restMD, totals.supportPlusRest
- [ ] T073 [US2] Style total row in App.css: bold font-weight, background color (e.g., #e3f2fd), top border to distinguish from feature rows
- [ ] T074 [US2] Add `<th scope="row">` for "Total" label cell in total row

### Manual Testing & Bug Fixes for US2

- [ ] T075 [US2] Manually test US2 Scenario 1: add 3+ features and verify each calculates independently
- [ ] T076 [US2] Manually test US2 Scenario 2: verify total row sums match manual calculation
- [ ] T077 [US2] Manually test US2 Scenario 3: delete middle feature, verify totals recalculate
- [ ] T078 [US2] Test edge case: delete all features, verify total row shows all zeros
- [ ] T079 [US2] Run `npm test` and verify all US2 component tests pass

**Checkpoint**: User Story 2 complete - Users can manage multiple features with accurate totals

---

## Phase 5: User Story 3 - Handle Empty and Invalid Inputs Gracefully (Priority: P3)

**Goal**: Users receive clear validation feedback for invalid inputs; empty fields treated as zero

**Independent Test**: Leave costs empty (should show 0 MD), enter negative number (should show error), enter non-numeric text (should show error)

**Duration**: ~2-3 hours

### Component Tests for US3 (TDD - Write First) ✅

- [ ] T080 [US3] Write component test: "treats empty cost fields as zero" (US3 Scenario 1)
- [ ] T081 [US3] Write component test: "shows error for negative cost" (US3 Scenario 2)
- [ ] T082 [US3] Write component test: "shows error for non-numeric cost" (US3 Scenario 3)
- [ ] T083 [US3] Write component test: "requires feature type selection" (US3 Scenario 4)

### Implement Input Validation in FeatureRow

- [ ] T084 [US3] Add local state to FeatureRow.tsx: `const [projectCostError, setProjectCostError] = useState<string>(); const [hardwareCostError, setHardwareCostError] = useState<string>();`
- [ ] T085 [US3] Add onBlur handler to project cost input in FeatureRow.tsx: call validateCostInput(), set error state if invalid, call onUpdate() if valid
- [ ] T086 [US3] Add onBlur handler to hardware cost input in FeatureRow.tsx: same validation pattern as T085
- [ ] T087 [US3] Display error message below project cost input in FeatureRow.tsx: render `<span className="error">{projectCostError}</span>` if error exists
- [ ] T088 [US3] Display error message below hardware cost input in FeatureRow.tsx: same as T087
- [ ] T089 [US3] Add error styling in App.css: `.error { color: #d32f2f; font-size: 0.875rem; margin-top: 4px; }`
- [ ] T090 [US3] Highlight invalid input fields with red border in App.css: `.input-error { border-color: #d32f2f; }`

### Handle Empty Inputs as Zero

- [ ] T091 [US3] Verify validateCostInput() already returns numericValue: 0 for empty string (from Phase 2) - no changes needed
- [ ] T092 [US3] Test empty inputs manually: leave both costs empty, verify estimatedMD shows 0, output table shows all zeros

### Feature Type Validation

- [ ] T093 [US3] Add validation to FeatureRow.tsx: if feature.type is empty string, display message "Feature type is required" in place of estimatedMD
- [ ] T094 [US3] Conditionally disable estimatedMD display in FeatureRow.tsx if type not selected: show "—" or "N/A" instead of 0
- [ ] T095 [US3] Update MandayTable.tsx to skip rows where feature.type is empty (don't include in calculations or display)

### Manual Testing & Bug Fixes for US3

- [ ] T096 [US3] Manually test US3 Scenario 1: leave costs empty, verify 0 MD
- [ ] T097 [US3] Manually test US3 Scenario 2: enter "-500", verify error message and red border
- [ ] T098 [US3] Manually test US3 Scenario 3: enter "abc", verify error message
- [ ] T099 [US3] Manually test US3 Scenario 4: leave type unselected, verify no calculation or message shown
- [ ] T100 [US3] Test edge case: enter negative, fix to valid, verify error clears
- [ ] T101 [US3] Run `npm test` and verify all US3 component tests pass

**Checkpoint**: User Story 3 complete - Robust input validation in place

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Accessibility, performance optimization, documentation, and final QA

**Duration**: ~2-3 hours

### Accessibility Improvements

- [ ] T102 [P] Add keyboard navigation support: ensure tab order is logical (project name → feature type → costs → delete → add feature → next row)
- [ ] T103 [P] Test keyboard-only navigation: navigate entire form using only Tab/Shift+Tab/Enter/Space, verify all interactive elements reachable
- [ ] T104 [P] Add focus indicators in App.css: `input:focus, select:focus, button:focus { outline: 2px solid #1976d2; outline-offset: 2px; }`
- [ ] T105 [P] Associate error messages with inputs using `aria-describedby` in FeatureRow.tsx: `<input aria-describedby="projectCostError">` when error exists
- [ ] T106 [P] Add `aria-live="polite"` to MandayTable.tsx so screen readers announce calculation updates
- [ ] T107 [P] Test with screen reader (VoiceOver on Mac/NVDA on Windows): verify table headers, labels, and errors are announced correctly

### Performance Validation

- [ ] T108 [P] Add performance test: create 20 features with random costs, measure calculation time using `performance.now()`, verify <100ms
- [ ] T109 [P] Test with 50 features (max per assumptions), verify no UI lag or performance degradation
- [ ] T110 [P] Run Lighthouse audit in Chrome DevTools, verify Accessibility score ≥90, Performance score ≥90

### CSS Polish

- [ ] T111 [P] Enhance table styling in App.css: alternating row colors, hover effect on rows, responsive width
- [ ] T112 [P] Add container max-width and center layout in App.css: `.app-container { max-width: 1200px; margin: 0 auto; padding: 20px; }`
- [ ] T113 [P] Style total row more prominently: increase font size, add background gradient, or bold all text
- [ ] T114 [P] Add responsive styles for mobile: stack input fields vertically, horizontal scroll for tables, larger touch targets

### Documentation Updates

- [ ] T115 [P] Update quickstart.md with actual dependency versions from package.json
- [ ] T116 [P] Add inline JSDoc comments to all functions in src/lib/calculations.ts: document formulas, parameters, return values
- [ ] T117 [P] Add README.md at repository root with project overview, setup instructions (link to quickstart.md), and test commands
- [ ] T118 [P] Update plan.md "Next Steps" section: mark implementation complete, add deployment notes
- [ ] T119 [P] Create example .env.example file if any environment variables needed (or document none needed for v1.0)

### Final QA & Bug Bash

- [ ] T120 Test all edge cases from spec.md: zero costs, large numbers (50M), fractional costs (12345.67), all roles zero, maximum 50 features
- [ ] T121 Cross-browser testing: verify functionality in Chrome, Firefox, Safari, Edge
- [ ] T122 Run full test suite with coverage: `npm run test:coverage`, verify 100% coverage of src/lib/calculations.ts, verify all component tests pass
- [ ] T123 Build production bundle: `npm run build`, verify no TypeScript errors, verify dist/ output size <200KB gzipped
- [ ] T124 Preview production build: `npm run preview`, test functionality in production mode
- [ ] T125 Run linter (if configured): `npm run lint`, fix any errors or warnings
- [ ] T126 Review Constitution Check in plan.md: verify all 7 principles satisfied, document any justified exceptions
- [ ] T127 Create plan.md's Acceptance Checklist verification: mark all 15+ items as complete

**Checkpoint**: Application complete, tested, and ready for deployment! 🎉

---

## Task Summary

| Phase | Task Count | Estimated Duration | Dependencies |
|-------|------------|-------------------|--------------|
| Phase 1: Setup | T001-T008 (8 tasks) | 1-2 hours | None |
| Phase 2: Foundational | T009-T020 (12 tasks) | 3-4 hours | Phase 1 |
| Phase 3: US1 (MVP) | T021-T066 (46 tasks) | 4-6 hours | Phase 2 |
| Phase 4: US2 | T067-T079 (13 tasks) | 2-3 hours | Phase 3 |
| Phase 5: US3 | T080-T101 (22 tasks) | 2-3 hours | Phase 3 |
| Phase 6: Polish | T102-T127 (26 tasks) | 2-3 hours | Phases 3-5 |
| **Total** | **127 tasks** | **14-21 hours** | Sequential |

## Dependency Graph (User Story Completion Order)

```
Phase 1 (Setup)
    ↓
Phase 2 (Foundational - Pure Functions)
    ↓
    ├─→ Phase 3 (US1 - MVP) ← MUST COMPLETE FIRST
    │       ↓
    │       ├─→ Phase 4 (US2 - Multiple Features)
    │       │
    │       └─→ Phase 5 (US3 - Validation)
    │               ↓
    └───────────────┴─→ Phase 6 (Polish)
```

**Critical Path**: Phase 1 → Phase 2 → Phase 3 (US1) is the MVP delivery path. Phases 4 and 5 can proceed in parallel after Phase 3 completes.

## Parallel Execution Opportunities

### Phase 2 (After T011 tests written):
- T016, T017, T018, T019 can be implemented in parallel (different functions)

### Phase 3 (After T021-T024 tests written):
- T025-T029 (component file creation) can be done in parallel
- T030-T032, T033-T037, T038-T044, T045-T050 (component implementation) can be done in parallel by different developers

### Phase 6:
- T102-T107 (accessibility), T108-T110 (performance), T111-T114 (CSS), T115-T119 (docs) can all run in parallel

## Independent Testing Criteria

### US1 Independent Test
✅ Can run without US2/US3 features
- Create new project in clean browser
- Enter project name "Test Project"
- Click "Add Feature"
- Select "Use Case / API" from dropdown
- Enter Project cost: 10000, Hardware cost: 0
- Verify: Estimated MD = 2 in input section
- Verify: Output table shows Feature="Use Case / API", SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2
- Modify Project cost to 15000
- Verify: Estimated MD updates to 3, output recalculates

### US2 Independent Test
✅ Can run after US1, without US3
- Complete US1 test above
- Click "Add Feature" 2 more times
- Feature 2: Type="Data Synchronization", Project cost=50000, Hardware cost=0 (E=10)
- Feature 3: Type="ESB Feature", Project cost=25000, Hardware cost=0 (E=5)
- Verify: Output table shows 3 feature rows + 1 total row
- Verify: Total row sums all columns correctly (manual calculation check)
- Delete Feature 2 (middle row)
- Verify: Output table shows 2 rows, totals recalculated

### US3 Independent Test
✅ Can run after US1, without US2
- Create new project, add one feature
- Leave both cost fields empty
- Verify: Estimated MD = 0, no errors shown
- Enter Project cost: -500
- Tab away (trigger blur)
- Verify: Red error message appears: "Project cost cannot be negative"
- Verify: Input field has red border
- Change to valid value: 10000
- Verify: Error clears, calculation proceeds

## Implementation Strategy

### Recommended MVP Scope (First Deliverable)
**Deliver Phase 1 + Phase 2 + Phase 3 (US1) first** = ~8-12 hours of work

This gives users:
- ✅ Working single-feature calculator
- ✅ Core calculation engine (100% tested)
- ✅ Real-time reactivity
- ✅ Basic accessible UI

Then incrementally add:
- **Iteration 2**: Phase 4 (US2) for multiple features + totals (~2-3 hours)
- **Iteration 3**: Phase 5 (US3) for validation (~2-3 hours)
- **Iteration 4**: Phase 6 for polish (~2-3 hours)

### Test-Driven Development Flow
1. **RED**: Write test that fails (e.g., T012 - test calculateEstimatedMD before it exists)
2. **GREEN**: Write minimal code to make test pass (T016 - implement function)
3. **REFACTOR**: Clean up code while keeping tests green
4. **REPEAT**: Move to next test

### Code Review Checkpoints
- After Phase 2: Review all pure functions and unit tests (foundational, must be correct)
- After Phase 3: Review MVP functionality (US1 complete)
- After Phase 6: Final review before deployment

---

**Ready to start?** Begin with Task T001 and follow the phases sequentially. Each phase builds on the previous one. Good luck! 🚀
