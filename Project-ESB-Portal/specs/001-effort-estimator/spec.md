# Feature Specification: Integration Effort Estimator

**Feature Branch**: `001-effort-estimator`  
**Created**: 2025-11-18  
**Status**: Draft  
**Input**: User description: "Integration Effort Estimator web application for calculating Mandays from integration features"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Calculate Single Feature Effort (Priority: P1)

A project manager enters one integration feature with its costs and immediately sees the calculated mandays and role-based effort distribution. This is the core value proposition—quick effort estimation for a single feature.

**Why this priority**: This is the minimum viable product. Even with just one feature calculation, users get immediate value and can validate the calculation formulas. All other functionality builds on this foundation.

**Independent Test**: Can be fully tested by entering project name, adding one feature row with costs, and verifying that the estimated MD appears in the input section and the role distribution table appears in the output section with correct calculations.

**Acceptance Scenarios**:

1. **Given** the user opens the application, **When** they enter a project name "API Integration Project" and add one "Use Case / API" feature with Project cost 10,000 THB and Hardware cost 0 THB, **Then** the system displays Estimated efforts = 2 MD and shows the output table with SA=0 MD, QA=0 MD, BE=0 MD, Support=1 MD, Rest MD=1 MD, Support+Rest MD=2 MD

2. **Given** a user has entered costs, **When** they modify the Project cost from 10,000 to 15,000 THB, **Then** the Estimated efforts updates to 3 MD and the output table recalculates all role distributions immediately

3. **Given** a user enters costs, **When** they select different feature types (e.g., switching from "Use Case / API" to "Data Synchronization"), **Then** the output table updates with the correct percentage allocations for that feature type

---

### User Story 2 - Manage Multiple Integration Features (Priority: P2)

A project manager can add, view, and calculate efforts for multiple integration features in a single project, seeing individual calculations for each feature plus a summary total row.

**Why this priority**: Real projects have multiple integration points. This story enables realistic project planning but depends on the single-feature calculation working correctly first.

**Independent Test**: Can be tested by adding 3-5 features of different types with varying costs, verifying that each row calculates independently, and confirming the summary row totals all MD columns correctly.

**Acceptance Scenarios**:

1. **Given** the user has entered one feature, **When** they click "Add Feature" (or similar action), **Then** a new empty feature row appears with all required fields

2. **Given** multiple features exist (e.g., 2 Use Case APIs, 1 Data Sync, 1 Batch), **When** calculations complete, **Then** the output table shows one row per feature with correct role distributions, plus a "Total" summary row summing all MD columns

3. **Given** multiple features exist, **When** the user deletes the second feature row, **Then** that feature disappears from the output table and the Total row recalculates excluding it

---

### User Story 3 - Handle Empty and Invalid Inputs Gracefully (Priority: P3)

Users receive clear validation feedback when inputs are missing or invalid, with empty cost fields treated as zero to reduce friction.

**Why this priority**: Improves user experience and data quality but isn't required to demonstrate core calculation functionality. Can be added after basic calculations work.

**Independent Test**: Can be tested by leaving cost fields empty (should treat as 0), entering negative numbers (should show error), entering non-numeric text (should show error), and omitting required fields like feature type (should show error or disable calculation).

**Acceptance Scenarios**:

1. **Given** a user adds a feature, **When** they leave Project cost and Hardware cost empty, **Then** the system treats both as 0, calculates Estimated efforts = 0 MD, and displays the output row with all zero values

2. **Given** a user enters a negative value (-500) in Hardware cost, **When** they blur the field or attempt calculation, **Then** an inline error message appears stating "Hardware cost cannot be negative" and the calculation does not proceed

3. **Given** a user enters non-numeric text ("abc") in Project cost, **When** they blur the field, **Then** an inline error message appears stating "Project cost must be a valid number" and the field is highlighted

4. **Given** a user adds a feature row, **When** they leave the Feature type dropdown unselected, **Then** the system either prevents calculation with a message "Feature type is required" or disables the output for that row until a type is selected

---

### User Story 4 - Configure Visible Roles and Navigate to Output (Priority: P2)

A project manager selects which roles (System Analyst, Quality Assurance, Back-End Developer, Client Support Operation) are in scope for a project and then switches to a dedicated Output page to review the manday distribution only for those relevant roles. This allows the estimator to fit different staffing models without changing the underlying business rules.

