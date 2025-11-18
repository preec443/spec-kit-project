# API Contracts: Integration Effort Estimator

**Feature**: 001-effort-estimator  
**Date**: 2025-11-18  
**Version**: v1.0 (Frontend-Only)

---

## Overview

This is a **frontend-only application** with **no backend API**. However, this document defines "contracts" as **function signatures** and **data interfaces** that act as internal APIs between components and calculation logic.

**Purpose**:

- Document expected inputs/outputs for all public functions
- Define component props interfaces (contracts between React components)
- Establish type safety boundaries
- Provide reference for testing and future refactoring

**Note**: If v2.0 introduces a backend, this directory would contain OpenAPI/GraphQL schemas.

---

## 1. Calculation Functions API

### calculateEstimatedMD

**Purpose**: Calculate total estimated mandays from project and hardware costs.

**Signature**:

```typescript
function calculateEstimatedMD(
  projectCost: number,
  hardwareCost: number
): number;
```

**Contract**:

| Parameter      | Type     | Required | Constraints  | Description                                                    |
| -------------- | -------- | -------- | ------------ | -------------------------------------------------------------- |
| `projectCost`  | `number` | Yes      | ≥ 0          | Project-related expenses in dollars                            |
| `hardwareCost` | `number` | Yes      | ≥ 0          | Hardware expenses in dollars                                   |
| **Returns**    | `number` | -        | ≥ 0, integer | Floored division: `floor((projectCost + hardwareCost) / 5000)` |

**Examples**:

```typescript
calculateEstimatedMD(15000, 5000); // Returns: 4
calculateEstimatedMD(0, 0); // Returns: 0
calculateEstimatedMD(7000, 6000); // Returns: 2 (13000/5000 = 2.6 → floor → 2)
calculateEstimatedMD(2499, 2499); // Returns: 0 (4998/5000 = 0.996 → floor → 0)
```

**Error Handling**: None (pure function assumes valid numeric input)

**Location**: `src/lib/calculations.ts`

---

### calculateRoleAllocations

**Purpose**: Calculate manday breakdown across 4 roles based on feature type.

**Signature**:

```typescript
function calculateRoleAllocations(
  featureType: FeatureType,
  estimatedMD: number
): RoleAllocation;
```

**Contract**:

| Parameter     | Type             | Required | Constraints              | Description                         |
| ------------- | ---------------- | -------- | ------------------------ | ----------------------------------- |
| `featureType` | `FeatureType`    | Yes      | One of 5 enum values     | Determines role percentages         |
| `estimatedMD` | `number`         | Yes      | ≥ 0, integer             | Total mandays to allocate           |
| **Returns**   | `RoleAllocation` | -        | All fields ≥ 0, integers | Object with sa/qa/be/support fields |

**Type Definitions**:

```typescript
type FeatureType =
  | "publish"
  | "consume"
  | "bi-directional"
  | "publish-external"
  | "consume-external";

interface RoleAllocation {
  sa: number; // Solution Architect mandays
  qa: number; // Quality Assurance mandays
  be: number; // Backend Engineer mandays
  support: number; // Support mandays
}
```

**Examples**:

```typescript
calculateRoleAllocations("publish", 10);
// Returns: { sa: 0, qa: 1, be: 6, support: 2 }
// Logic: sa = floor(10 × 5/100) = 0, qa = floor(10 × 15/100) = 1, etc.

calculateRoleAllocations("bi-directional", 20);
// Returns: { sa: 2, qa: 4, be: 10, support: 4 }

calculateRoleAllocations("publish-external", 7);
// Returns: { sa: 1, qa: 1, be: 3, support: 1 }
```

**Error Handling**: None (pure function assumes valid enum input)

**Location**: `src/lib/calculations.ts`

---

### calculateTotals

**Purpose**: Aggregate all calculations into total mandays and role allocations.

**Signature**:

```typescript
function calculateTotals(calculations: Calculation[]): TotalCalculation;
```

**Contract**:

| Parameter      | Type               | Required | Constraints          | Description                      |
| -------------- | ------------------ | -------- | -------------------- | -------------------------------- |
| `calculations` | `Calculation[]`    | Yes      | Array (may be empty) | All feature calculations         |
| **Returns**    | `TotalCalculation` | -        | All fields ≥ 0       | Sum of estimatedMD and each role |

**Type Definitions**:

```typescript
interface Calculation {
  featureId: string;
  estimatedMD: number;
  roleAllocations: RoleAllocation;
}

interface TotalCalculation {
  totalEstimatedMD: number;
  totalRoleAllocations: RoleAllocation;
}
```

**Examples**:

```typescript
const calculations = [
  {
    featureId: "1",
    estimatedMD: 4,
    roleAllocations: { sa: 0, qa: 0, be: 2, support: 0 },
  },
  {
    featureId: "2",
    estimatedMD: 8,
    roleAllocations: { sa: 0, qa: 1, be: 4, support: 1 },
  },
];

calculateTotals(calculations);
// Returns: {
//   totalEstimatedMD: 12,
//   totalRoleAllocations: { sa: 0, qa: 1, be: 6, support: 1 }
// }

calculateTotals([]);
// Returns: {
//   totalEstimatedMD: 0,
//   totalRoleAllocations: { sa: 0, qa: 0, be: 0, support: 0 }
// }
```

**Error Handling**: None (reduces over array, handles empty case)

**Location**: `src/lib/calculations.ts`

---

### validateCostInput

**Purpose**: Validate user input for cost fields (projectCost, hardwareCost).

**Signature**:

```typescript
function validateCostInput(value: string): ValidationResult;
```

**Contract**:

| Parameter   | Type               | Required | Constraints           | Description                                     |
| ----------- | ------------------ | -------- | --------------------- | ----------------------------------------------- |
| `value`     | `string`           | Yes      | Any string            | Raw input from `<input>` element                |
| **Returns** | `ValidationResult` | -        | Always returns object | Validation result with error message if invalid |

**Type Definition**:

```typescript
interface ValidationResult {
  valid: boolean;
  error?: string;
  numericValue: number;
}
```

**Examples**:

```typescript
validateCostInput("15000");
// Returns: { valid: true, numericValue: 15000 }

validateCostInput("");
// Returns: { valid: true, numericValue: 0 }

validateCostInput("abc");
// Returns: { valid: false, error: 'Must be a valid number', numericValue: 0 }

validateCostInput("-500");
// Returns: { valid: false, error: 'Cost cannot be negative', numericValue: 0 }

validateCostInput("3.14");
// Returns: { valid: true, numericValue: 3.14 }
```

**Error Messages**:

- `"Must be a valid number"` - Input is not numeric (NaN)
- `"Cost cannot be negative"` - Input is negative number

**Location**: `src/lib/calculations.ts`

---

## 2. Component Props Interfaces (React Component Contracts)

### App Component

**Purpose**: Root component managing project state.

**Props**: None (root component)

**State**:

```typescript
interface AppState {
  projectName: string;
  features: Feature[];
}
```

**Location**: `src/components/App.tsx`

---

### ProjectForm Component

**Purpose**: Display and edit project name.

**Props Contract**:

```typescript
interface ProjectFormProps {
  projectName: string;
  onProjectNameChange: (name: string) => void;
}
```

**Contract**:

| Prop                  | Type                     | Required | Description                             |
| --------------------- | ------------------------ | -------- | --------------------------------------- |
| `projectName`         | `string`                 | Yes      | Current project name (controlled input) |
| `onProjectNameChange` | `(name: string) => void` | Yes      | Callback when user types                |

**Example Usage**:

```typescript
<ProjectForm
  projectName="My Integration"
  onProjectNameChange={(name) => setProjectName(name)}
/>
```

**Location**: `src/components/ProjectForm.tsx`

---

### FeatureInputTable Component

**Purpose**: Wrapper for feature input rows + add button.

**Props Contract**:

```typescript
interface FeatureInputTableProps {
  features: Feature[];
  onFeatureUpdate: (id: string, field: keyof Feature, value: any) => void;
  onFeatureDelete: (id: string) => void;
  onFeatureAdd: () => void;
}
```

**Contract**:

| Prop              | Type                                                     | Required | Description                          |
| ----------------- | -------------------------------------------------------- | -------- | ------------------------------------ |
| `features`        | `Feature[]`                                              | Yes      | Array of all features (may be empty) |
| `onFeatureUpdate` | `(id: string, field: keyof Feature, value: any) => void` | Yes      | Update single field of a feature     |
| `onFeatureDelete` | `(id: string) => void`                                   | Yes      | Remove feature by id                 |
| `onFeatureAdd`    | `() => void`                                             | Yes      | Add new feature with defaults        |

**Type Definition**:

```typescript
interface Feature {
  id: string;
  type: FeatureType;
  projectCost: number;
  hardwareCost: number;
}
```

**Example Usage**:

```typescript
<FeatureInputTable
  features={features}
  onFeatureUpdate={(id, field, value) => {
    setFeatures(
      features.map((f) => (f.id === id ? { ...f, [field]: value } : f))
    );
  }}
  onFeatureDelete={(id) => setFeatures(features.filter((f) => f.id !== id))}
  onFeatureAdd={() =>
    setFeatures([
      ...features,
      {
        id: crypto.randomUUID(),
        type: "publish",
        projectCost: 0,
        hardwareCost: 0,
      },
    ])
  }
/>
```

