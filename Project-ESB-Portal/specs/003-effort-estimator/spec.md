# Feature Specification: Integration Effort Estimator

**Feature Branch**: `003-effort-estimator`  
**Created**: November 20, 2025  
**Status**: Draft  
**Input**: User description: "Integration Effort Estimator web application"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - Calculate Project Effort (Priority: P1)

A project manager needs to estimate the total mandays required for a new integration project. They input the project name and add integration features with their associated costs, then view a breakdown showing how effort is distributed across different roles.

**Why this priority**: This is the core value proposition of the application - providing quick effort estimates based on feature types and costs. Without this, the application has no purpose.

**Independent Test**: Can be fully tested by entering project details, adding multiple features with costs, and verifying that the calculated mandays match the expected formula (cost/5000 rounded up) and role allocations follow the specified percentages.

**Acceptance Scenarios**:

1. **Given** an empty input form, **When** user enters "Customer Portal Integration" as project name and adds a "Use Case / API" feature with Project cost 15000 THB and Hardware cost 10000 THB, **Then** the system displays estimated effort as 5 MD (ROUNDUP((15000+10000)/5000)).

2. **Given** a project with one feature added, **When** user adds a second feature "Data Synchronization" with Project cost 20000 THB and Hardware cost 5000 THB, **Then** the system displays total effort as 5 MD + 5 MD = 10 MD.

3. **Given** a completed input form with multiple features, **When** user navigates to the output page, **Then** the system displays a manday breakdown table with one row per feature plus a total summary row.

4. **Given** a "Use Case / API" feature with 10 MD, **When** viewing the output table with all roles selected, **Then** SA column shows 3 MD (ROUNDDOWN(10 _ 0.30)), QA shows 0 MD, BE shows 0 MD, Support shows 7 MD (ROUNDDOWN(10 _ 0.70)), and Rest MD shows 0 MD (10 - 3 - 0 - 0 - 7).

5. **Given** a "Data Synchronization" feature with 10 MD, **When** viewing output with all roles selected, **Then** SA shows 3 MD, QA shows 2 MD, BE shows 2 MD, Support shows 3 MD, and Rest MD shows 0 MD (10 - 3 - 2 - 2 - 3).

---

### User Story 2 - Selective Role Allocation (Priority: P2)

A project manager needs to see effort estimates for only the roles available on their team. They can uncheck roles they don't need (e.g., no QA team available), and the system adjusts the display to show only selected roles plus the "Rest MD" for unallocated effort.

**Why this priority**: Provides flexibility for different team compositions and project constraints. Essential for practical use but the application still provides value with all roles selected (P1 scenario).

**Independent Test**: Can be fully tested by selecting different role combinations and verifying that only selected role columns appear in the output table, and that Rest MD correctly shows the unallocated effort.

**Acceptance Scenarios**:

1. **Given** the input form with all roles checked by default, **When** user unchecks "Quality Assurance" and "Back-End Developer", **Then** only System Analyst, Client Support Operation, Rest MD, and Support + Rest MD columns appear in the output table.

2. **Given** a "Data Synchronization" feature with 10 MD and only SA selected, **When** viewing output, **Then** SA shows 3 MD and Rest MD shows 7 MD (10 - 3).

3. **Given** a "ESB Feature" with 10 MD and SA + QA selected, **When** viewing output, **Then** SA shows 2 MD, QA shows 3 MD, and Rest MD shows 5 MD (10 - 2 - 3).

4. **Given** a user attempting to uncheck all roles, **When** unchecking the last selected role, **Then** the system prevents unchecking and displays a validation message "At least one role must be selected".

5. **Given** multiple features with different role selections, **When** viewing the output table, **Then** the Total summary row displays summed values for each selected role and the combined Rest MD.

---

### User Story 3 - Manage Multiple Features (Priority: P2)

A project manager needs to add, edit, or remove integration features from their estimate as project scope changes. They can modify the features table dynamically without losing other entered data.

