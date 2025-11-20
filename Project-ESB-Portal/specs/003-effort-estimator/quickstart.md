# Quickstart: Integration Effort Estimator

**Feature Branch**: `003-effort-estimator`  
**Date**: November 20, 2025

This guide helps you set up, run, and test the Integration Effort Estimator application.

---

## Prerequisites

- **Node.js**: Version 18+ (LTS recommended)
- **npm** or **yarn**: Latest stable version
- **Git**: For cloning the repository

---

## Installation

### 1. Clone the Repository

```bash
git clone <repository-url>
cd spec-effort-estimator
```

### 2. Checkout Feature Branch

```bash
git checkout 003-effort-estimator
```

### 3. Install Dependencies

```bash
npm install
# or
yarn install
```

This installs:

- React 18+
- TypeScript 5+
- Jest (testing framework)
- React Testing Library (component tests)
- Other build tool dependencies

---

## Project Structure

```text
src/
├── lib/
│   ├── calculations.ts       # Pure calculation functions
│   └── calculations.test.ts  # Unit tests (100% coverage)
├── components/
│   ├── App.tsx               # Root component
│   ├── InputView.tsx         # Input page
│   ├── OutputView.tsx        # Output page
│   ├── FeatureRow.tsx        # Feature table row
│   └── MandayTable.tsx       # Output table
├── types/
│   ├── Feature.ts            # Feature & FeatureType
│   ├── Role.ts               # RoleSelection
│   └── MandayAllocation.ts   # MandayAllocation & TotalSummary
└── utils/
    └── validation.ts         # Input validation helpers
```

---

## Running the Application

### Development Server

Start the development server (hot-reload enabled):

```bash
npm start
# or
yarn start
```

The application opens at `http://localhost:3000` (or the port configured by your build tool).

**Note**: The exact command depends on your team's preferred React build tool. Refer to `package.json` scripts for the correct command.

### Build for Production

Create an optimized production build:

```bash
npm run build
# or
yarn build
```

Output is generated in the `build/` or `dist/` directory (tool-dependent).

---

## Running Tests

### Unit Tests (Calculation Functions)

Run unit tests for `src/lib/calculations.ts`:

```bash
npm test -- src/lib/calculations.test.ts
# or
yarn test src/lib/calculations.test.ts
```

**Expected Coverage**: 100% (constitution requirement)

### Component Tests

Run integration test for the full user flow:

```bash
npm test -- tests/integration/App.test.tsx
# or
yarn test tests/integration/App.test.tsx
```

This tests:

- Entering project name and features
- Selecting/deselecting roles
- Navigating to output view
- Verifying calculated manday values

### All Tests

Run all tests with coverage report:

```bash
npm test -- --coverage
# or
yarn test --coverage
```

**Minimum Coverage Requirements**:

- `calculations.ts`: 100%
- Overall: >80%

---

## Using the Application

### Input Page

1. **Enter Project Name** (required)

   - Text input at top of form
   - Must be non-empty to proceed to output

2. **Add Integration Features**

   - Click "Add Feature" button to add a new row
   - For each feature:
     - Select **Feature Type** from dropdown (5 options)
     - Enter **Project Cost (THB)** (non-negative number)
     - Enter **Hardware Cost (THB)** (non-negative number)
   - Click ❌ to remove a feature row

3. **Select Team Roles**

   - Check/uncheck roles available on your team:
     - System Analyst (SA)
     - Quality Assurance (QA)
     - Back-End Developer (BE)
     - Client Support Operation
   - At least one role must remain selected

4. **View Output**
   - Click "View Output" button
   - Validation errors appear if:
     - Project name is empty
     - Any feature is missing a type
     - No roles are selected

### Output Page

Displays a manday breakdown table with:

- One row per feature (showing feature type)
- Columns for selected roles (SA, QA, BE, Support)
- **Rest MD** column (unallocated effort)
- **Support + Rest MD** column (always visible)
- **Total** summary row at bottom

**Total Row Styling**:

- Client Support and Rest MD cells: grey background, no numeric value
- All other role columns: sum of all features

**Navigation**:

- Click "Back to Input" to return to input form (data preserved)

---

## Calculation Examples

### Example 1: Use Case / API Feature

**Input**:

- Feature Type: Use Case / API
- Project Cost: 15,000 THB
- Hardware Cost: 10,000 THB
- Roles: All selected

**Calculations**:

```
Total MD = ceil((15000 + 10000) / 5000) = ceil(5) = 5 MD

Role Allocations (Use Case: SA 30%, Support 70%):
  SA = floor(5 * 0.30) = 1 MD
  QA = floor(5 * 0.00) = 0 MD
  BE = floor(5 * 0.00) = 0 MD
  Support = floor(5 * 0.70) = 3 MD

Rest MD = 5 - (1+0+0+3) = 1 MD
Support + Rest MD = 3 + 1 = 4 MD
```

### Example 2: Data Synchronization Feature

**Input**:

- Feature Type: Data Synchronization
- Project Cost: 20,000 THB
- Hardware Cost: 5,000 THB
- Roles: SA and QA only

**Calculations**:

```
Total MD = ceil((20000 + 5000) / 5000) = ceil(5) = 5 MD

Role Allocations (DataSync: SA 30%, QA 20%, BE 20%, Support 30%):
  SA = floor(5 * 0.30) = 1 MD
  QA = floor(5 * 0.20) = 1 MD
  (BE not selected - omitted)
  (Support not selected - omitted)

Rest MD = 5 - (1+1) = 3 MD
Support + Rest MD = 0 + 3 = 3 MD
```

---

## Validation Rules

### Project Name

- **Required**: Cannot be empty
- **Error**: "Project name is required"

### Feature Type

- **Required**: Must be selected from dropdown
- **Error**: "Feature type must be selected"

### Costs (Project & Hardware)

- **Non-negative**: Cannot be negative numbers
- **Empty treated as 0**: No error if left blank
- **Error**: "Cost cannot be negative"

### Role Selection

- **At least one**: Cannot uncheck all roles
- **Error**: "At least one role must be selected"

---

## Troubleshooting

### Tests Failing

**Problem**: Unit tests for calculations fail  
**Solution**: Verify formula implementations:

- Total MD uses `Math.ceil((costs) / 5000)`
- Role MD uses `Math.floor(totalMD * percentage)`

**Problem**: Component tests fail  
**Solution**: Check React Testing Library queries use accessible roles/labels

### Application Errors

**Problem**: "Type is not assignable" TypeScript error  
**Solution**: Enable strict mode in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

**Problem**: Calculations return incorrect values  
**Solution**: Check input validation - ensure costs are numbers, not strings

---

## Performance Benchmarks

Expected performance (constitution requirements):

- **Calculation Speed**: <100ms for 20 features
- **UI Update**: <50ms after input change
- **Initial Load**: <2 seconds on 3G connection

To measure calculation performance, use browser DevTools Performance profiler or add timing logs:

```typescript
console.time("calculateAllAllocations");
const allocations = features.map((f) => calculateFeatureAllocation(f, roles));
console.timeEnd("calculateAllAllocations");
```

---

## Next Steps

- **Phase 2**: Run `/speckit.tasks` to generate implementation tasks
- **Implementation**: Follow task list in `tasks.md`
- **Testing**: Write unit tests before implementing calculation functions
- **Review**: Validate against constitution checklist before merge

---

## Support

For questions or issues:

1. Review the [Feature Specification](spec.md)
2. Check the [Data Model](data-model.md) for entity definitions
3. Consult the [Constitution](.specify/memory/constitution.md) for principles
4. Review [TypeScript interfaces](contracts/README.md) for API contracts
