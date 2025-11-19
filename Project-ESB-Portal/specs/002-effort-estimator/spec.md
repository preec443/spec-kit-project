# Feature Specification: Integration Effort Estimator

**Feature Branch**: `002-effort-estimator`  
**Created**: 2025-11-19  
**Status**: Draft  
**Priority**: P1 (High)

## Overview

The Integration Effort Estimator is a web-based tool that helps project managers calculate and visualize effort distribution (in mandays) for integration projects. Users input integration feature details with associated costs, and the system automatically calculates estimated mandays with role-based effort allocation across System Analyst, Quality Assurance, Back-End Developer, and Client Support Operation roles.

## Problem Statement

Integration project planning requires accurate effort estimation distributed across multiple technical roles. Manual calculation of mandays from costs and subsequent role-based allocation is time-consuming, error-prone, and difficult to visualize. Teams need a quick, reliable tool that:

- Converts project and hardware costs into estimated mandays using a standardized formula
- Automatically distributes effort across relevant roles based on integration feature type
- Provides flexibility to focus on specific roles that are in scope for a project
- Clearly separates data input from results visualization

## User Stories

### User Story 1: Calculate Effort for Integration Features

**As a** project manager  
**I want to** input integration feature types with their associated costs  
**So that** I can quickly estimate total mandays and see role-based effort distribution

**Acceptance Criteria**:

1. User can enter a project name (required field)
2. User can add multiple integration features to a table, where each feature has:
   - Feature Type selection (dropdown with 5 options)
   - Project Cost in Thai Baht (numeric input, non-negative)
   - Hardware Cost in Thai Baht (numeric input, non-negative)
3. System calculates Estimated Mandays (MD) for each feature using: `MD = ROUNDDOWN((Project Cost + Hardware Cost) / 5000)`
4. All estimated MD values are displayed as integers (no decimals)
5. User can delete features from the table
6. Empty cost fields are treated as 0 in calculations

**Acceptance Scenarios**:

- **Scenario 1**: User adds "Use Case / API" feature with Project cost 10,000 THB and Hardware cost 0 THB → System displays Estimated MD = 2
- **Scenario 2**: User adds "Data Synchronization" feature with costs 50,000 and 0 → System displays Estimated MD = 10
- **Scenario 3**: User enters 12,500 THB total cost → System displays Estimated MD = 2 (rounds down from 2.5)

### User Story 2: Control Role Visibility

**As a** project manager  
**I want to** select which roles are relevant to my project  
**So that** I can focus the output table on only the roles I need to plan for

**Acceptance Criteria**:

1. Role Selection section displays four checkboxes:
   - System Analyst (SA)
   - Quality Assurance (QA)
   - Back-End Developer (BE)
   - Client Support Operation (Support)
2. All four checkboxes are checked by default when the application loads
3. User can check/uncheck any role checkbox
4. At least one role must remain selected at all times
5. When user attempts to uncheck the last remaining role, the system prevents the action and shows an inline message: "At least one role must be selected."
6. Selected roles determine which columns appear in the Output view

**Acceptance Scenarios**:

- **Scenario 1**: User unchecks "QA" and "BE" → Output table shows only SA and Support columns (plus always-visible columns)
- **Scenario 2**: User has only "SA" selected and tries to uncheck it → Checkbox remains checked, inline message appears
- **Scenario 3**: User checks all four roles → Output table displays all role columns

### User Story 3: View Detailed Role-Based Effort Distribution

**As a** project manager  
**I want to** navigate to a separate Output view that shows manday allocation per role  
**So that** I can understand how effort is distributed across different technical functions

**Acceptance Criteria**:

1. Input view provides a "View Output" button to navigate to Output view
2. Output view displays a Manday table with the following columns:
   - Feature (always visible)
   - SA (MD) - visible only if SA role is selected
   - QA (MD) - visible only if QA role is selected
   - BE (MD) - visible only if BE role is selected
   - Support (MD) - visible only if Support role is selected
   - Rest MD (always visible)
   - Support + Rest MD (always visible)