**Location**: `src/components/FeatureInputTable.tsx`

---

### FeatureRow Component

**Purpose**: Single row for feature input (type + costs + delete).

**Props Contract**:

```typescript
interface FeatureRowProps {
  feature: Feature;
  onUpdate: (field: keyof Feature, value: any) => void;
  onDelete: () => void;
}
```

**Contract**:

| Prop       | Type                                         | Required | Description                                            |
| ---------- | -------------------------------------------- | -------- | ------------------------------------------------------ |
| `feature`  | `Feature`                                    | Yes      | Feature data for this row                              |
| `onUpdate` | `(field: keyof Feature, value: any) => void` | Yes      | Update single field (no id needed, scoped to this row) |
| `onDelete` | `() => void`                                 | Yes      | Delete this feature (no id needed, scoped to this row) |

**Example Usage**:

```typescript
<FeatureRow
  feature={feature}
  onUpdate={(field, value) => onFeatureUpdate(feature.id, field, value)}
  onDelete={() => onFeatureDelete(feature.id)}
/>
```

**Internal State**:

```typescript
interface FeatureRowState {
  projectCostInput: string; // Raw input (may be invalid)
  hardwareCostInput: string; // Raw input (may be invalid)
  projectCostError?: string; // Error message if invalid
  hardwareCostError?: string; // Error message if invalid
}
```

**Location**: `src/components/FeatureRow.tsx`

---

### MandayTable Component

**Purpose**: Display calculated mandays for all features + totals.

**Props Contract**:

```typescript
interface MandayTableProps {
  features: Feature[];
  calculations: Calculation[];
  totals: TotalCalculation;
}
```

**Contract**:

| Prop           | Type               | Required | Description                                 |
| -------------- | ------------------ | -------- | ------------------------------------------- |
| `features`     | `Feature[]`        | Yes      | Array of features (for feature type labels) |
| `calculations` | `Calculation[]`    | Yes      | Calculated mandays for each feature         |
| `totals`       | `TotalCalculation` | Yes      | Aggregated totals row                       |

**Type Definitions**:

```typescript
interface Calculation {
  featureId: string;
  estimatedMD: number;
  roleAllocations: RoleAllocation;
}

interface TotalCalculation {
  totalEstimatedMD: number;
  totalRoleAllocations: RoleAllocation;
}
```

**Example Usage**:

```typescript
const calculations = features.map((f) => ({
  featureId: f.id,
  estimatedMD: calculateEstimatedMD(f.projectCost, f.hardwareCost),
  roleAllocations: calculateRoleAllocations(f.type, estimatedMD),
}));

const totals = calculateTotals(calculations);

<MandayTable features={features} calculations={calculations} totals={totals} />;
```

**Location**: `src/components/MandayTable.tsx`

---

## 3. Constants API

### ROLE_ALLOCATIONS

**Purpose**: Lookup table for role percentages by feature type.

**Type**:

```typescript
const ROLE_ALLOCATIONS: Record<FeatureType, RoleAllocation>;
```

**Contract**:

| Feature Type       | SA % | QA % | BE % | Support % |
| ------------------ | ---- | ---- | ---- | --------- |
| `publish`          | 5    | 15   | 60   | 20        |
| `consume`          | 5    | 15   | 60   | 20        |
| `bi-directional`   | 10   | 20   | 50   | 20        |
| `publish-external` | 15   | 20   | 45   | 20        |
| `consume-external` | 15   | 20   | 45   | 20        |

**Example Usage**:

```typescript
import { ROLE_ALLOCATIONS } from "./constants";

const percentages = ROLE_ALLOCATIONS["bi-directional"];
// Returns: { sa: 10, qa: 20, be: 50, support: 20 }
```

**Location**: `src/lib/constants.ts`

---

### FEATURE_TYPE_OPTIONS

**Purpose**: Dropdown options for feature type selector.

**Type**:

```typescript
const FEATURE_TYPE_OPTIONS: Array<{ value: FeatureType; label: string }>;
```

**Contract**:

```typescript
[
  { value: "publish", label: "Publish" },
  { value: "consume", label: "Consume" },
  { value: "bi-directional", label: "Bi-directional" },
  { value: "publish-external", label: "Publish External" },
  { value: "consume-external", label: "Consume External" },
];
```

**Example Usage**:

```typescript
import { FEATURE_TYPE_OPTIONS } from "./constants";

<select value={feature.type} onChange={(e) => onUpdate("type", e.target.value)}>
  {FEATURE_TYPE_OPTIONS.map((opt) => (
    <option key={opt.value} value={opt.value}>
      {opt.label}
    </option>
  ))}
</select>;
```

**Location**: `src/lib/constants.ts`

---

## 4. Future API Contracts (v2.0+)

### REST API Endpoints (if backend added)

**Base URL**: `/api/v1`

#### POST /projects

Create new project with features.

**Request Body**:

```json
{
  "name": "My Integration Project",
  "features": [
    {
      "type": "publish",
      "projectCost": 15000,
      "hardwareCost": 5000
    }
  ]
}
```

**Response** (201 Created):

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "My Integration Project",
  "features": [...],
  "calculations": [...],
  "totals": {...},
  "createdAt": "2025-11-18T10:30:00Z"
}
```

#### GET /projects/:id

Retrieve saved project.

**Response** (200 OK):

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "My Integration Project",
  "features": [...],
  "createdAt": "2025-11-18T10:30:00Z",
  "updatedAt": "2025-11-18T10:35:00Z"
}
```

#### PUT /projects/:id

Update existing project.

#### DELETE /projects/:id

Delete project.

---

### GraphQL Schema (alternative to REST)

```graphql
type Project {
  id: ID!
  name: String!
  features: [Feature!]!
  calculations: [Calculation!]!
  totals: TotalCalculation!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Feature {
  id: ID!
  type: FeatureType!
  projectCost: Float!
  hardwareCost: Float!
}

enum FeatureType {
  PUBLISH
  CONSUME
  BI_DIRECTIONAL
  PUBLISH_EXTERNAL
  CONSUME_EXTERNAL
}

type Calculation {
  featureId: ID!
  estimatedMD: Int!
  roleAllocations: RoleAllocation!
}

type RoleAllocation {
  sa: Int!
  qa: Int!
  be: Int!
  support: Int!
}

type TotalCalculation {
  totalEstimatedMD: Int!
  totalRoleAllocations: RoleAllocation!
}

type Query {
  project(id: ID!): Project
  projects: [Project!]!
}

type Mutation {
  createProject(input: CreateProjectInput!): Project!
  updateProject(id: ID!, input: UpdateProjectInput!): Project!
  deleteProject(id: ID!): Boolean!
}

input CreateProjectInput {
  name: String!
  features: [FeatureInput!]!
}

input FeatureInput {
  type: FeatureType!
  projectCost: Float!
  hardwareCost: Float!
}
```

---

## 5. Testing Contracts

### Test Coverage Requirements

All public functions must have:

- **Unit tests**: 100% branch coverage
- **Edge case tests**: 0, negative, large numbers, empty strings
- **Integration tests**: Component + calculation function interaction

**Test Suites**:

```typescript
// tests/unit/calculations.test.ts
describe('calculateEstimatedMD', () => {
  it('handles zero costs', () => { ... });
  it('floors fractional results', () => { ... });
  it('handles large numbers', () => { ... });
});

// tests/components/FeatureRow.test.tsx
describe('FeatureRow', () => {
  it('displays validation errors on invalid input', () => { ... });
  it('calls onUpdate with numeric value on valid blur', () => { ... });
  it('allows deletion', () => { ... });
});
```

**Location**: `tests/` directory

---

## 6. Contract Versioning

**v1.0** (Current):

- Frontend-only function contracts
- No backend API
- Session-based state (no persistence)

**v2.0** (Future):

- Add REST API endpoints
- Persistent storage (database)
- User authentication

**v3.0** (Future):

- Export to Excel/CSV
- Historical project comparison
- Multi-currency support

**Breaking Changes Policy**:

- Function signatures must maintain backward compatibility
- Deprecation warnings for 2 releases before removal
- Semantic versioning for API changes

---

## Summary

**Total Contracts**:

- **5 Pure Functions**: calculateEstimatedMD, calculateRoleAllocations, calculateTotals, validateCostInput, (+ utility functions)
- **5 Component Props**: App, ProjectForm, FeatureInputTable, FeatureRow, MandayTable
- **2 Constants**: ROLE_ALLOCATIONS, FEATURE_TYPE_OPTIONS
- **6 TypeScript Interfaces**: Feature, FeatureType, RoleAllocation, Calculation, TotalCalculation, ValidationResult

**Type Safety**: All contracts enforced by TypeScript strict mode

**Testing**: All contracts covered by unit + component tests

**Documentation**: This file serves as single source of truth for internal APIs

---

**Next Steps**: Begin implementation (Task T001 in tasks.md)