**Why this priority**: Enables iterative estimation and scope adjustments. Improves usability but the core estimation (P1) works with static feature entry.

**Independent Test**: Can be fully tested by adding multiple features, editing their values, removing features, and verifying that calculations update correctly and no data is lost unexpectedly.

**Acceptance Scenarios**:

1. **Given** an empty features table, **When** user clicks "Add Feature", **Then** a new row appears with empty Feature Type dropdown and cost input fields.

2. **Given** a feature row with data entered, **When** user changes the Feature Type from "Use Case / API" to "Batch / File", **Then** the calculated MD updates if costs changed, and output table reflects new role allocation percentages.

3. **Given** three features in the table, **When** user removes the second feature, **Then** the remaining two features remain intact with their data preserved, and total MD recalculates excluding the removed feature.

4. **Given** a feature with costs entered, **When** user modifies the Project cost from 10000 to 15000 THB, **Then** the estimated MD updates immediately using the formula ROUNDUP((15000 + Hardware cost) / 5000).

---

### User Story 4 - Clear Output Presentation (Priority: P3)

A project manager needs to understand the effort breakdown at a glance, with clear visual separation between calculated role effort and unallocated "Rest MD". The total summary row provides aggregate figures while maintaining visual clarity for non-summed cells.

**Why this priority**: Enhances usability and comprehension but doesn't change core functionality. The P1 scenario already provides the data; this improves how it's presented.

**Independent Test**: Can be fully tested by viewing the output table and verifying that the Total summary row has appropriate styling (grey background for non-numeric cells) and that column visibility matches role selection.

**Acceptance Scenarios**:

1. **Given** an output table with multiple features, **When** viewing the Total summary row, **Then** the Client Support Operation and Rest MD cells display no numeric value and have grey background color.

2. **Given** an output table, **When** viewing any feature row, **Then** the "Support + Rest MD" column shows the sum of that feature's Support MD and Rest MD values.

3. **Given** a completed estimation, **When** viewing the output page, **Then** the project name from the input form appears prominently at the top of the output view.

4. **Given** the output table, **When** comparing feature rows to the Total summary row, **Then** all selected role columns show summed totals and Support + Rest MD shows the sum of all features' Support + Rest MD.

---

### Edge Cases

- What happens when Project cost and Hardware cost are both zero? (Expected: 0 MD, but ROUNDUP(0/5000) = 0)
- What happens when costs are entered with decimal values (e.g., 12345.67 THB)? (Expected: System accepts decimals, calculation uses full precision before rounding)
- What happens when the sum of ROUNDDOWN(role percentages) leaves a large Rest MD? (Expected: Rest MD accurately reflects unallocated effort)
- What happens when a user deselects all roles except one, then tries to deselect that last role? (Expected: Validation prevents unchecking; at least one role must remain selected)
- What happens when no features are added? (Expected: Output page shows project name but empty table or message "No features added")
- What happens when Feature Type is not selected for a row? (Expected: Validation prevents navigation to output or displays error message)
- What happens when costs are negative? (Expected: Input validation prevents negative values or displays error)

## Requirements _(mandatory)_

### Functional Requirements

#### Input Page Requirements

- **FR-001**: System MUST provide a text input field for Project Name that is required before proceeding to output.
- **FR-002**: System MUST provide a dynamic table for Integration Features where users can add multiple feature rows.
- **FR-003**: Each feature row MUST include a dropdown to select Feature Type with options: "Use Case / API", "Topic / Event", "Data Synchronization", "Batch / File", "ESB Feature".
- **FR-004**: Each feature row MUST include input fields for Project cost (THB) and Hardware cost (THB), both accepting numeric values.
- **FR-005**: System MUST calculate estimated effort in mandays (MD) for each feature using the formula: MD = ROUNDUP((Project cost + Hardware cost) / 5000).
- **FR-006**: System MUST provide a Role Selection section with checkboxes for: System Analyst (SA), Quality Assurance (QA), Back-End Developer (BE), Client Support Operation.
- **FR-007**: All role checkboxes MUST be checked by default when the input page loads.
- **FR-008**: System MUST enforce that at least one role remains selected at all times (prevent unchecking all roles).
- **FR-009**: System MUST allow users to add new feature rows dynamically.
- **FR-010**: System MUST allow users to remove existing feature rows.
- **FR-011**: System MUST validate that Project Name is not empty before allowing navigation to output page.
- **FR-012**: System MUST validate that all feature rows have a Feature Type selected before allowing navigation to output page.