3. Table displays one row per feature showing calculated mandays for each role
4. Table includes a Total row summing all columns
5. Output view provides a "Back to Input" button to return to Input view
6. All role MD values are integers (no decimals)
7. Navigation between views preserves all entered data

**Role Allocation Formula** (per feature type):

For each feature with Estimated MD = E:

- **Use Case / API**:

  - SA = ROUNDDOWN(E × 0.3)
  - QA = ROUNDDOWN(E × 0.0) = 0
  - BE = ROUNDDOWN(E × 0.0) = 0
  - Support = ROUNDDOWN(E × 0.7)

- **Topic / Event**:

  - SA = ROUNDDOWN(E × 0.3)
  - QA = ROUNDDOWN(E × 0.0) = 0
  - BE = ROUNDDOWN(E × 0.0) = 0
  - Support = ROUNDDOWN(E × 0.7)

- **Data Synchronization**:

  - SA = ROUNDDOWN(E × 0.3)
  - QA = ROUNDDOWN(E × 0.2)
  - BE = ROUNDDOWN(E × 0.2)
  - Support = ROUNDDOWN(E × 0.3)

- **Batch / File**:

  - SA = ROUNDDOWN(E × 0.3)
  - QA = ROUNDDOWN(E × 0.2)
  - BE = ROUNDDOWN(E × 0.2)
  - Support = ROUNDDOWN(E × 0.3)

- **ESB Feature**:
  - SA = ROUNDDOWN(E × 0.2)
  - QA = ROUNDDOWN(E × 0.3)
  - BE = ROUNDDOWN(E × 0.2)
  - Support = ROUNDDOWN(E × 0.3)

**Derived Calculations**:

- Rest MD = E - (SA + QA + BE + Support)
- Support + Rest MD = Support + Rest MD

**Acceptance Scenarios**:

- **Scenario 1**: "Use Case / API" with E=2 → SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2
- **Scenario 2**: "Data Synchronization" with E=10 → SA=3, QA=2, BE=2, Support=3, Rest=0, Support+Rest=3
- **Scenario 3**: Multiple features → Total row correctly sums each column

### User Story 4: Handle Invalid Input Gracefully

**As a** user  
**I want to** receive clear feedback when I enter invalid data  
**So that** I can correct mistakes quickly without confusion

**Acceptance Criteria**:

1. Empty cost fields are treated as 0 (no error shown)
2. Negative cost values trigger validation error with message: "Cost cannot be negative"
3. Non-numeric input triggers validation error with message: "Must be a valid number"
4. Invalid input fields are highlighted with red border
5. Error messages appear inline below the relevant input field
6. Validation triggers on blur (focus leaves the field)
7. User can correct invalid input, and error clears when valid value is entered
8. Feature type dropdown is required; unselected type prevents calculation

**Acceptance Scenarios**:

- **Scenario 1**: User leaves both cost fields empty → Estimated MD shows 0, no error
- **Scenario 2**: User enters -500 in Project cost → Red error appears: "Cost cannot be negative"
- **Scenario 3**: User enters "abc" in Hardware cost → Red error appears: "Must be a valid number"
- **Scenario 4**: User corrects invalid input to valid value → Error message disappears

## Functional Requirements

### Input Management

- **FR-001**: System shall provide a text input field for Project Name marked as required
- **FR-002**: System shall allow users to add multiple integration features dynamically
- **FR-003**: System shall provide a dropdown for Feature Type with exactly 5 options: "Use Case / API", "Topic / Event", "Data Synchronization", "Batch / File", "ESB Feature"
- **FR-004**: System shall provide numeric input fields for Project Cost (THB) and Hardware Cost (THB) per feature
- **FR-005**: System shall treat empty cost fields as numeric value 0
- **FR-006**: System shall validate that cost fields contain non-negative numbers only
- **FR-007**: System shall calculate Estimated MD using formula: `ROUNDDOWN((Project Cost + Hardware Cost) / 5000)`
- **FR-008**: System shall use ROUNDDOWN function (equivalent to Math.floor() in JavaScript) for all manday calculations

### Role Selection