**Why this priority**: Provides flexibility for different organizational structures where not all roles may be relevant to every project. Enables cleaner output presentation by hiding irrelevant columns. Depends on core calculation functionality (US1) but enhances usability alongside US2.

**Independent Test**: With several integration features entered, the user can toggle role checkboxes and switch between Input and Output views. The Output page immediately updates the visible columns and total row for the selected roles while keeping Estimated MD, Rest MD, and Support + Rest MD correct.

**Acceptance Scenarios**:

1. **Given** all four roles are checked by default, **When** the user unchecks 'Quality Assurance' in the Input view, **Then** the Output page hides the QA (MD) column for all feature rows and for the Total row, while SA/BE/Support, Rest MD, and Support + Rest MD remain visible and correctly calculated

2. **Given** at least one role is currently selected, **When** the user attempts to uncheck the last remaining role, **Then** the system blocks this action or immediately re-enables that role and shows a message such as 'At least one role must be selected.'

3. **Given** features and costs have already been entered in the Input view, **When** the user navigates to the Output page, **Then** the Output page displays the latest calculations; when the user navigates back to Input and edits costs or feature types, and then returns to Output, all role allocations and totals are updated without losing data

---

### Edge Cases

- **Zero costs**: When both Project cost and Hardware cost are 0 or empty, Estimated efforts should be 0 MD, and all role MD values should be 0
- **Large numbers**: System should handle costs in the millions (e.g., 50,000,000 THB) without performance degradation or display issues
- **Fractional costs**: If users enter fractional THB (e.g., 12,345.67 THB), calculations should still work correctly with ROUNDDOWN behavior
- **Rounding edge cases**: When E \* percentage results in a fraction, ROUNDDOWN ensures integer MD; Rest MD absorbs any remainder after role allocations
- **All roles zero**: For some small Estimated efforts values and percentage combinations, all role MD values might round down to 0, leaving all effort in Rest MD
- **Feature type changes**: When switching feature type with existing costs, role percentages change but Estimated efforts (E) remains constant
- **Project name length**: Very long project names should wrap or truncate gracefully in the UI
- **Maximum features**: System should handle at least 50 integration features without performance issues (per Constitution: <100ms calculation time)

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST provide a text input field for project name (required field)
- **FR-002**: System MUST allow users to add multiple integration feature rows dynamically
- **FR-003**: System MUST provide a dropdown for Feature type with exactly 5 options: "Use Case / API", "Topic / Event", "Data Synchronization", "Batch / File", "ESB Feature"
- **FR-004**: System MUST provide numeric input fields for "Summation of Project cost (THB)" and "Summation of Hardware cost (THB)" for each feature row
- **FR-005**: System MUST treat empty cost fields as zero (0) in all calculations
- **FR-006**: System MUST validate that cost fields are non-negative numbers and display inline error messages for negative or invalid inputs
- **FR-007**: System MUST calculate Estimated efforts (MD) for each feature using the formula: `MD = floor((Project cost + Hardware cost) / 5000)`
- **FR-008**: System MUST use JavaScript `Math.floor()` for all ROUNDDOWN operations to ensure consistent rounding behavior
- **FR-009**: System MUST calculate role-based MD allocations using feature-type-specific percentages applied to Estimated efforts (E) with ROUNDDOWN for each multiplication
- **FR-010**: System MUST calculate Rest MD as: `Rest MD = E - (SA + QA + BE + Support)` for each feature
- **FR-011**: System MUST calculate Support + Rest MD as: `Support + Rest MD = Support + Rest MD` for each feature
- **FR-012**: System MUST display an output table with columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD
- **FR-013**: System MUST display one row per feature in the output table showing the feature type and calculated MD values
- **FR-014**: System MUST display a "Total" summary row at the bottom of the output table summing each MD column across all features
- **FR-015**: System MUST recalculate all output values immediately when any input value changes (real-time reactivity)
- **FR-016**: System MUST use the following percentage allocations for "Use Case / API" type: SA=30%, QA=0%, BE=0%, Support=70%
- **FR-017**: System MUST use the following percentage allocations for "Topic / Event" type: SA=30%, QA=0%, BE=0%, Support=70%
- **FR-018**: System MUST use the following percentage allocations for "Data Synchronization" type: SA=30%, QA=20%, BE=20%, Support=30%
- **FR-019**: System MUST use the following percentage allocations for "Batch / File" type: SA=30%, QA=20%, BE=20%, Support=30%
- **FR-020**: System MUST use the following percentage allocations for "ESB Feature" type: SA=20%, QA=30%, BE=20%, Support=30%
- **FR-021**: System MUST provide a Role Selection control in the Input section with four checkboxes: 'System Analyst', 'Quality Assurance', 'Back-End Developer', and 'Client Support Operation'. All four MUST be selected by default when the application loads.
- **FR-022**: System MUST ensure that at least one role checkbox is selected at all times. If the user attempts to deselect the final remaining role, the system MUST prevent that state and display an inline message such as 'At least one role must be selected.'
- **FR-023**: System MUST respect role selection when rendering the output table. The Manday output table MUST only display columns for the roles that are currently selected. 'Rest MD' and 'Support + Rest MD' columns MUST always be visible regardless of role selection.
- **FR-024**: System MUST provide simple navigation between an Input view and an Output view within the same React SPA. The default landing view MUST be the Input view. From the Input view the user can navigate to the Output view (e.g., via a button labeled 'View Output'), and from the Output view they can return to the Input view (e.g., via 'Back to Input').
- **FR-025**: Role selection MUST NOT change the underlying manday calculations. Estimated efforts (MD) and role allocations (SA, QA, BE, Support, Rest MD, Support + Rest MD) MUST still be calculated using the existing formulas and feature-type-specific percentages; role selection only controls which role columns are visible in the output table.