#### Output Page Requirements

- **FR-013**: System MUST display the Project Name at the top of the output view.
- **FR-014**: System MUST display a manday breakdown table with the following columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD.
- **FR-015**: System MUST show only the columns corresponding to selected roles, while Rest MD and Support + Rest MD columns MUST always be visible.
- **FR-016**: System MUST display one row per feature with the feature type name in the Feature column.
- **FR-017**: System MUST display a Total summary row at the bottom of the table.
- **FR-018**: System MUST calculate role-specific mandays for each feature using ROUNDDOWN(total_MD \* role_percentage).

#### Role Allocation Rules

- **FR-019**: For "Use Case / API" features, system MUST allocate effort as: SA 30%, QA 0%, BE 0%, Support 70%.
- **FR-020**: For "Topic / Event" features, system MUST allocate effort as: SA 30%, QA 0%, BE 0%, Support 70%.
- **FR-021**: For "Data Synchronization" features, system MUST allocate effort as: SA 30%, QA 20%, BE 20%, Support 30%.
- **FR-022**: For "Batch / File" features, system MUST allocate effort as: SA 30%, QA 20%, BE 20%, Support 30%.
- **FR-023**: For "ESB Feature" features, system MUST allocate effort as: SA 20%, QA 30%, BE 20%, Support 30%.

#### Calculation Rules

- **FR-024**: For each feature, system MUST calculate roleMD = ROUNDDOWN(total_MD \* role_percentage) for each selected role.
- **FR-025**: System MUST calculate Rest MD = total_MD - (sum of all selected role MD values).
- **FR-026**: System MUST calculate Support + Rest MD = Support MD + Rest MD for each feature.
- **FR-027**: In the Total summary row, system MUST sum all role MD values across all features for each selected role.
- **FR-028**: In the Total summary row, system MUST display the sum of all Support + Rest MD values across all features.
- **FR-029**: In the Total summary row, system MUST NOT display numeric values for Client Support Operation MD and Rest MD cells.
- **FR-030**: In the Total summary row, system MUST set background color to grey for Client Support Operation and Rest MD cells.

### Key Entities

- **Project**: Represents the overall integration project with a unique name and a collection of features.
- **Integration Feature**: Represents a specific integration task with a type (Use Case/API, Topic/Event, Data Synchronization, Batch/File, ESB Feature), project cost, hardware cost, and calculated manday estimate.
- **Role**: Represents a team role (System Analyst, Quality Assurance, Back-End Developer, Client Support Operation) with selection state and allocation percentage based on feature type.
- **Manday Allocation**: Represents the calculated effort distribution for a feature across selected roles, including allocated MD per role, Rest MD (unallocated), and Support + Rest MD.

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: Users can complete a full effort estimation (from empty form to viewing output) in under 3 minutes for a project with 5 features.
- **SC-002**: The application calculates and displays manday estimates instantly (under 500ms) after users modify costs or feature types.
- **SC-003**: 95% of manday calculations match manual spreadsheet calculations when verified with the same input data.
- **SC-004**: Users can successfully add up to 20 features without performance degradation or usability issues.
- **SC-005**: The output table correctly hides/shows role columns within 100ms of role selection changes.
- **SC-006**: 100% of edge cases (zero costs, decimal values, single role selection) are handled without application errors or crashes.
- **SC-007**: The Total summary row correctly sums role allocations and displays non-numeric cells with grey background for 100% of test cases.