- **FR-009**: System shall display four role checkboxes: System Analyst, Quality Assurance, Back-End Developer, Client Support Operation
- **FR-010**: System shall check all four role checkboxes by default on application load
- **FR-011**: System shall enforce constraint: at least one role must remain selected at all times
- **FR-012**: System shall prevent unchecking the last remaining role and display inline message: "At least one role must be selected."
- **FR-013**: System shall use selected roles to control column visibility in Output view

### Output Display

- **FR-014**: System shall provide navigation between Input view and Output view within a single-page application
- **FR-015**: System shall display a Manday table in Output view with columns: Feature, SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD
- **FR-016**: System shall conditionally display role columns (SA, QA, BE, Support) based on role selection; Rest MD and Support + Rest MD are always visible
- **FR-017**: System shall display one table row per feature showing role-based manday allocations
- **FR-018**: System shall display a Total summary row summing all manday columns
- **FR-019**: System shall preserve all input data when navigating between Input and Output views

### Calculation Rules

- **FR-020**: System shall calculate role allocations using feature-type-specific percentages:

  - **Use Case / API**: SA 30%, QA 0%, BE 0%, Support 70%
  - **Topic / Event**: SA 30%, QA 0%, BE 0%, Support 70%
  - **Data Synchronization**: SA 30%, QA 20%, BE 20%, Support 30%
  - **Batch / File**: SA 30%, QA 20%, BE 20%, Support 30%
  - **ESB Feature**: SA 20%, QA 30%, BE 20%, Support 30%

- **FR-021**: System shall calculate each role's MD as: `ROUNDDOWN(Estimated MD × role percentage)`
- **FR-022**: System shall calculate Rest MD as: `Estimated MD - (SA + QA + BE + Support)`
- **FR-023**: System shall calculate Support + Rest MD as: `Support MD + Rest MD`
- **FR-024**: System shall display all MD values as integers (no decimal places)
- **FR-025**: System shall calculate Total row by summing each column across all features

## Success Criteria

- **SC-001**: User can input project details and features, calculating mandays in under 30 seconds (user task completion time)
- **SC-002**: Calculations are mathematically accurate: 100% of test cases pass with expected integer results
- **SC-003**: Application calculates and displays results for 20 features in under 100 milliseconds
- **SC-004**: Role selection changes update Output view visibility instantly (< 200ms perceived latency)
- **SC-005**: Zero calculation errors reported in user acceptance testing (10+ test scenarios)
- **SC-006**: Keyboard-only users can navigate entire application without mouse
- **SC-007**: Application loads initial page in under 2 seconds on 3G connection

## Non-Functional Requirements

### Performance

- Calculation performance: < 100ms for up to 50 features
- Initial page load: < 2 seconds on 3G network
- No UI blocking: calculations complete without freezing interface

### Usability

- Clear visual distinction between Input and Output views
- Inline validation feedback appears within 100ms of blur event
- Error messages use plain language (no technical jargon)
- All interactive elements are keyboard-accessible with logical tab order

### Accessibility

- Semantic HTML tables for all tabular data
- ARIA labels for all form inputs
- Keyboard navigation for all interactive elements
- Screen reader compatible (tested with VoiceOver/NVDA)

### Browser Compatibility

- Support for modern browsers: Chrome, Firefox, Safari, Edge (last 2 years)
- Responsive design for desktop screens (1024px width minimum)

## Technical Constraints

- **Frontend-only architecture**: No backend services or APIs in version 1.0
- **React + TypeScript**: Single-page application built with React 18+ and TypeScript 5.3+
- **Pure function calculations**: All business logic implemented as pure functions without side effects
- **Build tool agnostic**: Implementation should not be locked to specific build tools (CRA, Next.js, etc.)
- **Test coverage**: 100% unit test coverage for calculation functions; component tests for main user flows
- **In-memory state**: All data stored in client memory; no persistence in version 1.0

## Assumptions

- Users understand Thai Baht (THB) currency context
- Users are familiar with integration project terminology (API, ESB, batch processing)
- Maximum 50 features per project (performance boundary)
- Cost values will not exceed JavaScript's safe integer limit (2^53 - 1)
- Users have access to modern web browsers with JavaScript enabled
- Role percentages are fixed and do not require user configuration
- Formula divisor (5000) is constant and not user-configurable

