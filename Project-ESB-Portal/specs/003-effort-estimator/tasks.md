---
description: "Task list for Integration Effort Estimator implementation"
---

# Tasks: Integration Effort Estimator

**Input**: Design documents from `/specs/003-effort-estimator/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are MANDATORY for the Integration Effort Estimator per Constitution Principle VI (Test-First Development). All calculation functions require unit tests; main user flow requires component test(s).

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `- [ ] [ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- **Frontend-only React SPA**: `src/`, `tests/` at repository root
- All paths assume single project structure per plan.md

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Initialize a React + TypeScript project with the team's chosen build tool (ensure strict mode enabled in tsconfig.json)
- [ ] T002 [P] Install core dependencies: React 18+, TypeScript 5+
- [ ] T003 [P] Install testing dependencies: React-compatible test runner (e.g., Jest) and React Testing Library
- [ ] T004 [P] Configure linting (ESLint) and formatting (Prettier) with TypeScript rules
- [ ] T005 Create project directory structure: src/lib/, src/components/, src/types/, src/utils/, tests/integration/
- [ ] T006 [P] Setup base CSS file at src/App.css with table layout styles
- [ ] T007 [P] Create .gitignore for node_modules, build output, and IDE files

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core TypeScript interfaces and pure calculation functions that ALL user stories depend on

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T008 [P] Create Feature interface and FeatureType enum in src/types/Feature.ts
- [ ] T009 [P] Create RoleSelection interface in src/types/Role.ts
- [ ] T010 [P] Create MandayAllocation and TotalSummary interfaces in src/types/MandayAllocation.ts
- [ ] T011 [P] Create validation helper interfaces in src/utils/validation.ts (ValidationResult, CostValidationResult)

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - Calculate Project Effort (Priority: P1) 🎯 MVP

**Goal**: Enable project managers to input project details and integration features, then view a manday breakdown showing how effort is distributed across roles.

**Independent Test**: Enter project name, add features with costs, view output table with correct MD calculations and role allocations.

### Tests for User Story 1 (MANDATORY per Constitution) ✅

> **NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T012 [P] [US1] Unit test: calculateTotalMD function in src/lib/calculations.test.ts (test cases: zero costs, decimals, large values, edge case 0/5000=0)
- [ ] T013 [P] [US1] Unit test: getRolePercentages function in src/lib/calculations.test.ts (test all 5 feature types return correct percentages)
- [ ] T014 [P] [US1] Unit test: calculateRoleMD function in src/lib/calculations.test.ts (test round-down edge cases: 5*0.3=1, 10*0.7=7)
- [ ] T015 [P] [US1] Unit test: calculateRestMD function in src/lib/calculations.test.ts (test partial role selections)
- [ ] T016 [P] [US1] Unit test: calculateFeatureAllocation function in src/lib/calculations.test.ts (test each feature type + role combination from spec scenarios)
- [ ] T017 [US1] Component test: Full user flow in tests/integration/App.test.tsx (render → enter project name → add 2 features → view output → verify MD values)

### Implementation for User Story 1

- [ ] T018 [P] [US1] Implement calculateTotalMD(projectCost, hardwareCost) in src/lib/calculations.ts using Math.ceil((costs)/5000)
- [ ] T019 [P] [US1] Implement getRolePercentages(featureType) in src/lib/calculations.ts with percentage map for all 5 types (FR-019 to FR-023)
- [ ] T020 [P] [US1] Implement calculateRoleMD(totalMD, percentage) in src/lib/calculations.ts using Math.floor(totalMD \* percentage)
- [ ] T021 [P] [US1] Implement calculateRestMD(totalMD, allocatedRoles) in src/lib/calculations.ts
- [ ] T022 [US1] Implement calculateFeatureAllocation(feature, selectedRoles) in src/lib/calculations.ts (depends on T018-T021)
- [ ] T023 [P] [US1] Create App component in src/components/App.tsx with state management (viewState, projectName, features, selectedRoles)
- [ ] T024 [P] [US1] Create InputView component in src/components/InputView.tsx with project name input and "View Output" button
- [ ] T025 [US1] Create FeatureRow component in src/components/FeatureRow.tsx (dropdown for type, inputs for costs, no remove button yet)
- [ ] T026 [US1] Add FeaturesTable to InputView with single initial FeatureRow (hardcoded, no add/remove yet)
- [ ] T027 [US1] Create RoleSelection component in InputView with 4 checkboxes (all checked by default, no validation yet)
- [ ] T028 [US1] Create OutputView component in src/components/OutputView.tsx with project name header and "Back to Input" button
- [ ] T029 [US1] Create MandayTable component in src/components/MandayTable.tsx with header row and feature rows
- [ ] T030 [US1] Implement dynamic column rendering in MandayTable based on selectedRoles (show only selected role columns + Rest MD + Support+Rest MD)
- [ ] T031 [US1] Implement TotalRow in MandayTable with summed values and grey background for Support/Rest MD cells (FR-029, FR-030)
- [ ] T032 [US1] Wire App state to InputView and OutputView, toggle viewState on button clicks
- [ ] T033 [US1] Add basic validation: prevent navigation to output if project name is empty (FR-011)
- [ ] T034 [US1] Add CSS styling in src/App.css: table layouts, spacing, grey backgrounds (#e0e0e0) for Total row cells

**Checkpoint**: At this point, User Story 1 (MVP) should be fully functional - user can enter one feature, view output with correct calculations

---

## Phase 4: User Story 2 - Selective Role Allocation (Priority: P2)

**Goal**: Allow project managers to select only the roles available on their team, with the output table dynamically showing only selected role columns and correct Rest MD.

**Independent Test**: Check/uncheck different role combinations, verify output table shows only selected columns and Rest MD calculates correctly.

### Tests for User Story 2 (MANDATORY per Constitution) ✅

- [ ] T035 [P] [US2] Unit test: validateRoleSelection function in src/utils/validation.ts (test "at least one role" validation)
- [ ] T036 [US2] Component test: Role selection validation in tests/integration/App.test.tsx (attempt to uncheck all roles, verify error message)
- [ ] T037 [US2] Component test: Dynamic column visibility in tests/integration/App.test.tsx (uncheck QA/BE, verify only SA/Support/Rest/Support+Rest columns appear)

### Implementation for User Story 2

- [ ] T038 [P] [US2] Implement validateRoleSelection(roles) in src/utils/validation.ts (return {valid: false, error: "..."} if all false)
- [ ] T039 [US2] Add role validation to RoleSelection component: disable last checkbox if only one role selected (FR-008)
- [ ] T040 [US2] Display inline error message in RoleSelection when validation fails: "At least one role must be selected"
- [ ] T041 [US2] Update MandayTable to recalculate Rest MD when role selection changes (verify formula: totalMD - sum of selected roles)
- [ ] T042 [US2] Add conditional rendering logic in MandayTable header to hide/show role columns based on selectedRoles (FR-015)
- [ ] T043 [US2] Update TotalRow to sum only visible role columns, keep Support/Rest MD grey and empty (FR-029, FR-030)
- [ ] T044 [US2] Add CSS for disabled checkbox styling in src/App.css

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently - role selection affects output display

---

## Phase 5: User Story 3 - Manage Multiple Features (Priority: P2)

**Goal**: Enable project managers to dynamically add, edit, and remove integration features without losing other entered data.

**Independent Test**: Add multiple features, edit their values, remove features, verify calculations update correctly and data is preserved.

### Tests for User Story 3 (MANDATORY per Constitution) ✅

- [ ] T045 [P] [US3] Component test: Add feature in tests/integration/App.test.tsx (click "Add Feature", verify new row appears with empty fields)
- [ ] T046 [P] [US3] Component test: Remove feature in tests/integration/App.test.tsx (add 3 features, remove middle one, verify remaining features intact)
- [ ] T047 [US3] Component test: Edit feature in tests/integration/App.test.tsx (change feature type and costs, verify MD recalculates correctly)

### Implementation for User Story 3

- [ ] T048 [P] [US3] Add "Add Feature" button to InputView component (onClick handler adds new Feature object to state array)
- [ ] T049 [US3] Update FeatureRow component to include remove button (X icon) with onClick handler to remove feature from state array
- [ ] T050 [US3] Implement feature add logic in App component: generate unique ID (UUID), add to features array with empty type and 0 costs
- [ ] T051 [US3] Implement feature remove logic in App component: filter features array by ID, preserve other features
- [ ] T052 [US3] Make FeatureRow inputs controlled components: onChange handlers update feature in state array by ID
- [ ] T053 [US3] Add real-time calculation updates: recalculate output whenever features array or costs change
- [ ] T054 [US3] Add CSS for Add/Remove buttons in src/App.css (button styling, hover states)

**Checkpoint**: At this point, User Stories 1, 2, AND 3 should all work independently - dynamic feature management works

---

## Phase 6: User Story 4 - Clear Output Presentation (Priority: P3)

**Goal**: Enhance output table presentation with clear visual styling, proper column headers, and polished Total row appearance.

**Independent Test**: View output table and verify Total row has grey backgrounds for non-numeric cells, project name is prominent, and Support+Rest MD is calculated correctly.

### Tests for User Story 4 (Optional - visual/styling validation) ⚠️

- [ ] T055 [P] [US4] Visual test: Verify Total row grey backgrounds in tests/integration/App.test.tsx (check className or style attributes)
- [ ] T056 [P] [US4] Visual test: Verify project name appears at top of OutputView in tests/integration/App.test.tsx

### Implementation for User Story 4

- [ ] T057 [P] [US4] Style project name header in OutputView with larger font, bold weight, top margin (src/App.css)
- [ ] T058 [P] [US4] Add table borders, padding, and alternating row colors to MandayTable (src/App.css)
- [ ] T059 [P] [US4] Style TotalRow with bold text, top border, grey background (#e0e0e0) for Support and Rest MD cells (src/App.css)
- [ ] T060 [P] [US4] Add column header labels with "(MD)" suffix in MandayTable header row
- [ ] T061 [P] [US4] Ensure Support + Rest MD column always shows numeric sum in both feature rows and Total row (FR-026, FR-028)
- [ ] T062 [P] [US4] Add responsive design: table scrolls horizontally on narrow screens (src/App.css)

**Checkpoint**: All user stories should now be independently functional with polished presentation

---

## Phase 7: Validation & UX Polish

**Purpose**: Input validation, error handling, and user experience improvements

- [ ] T063 [P] Implement validateProjectName(name) in src/utils/validation.ts (return error if empty/whitespace)
- [ ] T064 [P] Implement validateCost(value) in src/utils/validation.ts (return error if negative, parse as 0 if empty)
- [ ] T065 [P] Implement validateFeature(feature) in src/utils/validation.ts (return error if type not selected)
- [ ] T066 Add inline validation to project name input in InputView: show error on blur if empty (FR-011)
- [ ] T067 Add inline validation to cost inputs in FeatureRow: show error on blur if negative (constitution principle)
- [ ] T068 Add form-level validation on "View Output" button click: check all features have types selected (FR-012)
- [ ] T069 Display validation error messages with role="alert" for screen readers
- [ ] T070 Add loading state handling: disable "View Output" button while validating
- [ ] T071 Add empty state handling: show message "No features added" if features array is empty when viewing output
- [ ] T072 Add edge case handling: display "0 MD" correctly when costs are zero (edge case from spec)
- [ ] T073 Add decimal number support: ensure calculations work with decimal cost values (edge case from spec)

---

## Phase 8: Accessibility & Documentation

**Purpose**: Ensure keyboard navigation, screen reader support, and complete documentation

- [ ] T074 [P] Add aria-label to "Add Feature" button: "Add new integration feature"
- [ ] T075 [P] Add aria-label to remove buttons in FeatureRow: "Remove feature"
- [ ] T076 [P] Add proper <label for="..."> elements to all form inputs (project name, costs)
- [ ] T077 [P] Verify keyboard navigation: Tab through all inputs, Enter to submit, Space for checkboxes
- [ ] T078 [P] Add focus management: focus first input on mount, focus first error on validation failure
- [ ] T079 [P] Test with screen reader: verify all interactive elements are announced correctly
- [ ] T080 [P] Add JSDoc comments to all calculation functions in src/lib/calculations.ts (inputs, outputs, formulas)
- [ ] T081 [P] Create README.md at repository root with setup instructions, pointing to specs/003-effort-estimator/quickstart.md
- [ ] T082 [P] Verify all TypeScript interfaces match contracts/README.md definitions
- [ ] T083 Run quickstart.md validation: follow setup steps, verify all commands work as documented

---

## Phase 9: Testing & Quality Assurance

**Purpose**: Achieve 100% calculation coverage and validate constitution compliance

- [ ] T084 Run test coverage report: verify src/lib/calculations.ts has 100% coverage (constitution requirement)
- [ ] T085 [P] Add additional edge case tests for calculation functions if coverage <100%
- [ ] T086 [P] Run linting: verify no ESLint errors with TypeScript rules
- [ ] T087 [P] Run TypeScript compiler: verify strict mode passes with no errors
- [ ] T088 Verify performance: calculations complete in <100ms for 20 features (constitution constraint)
- [ ] T089 Verify UI responsiveness: input changes update in <50ms (constitution constraint)
- [ ] T090 Test browser compatibility: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- [ ] T091 Manual testing: validate all acceptance scenarios from spec.md user stories
- [ ] T092 Constitution compliance check: verify all 7 principles satisfied (frontend-only, pure functions, tables, validation, integer MD, tests, docs)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-6)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P2 → P3)
- **Validation & UX (Phase 7)**: Can start after User Story 1 (P1) is complete
- **Accessibility (Phase 8)**: Can start after User Story 1 (P1) is complete
- **Testing & QA (Phase 9)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Builds on US1 but independently testable
- **User Story 3 (P2)**: Can start after Foundational (Phase 2) - Builds on US1 but independently testable
- **User Story 4 (P3)**: Can start after US1 - Enhances presentation but doesn't change functionality

### Within Each User Story

- Tests (marked ✅) MUST be written FIRST and FAIL before implementation
- TypeScript interfaces before implementation functions
- Calculation functions before UI components
- Parent components (App) before child components (InputView, OutputView)
- Basic functionality before styling
- Core features before edge case handling
- Story complete and passing all tests before moving to next priority

### Parallel Opportunities

**Phase 1 (Setup)**:

- T002, T003, T004, T006, T007 can all run in parallel

**Phase 2 (Foundational)**:

- T008, T009, T010, T011 can all run in parallel (different type files)

**Phase 3 (User Story 1)**:

- All test tasks T012-T016 can run in parallel (different test functions)
- T018, T019, T020, T021 can run in parallel (different calculation functions)
- T023, T024 can run in parallel (different component files)

**Phase 4 (User Story 2)**:

- T035, T036, T037 can run in parallel (different test scenarios)

**Phase 5 (User Story 3)**:

- T045, T046 can run in parallel (different test scenarios)
- T048, T049 can run in parallel (add button vs remove button)

**Phase 6 (User Story 4)**:

- T055, T056 can run in parallel (different visual tests)
- T057, T058, T059, T060, T061, T062 can all run in parallel (different CSS rules)

**Phase 7 (Validation)**:

- T063, T064, T065 can run in parallel (different validation functions)

**Phase 8 (Accessibility)**:

- T074, T075, T076, T077, T078, T079, T080, T081, T082 can all run in parallel (different files/concerns)

**Phase 9 (Testing)**:

- T085, T086, T087 can run in parallel (different validation tools)

**Cross-Phase Parallelization**:

- Once Phase 2 (Foundational) is complete, Phase 3 (US1), Phase 4 (US2), and Phase 5 (US3) can all start in parallel if team has 3+ developers
- Phase 7 and Phase 8 can run in parallel with Phase 4-6 (different concerns)

---

## Parallel Example: User Story 1 (MVP)

```bash
# After Phase 2 complete, launch all US1 tests together:
Task T012: Unit test calculateTotalMD in src/lib/calculations.test.ts
Task T013: Unit test getRolePercentages in src/lib/calculations.test.ts
Task T014: Unit test calculateRoleMD in src/lib/calculations.test.ts
Task T015: Unit test calculateRestMD in src/lib/calculations.test.ts
Task T016: Unit test calculateFeatureAllocation in src/lib/calculations.test.ts

# Then implement calculation functions in parallel:
Task T018: Implement calculateTotalMD in src/lib/calculations.ts
Task T019: Implement getRolePercentages in src/lib/calculations.ts
Task T020: Implement calculateRoleMD in src/lib/calculations.ts
Task T021: Implement calculateRestMD in src/lib/calculations.ts

# Then create component structure in parallel:
Task T023: Create App component in src/components/App.tsx
Task T024: Create InputView component in src/components/InputView.tsx
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T007)
2. Complete Phase 2: Foundational (T008-T011) - CRITICAL
3. Complete Phase 3: User Story 1 (T012-T034)
4. **STOP and VALIDATE**: Test User Story 1 independently
   - User can enter project name
   - User can input one feature with costs
   - User can view output with correct MD calculations
   - All tests pass (100% calculation coverage)
5. Deploy/demo MVP if ready

### Incremental Delivery

1. Complete Setup + Foundational (Phases 1-2) → Foundation ready
2. Add User Story 1 (Phase 3) → Test independently → Deploy/Demo (MVP! Core estimation works)
3. Add User Story 2 (Phase 4) → Test independently → Deploy/Demo (Role selection flexibility)
4. Add User Story 3 (Phase 5) → Test independently → Deploy/Demo (Dynamic feature management)
5. Add User Story 4 (Phase 6) → Test independently → Deploy/Demo (Polished presentation)
6. Complete Validation & UX (Phase 7) → Production-ready
7. Complete Accessibility (Phase 8) → Fully accessible
8. Complete Testing & QA (Phase 9) → Release

Each phase adds value without breaking previous functionality.

### Parallel Team Strategy

With 3 developers:

1. Team completes Setup (Phase 1) together
2. Team completes Foundational (Phase 2) together - CRITICAL GATE
3. Once Phase 2 is done:
   - **Developer A**: User Story 1 (Phase 3) - T012 to T034
   - **Developer B**: User Story 2 (Phase 4) - T035 to T044 (starts after own foundational work)
   - **Developer C**: User Story 3 (Phase 5) - T045 to T054 (starts after own foundational work)
4. Team converges for Phase 6 (US4 - polish) or continues parallel on Phases 7-9

With 2 developers:

1. Both complete Phases 1-2 together
2. Developer A: US1 (Phase 3), Developer B: US2 (Phase 4)
3. Developer A: US3 (Phase 5), Developer B: US4 (Phase 6)
4. Both complete Phases 7-9 together

---

## Notes

- **[P] tasks**: Different files, no dependencies - safe to parallelize
- **[Story] labels**: Map tasks to user stories for traceability and independent testing
- **Test-first**: All calculation tests (T012-T016) MUST be written before implementation (T018-T022)
- **Constitution compliance**: 100% calculation coverage is NON-NEGOTIABLE (verify with T084)
- **Tool-agnostic**: Build tool not specified - use team's preference (T001)
- **Testing framework**: React-compatible test runner (e.g., Jest) suggested but not mandatory (T003)
- **Independent stories**: Each user story should be completable and testable without the others
- **Commit frequency**: Commit after each task or logical group for rollback safety
- **Checkpoints**: Stop at each phase checkpoint to validate independently
- **Performance**: Validate calculation speed (T088) and UI responsiveness (T089) against constitution constraints
- **Accessibility**: Phase 8 ensures keyboard navigation and screen reader support per constitution

---

## Total Task Count

- **Phase 1 (Setup)**: 7 tasks
- **Phase 2 (Foundational)**: 4 tasks
- **Phase 3 (User Story 1 - P1 MVP)**: 23 tasks (6 tests + 17 implementation)
- **Phase 4 (User Story 2 - P2)**: 10 tasks (3 tests + 7 implementation)
- **Phase 5 (User Story 3 - P2)**: 10 tasks (3 tests + 7 implementation)
- **Phase 6 (User Story 4 - P3)**: 8 tasks (2 tests + 6 implementation)
- **Phase 7 (Validation & UX)**: 11 tasks
- **Phase 8 (Accessibility)**: 10 tasks
- **Phase 9 (Testing & QA)**: 9 tasks

**Total**: 92 tasks

**Parallel opportunities**: 45+ tasks can be executed in parallel (all marked [P])

**MVP scope** (Phases 1-3): 34 tasks to deliver core estimation functionality

**Production-ready scope** (Phases 1-9): All 92 tasks for complete, accessible, validated application