### Key Entities

- **Project**: Represents an integration project with a name and multiple integration features. Attributes: name (string, required)

- **Integration Feature**: Represents a single integration point or feature to be estimated. Attributes:

  - Feature type (enum: Use Case/API, Topic/Event, Data Synchronization, Batch/File, ESB Feature)
  - Project cost (THB): non-negative number, defaults to 0 if empty
  - Hardware cost (THB): non-negative number, defaults to 0 if empty
  - Estimated efforts (MD): calculated integer, derived from costs

- **Role Allocation**: Represents the distribution of mandays across roles for a feature. Attributes:

  - System Analyst MD (integer)
  - Quality Assurance MD (integer)
  - Back-End Developer MD (integer)
  - Client Support Operation MD (integer)
  - Rest MD (integer, absorbs rounding remainder)
  - Support + Rest MD (integer, combined support effort)

- **Role Visibility**: Represents which standard roles are currently displayed in the output table. Attributes:
  - visibleRoles: array of role keys (sa, qa, be, support) with at least one entry at all times

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Users can enter a project with 5 integration features and see calculated role-based efforts in under 5 seconds from opening the application

- **SC-002**: Calculation results are accurate to the formula specification in 100% of test cases (validated through comprehensive unit tests)

- **SC-003**: Users can add up to 20 integration features without the calculation taking longer than 100ms (per Constitution performance requirement)

- **SC-004**: 90% of users successfully complete their first effort estimation (project name + at least one feature with valid costs) without external help or error recovery

- **SC-005**: All input validation errors provide clear, actionable messages that enable users to correct issues on the first attempt

- **SC-006**: The output table is readable and understandable by non-technical project managers without requiring calculation formula knowledge

- **SC-007**: Changes to input costs reflect in the output table within 100ms, providing real-time feedback

## Assumptions

- **Currency**: All costs are in Thai Baht (THB); no currency conversion needed
- **Divisor constant**: The 5,000 THB divisor in the MD formula is a fixed business rule and will not need to be configurable in v1.0
- **Role definitions**: The four roles (SA, QA, BE, Support) are standard across all projects; custom roles not needed in v1.0
- **Percentage allocations**: The five feature types and their percentage allocations are comprehensive and fixed; no custom feature types needed in v1.0
- **Role visibility**: The four standard roles (SA, QA, BE, Support) remain fixed for calculations, but users MAY toggle the visibility of each role in the output table per project. Role visibility is a presentation concern only and does not alter the underlying percentage allocations or formulas.
- **Data persistence**: In v1.0, data only needs to persist during the browser session (in-memory state); no save/load functionality required unless explicitly requested
- **Browser support**: Modern browsers (Chrome, Firefox, Safari, Edge) from the last 2 years are the target; no IE11 support needed
- **User authentication**: Not required for v1.0; application is accessible without login
- **Collaboration**: Single-user experience; no multi-user editing or sharing features in v1.0
- **Export functionality**: Not required for v1.0 unless explicitly requested; users can manually copy data from the output table
- **Input limits**: Maximum 50 features per project is a reasonable upper bound for v1.0 performance testing

## Open Questions

None at this time. All requirements are sufficiently clear for implementation based on the detailed user input and constitution principles.