## Key Entities

### Integration Feature

- **Attributes**: Feature Type, Project Cost (THB), Hardware Cost (THB), Estimated MD
- **Validation**: Feature Type required, costs must be non-negative numbers
- **Lifecycle**: Created by user, can be deleted, data not persisted

### Role Allocation

- **Attributes**: SA (MD), QA (MD), BE (MD), Support (MD), Rest MD, Support + Rest MD
- **Calculation**: Derived from Estimated MD and feature type percentages
- **Display**: Conditionally visible based on selected roles

### Role Selection State

- **Attributes**: Four boolean flags (SA selected, QA selected, BE selected, Support selected)
- **Constraint**: At least one flag must be true at all times
- **Default**: All four flags true on application initialization

## Out of Scope (Future Versions)

- Data persistence (saving/loading projects)
- Export to Excel/PDF
- User authentication and multi-user support
- Backend API integration
- Customizable role percentages
- Project history or version tracking
- Real-time collaboration
- Mobile-optimized responsive design (desktop-first in v1.0)

## Dependencies

- None (standalone frontend application)

## Risks and Mitigations

| Risk                                            | Impact | Likelihood | Mitigation                                                  |
| ----------------------------------------------- | ------ | ---------- | ----------------------------------------------------------- |
| Rounding errors accumulate across many features | Medium | Low        | Use integer math throughout; validate with test scenarios   |
| Performance degrades with 50+ features          | Medium | Medium     | Implement performance benchmarks; document 50-feature limit |
| Accessibility compliance gaps                   | High   | Medium     | Test with screen readers; follow WCAG 2.1 AA guidelines     |
| Role selection constraint confuses users        | Low    | Medium     | Clear inline messaging; test with real users during UAT     |

## Acceptance Testing Plan

### Test Scenario 1: Basic Calculation Flow

1. Enter project name "API Gateway Integration"
2. Add feature: "Use Case / API", costs 10,000 / 0
3. Verify Estimated MD = 2
4. Navigate to Output
5. Verify: SA=0, QA=0, BE=0, Support=1, Rest=1, Support+Rest=2

### Test Scenario 2: Multiple Features with Totals

1. Add three features with different types and varying costs
2. Navigate to Output
3. Verify each row calculates independently
4. Verify Total row sums all columns correctly

### Test Scenario 3: Role Selection Filtering

1. Uncheck QA and BE roles
2. Navigate to Output
3. Verify only SA and Support columns visible (plus Rest MD, Support+Rest MD)
4. Verify calculations unchanged despite hidden columns

### Test Scenario 4: Input Validation

1. Leave cost fields empty → Verify MD = 0
2. Enter negative cost → Verify error message and red border
3. Enter text in cost field → Verify error message
4. Correct to valid value → Verify error clears

### Test Scenario 5: Navigation State Preservation

1. Enter multiple features with data
2. Navigate to Output, review results
3. Navigate back to Input
4. Verify all entered data preserved
5. Modify a cost value
6. Navigate to Output
7. Verify updated calculations reflected

## Glossary

- **MD (Mandays)**: Unit of effort measurement representing one person working for one day
- **ROUNDDOWN**: Mathematical function that rounds a number down to the nearest integer (equivalent to Math.floor() in JavaScript)
- **SA**: System Analyst role responsible for requirements analysis and system design
- **QA**: Quality Assurance role responsible for testing and quality validation
- **BE**: Back-End Developer role responsible for server-side implementation
- **Support**: Client Support Operation role responsible for production support and maintenance
- **Rest MD**: Residual mandays after allocating to specific roles (due to rounding)
- **Integration Feature**: A distinct technical capability being integrated (API, event, batch job, etc.)
- **THB**: Thai Baht, Thailand's currency unit

---

**Document Status**: Ready for review and planning  
**Next Steps**: Create implementation plan (`/speckit.plan`) and task breakdown (`/speckit.tasks`)
