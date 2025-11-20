# Data Model: Integration Effort Estimator

**Feature Branch**: `003-effort-estimator`  
**Date**: November 20, 2025

## Overview

This document defines the data entities and their relationships for the Integration Effort Estimator application. All entities are client-side TypeScript interfaces with no backend persistence in v1.0.

## Core Entities

### Feature

Represents a single integration feature with costs and calculated manday estimate.

**Attributes**:

- `id: string` - Unique identifier (UUID) for React key prop and state management
- `type: FeatureType` - Type of integration feature (enum)
- `projectCost: number` - Project cost in THB (non-negative, default 0)
- `hardwareCost: number` - Hardware cost in THB (non-negative, default 0)

**Validation Rules**:

- `type` is required (must be selected from dropdown)
- `projectCost` and `hardwareCost` must be non-negative
- Empty cost fields are treated as 0 in calculations

**Derived Values** (computed, not stored):

- `totalMD: number` - Calculated as `Math.ceil((projectCost + hardwareCost) / 5000)`

**Example**:

```typescript
{
  id: '550e8400-e29b-41d4-a716-446655440000',
  type: FeatureType.DataSync,
  projectCost: 20000,
  hardwareCost: 5000
  // totalMD computed: Math.ceil(25000 / 5000) = 5
}
```

---

### FeatureType (Enum)

Defines the 5 types of integration features with distinct role allocation rules.

**Values**:

- `UseCase = 'Use Case / API'` - API integration use cases
- `Topic = 'Topic / Event'` - Event-driven topic integrations
- `DataSync = 'Data Synchronization'` - Data synchronization tasks
- `Batch = 'Batch / File'` - Batch processing and file transfers
- `ESB = 'ESB Feature'` - Enterprise Service Bus features

**Role Allocation Mapping** (percentages):

```typescript
{
  UseCase:  { SA: 0.30, QA: 0.00, BE: 0.00, Support: 0.70 },
  Topic:    { SA: 0.30, QA: 0.00, BE: 0.00, Support: 0.70 },
  DataSync: { SA: 0.30, QA: 0.20, BE: 0.20, Support: 0.30 },
  Batch:    { SA: 0.30, QA: 0.20, BE: 0.20, Support: 0.30 },
  ESB:      { SA: 0.20, QA: 0.30, BE: 0.20, Support: 0.30 }
}
```

---

### RoleSelection

Represents which team roles are selected for the project (affects column visibility in output).

**Attributes**:

- `systemAnalyst: boolean` - System Analyst (SA) role selected
- `qualityAssurance: boolean` - Quality Assurance (QA) role selected
- `backendDeveloper: boolean` - Back-End Developer (BE) role selected
- `clientSupport: boolean` - Client Support Operation role selected

**Validation Rules**:

- At least one role must be `true` at all times
- Attempting to uncheck the last selected role is prevented with validation error

**Default State**:
All roles set to `true` on application mount.

**Example**:

```typescript
{
  systemAnalyst: true,
  qualityAssurance: false,    // QA not available on team
  backendDeveloper: false,    // BE not available on team
  clientSupport: true
}
```

---

### MandayAllocation

Represents the calculated manday distribution for a single feature across selected roles.

**Attributes**:

- `featureId: string` - Reference to Feature.id
- `featureName: string` - Display name (Feature.type value)
- `totalMD: number` - Total mandays for the feature (from `calculateTotalMD`)
- `systemAnalyst?: number` - MD allocated to SA (optional, only if role selected)
- `qualityAssurance?: number` - MD allocated to QA (optional, only if role selected)
- `backendDeveloper?: number` - MD allocated to BE (optional, only if role selected)
- `clientSupport?: number` - MD allocated to Support (optional, only if role selected)
- `restMD: number` - Unallocated effort (always calculated)
- `supportPlusRest: number` - Sum of Support MD and Rest MD (always calculated)

**Calculation Rules**:

- Role MD values: `Math.floor(totalMD * rolePercentage)` for each selected role
- `restMD = totalMD - (sum of all selected role MD values)`
- `supportPlusRest = (clientSupport || 0) + restMD`
- Optional role fields are omitted (not set to 0) if role is not selected

**Example** (Data Synchronization with all roles selected):

```typescript
{
  featureId: '550e8400-e29b-41d4-a716-446655440000',
  featureName: 'Data Synchronization',
  totalMD: 10,
  systemAnalyst: 3,      // floor(10 * 0.30) = 3
  qualityAssurance: 2,   // floor(10 * 0.20) = 2
  backendDeveloper: 2,   // floor(10 * 0.20) = 2
  clientSupport: 3,      // floor(10 * 0.30) = 3
  restMD: 0,             // 10 - (3+2+2+3) = 0
  supportPlusRest: 3     // 3 + 0 = 3
}
```

**Example** (Same feature with only SA and QA selected):

```typescript
{
  featureId: '550e8400-e29b-41d4-a716-446655440000',
  featureName: 'Data Synchronization',
  totalMD: 10,
  systemAnalyst: 3,      // floor(10 * 0.30) = 3
  qualityAssurance: 2,   // floor(10 * 0.20) = 2
  // backendDeveloper omitted (role not selected)
  // clientSupport omitted (role not selected)
  restMD: 5,             // 10 - (3+2) = 5
  supportPlusRest: 5     // 0 + 5 = 5
}
```

---

### TotalSummary

Represents aggregated manday totals across all features (displayed in output table's Total row).

**Attributes**:

- `systemAnalyst?: number` - Sum of all features' SA MD (optional, only if role selected)
- `qualityAssurance?: number` - Sum of all features' QA MD (optional, only if role selected)
- `backendDeveloper?: number` - Sum of all features' BE MD (optional, only if role selected)
- `clientSupport?: number` - Not displayed in Total row (cell has grey background, no value)
- `restMD?: number` - Not displayed in Total row (cell has grey background, no value)
- `supportPlusRest: number` - Sum of all features' Support+Rest MD (always displayed)

**Display Rules**:

- Only summed role columns show numeric values
- `clientSupport` and `restMD` cells have grey background (`background-color: #e0e0e0`) with no text
- `supportPlusRest` always displays sum

**Example**:

```typescript
{
  systemAnalyst: 12,        // Sum across all features
  qualityAssurance: 8,      // Sum across all features
  // backendDeveloper omitted (role not selected)
  clientSupport: undefined, // Not displayed in Total row
  restMD: undefined,        // Not displayed in Total row
  supportPlusRest: 25       // Sum across all features
}
```

---

## Application State

The root `App` component manages all application state using React `useState`.

**State Shape**:

```typescript
interface AppState {
  viewState: "input" | "output";
  projectName: string;
  features: Feature[];
  selectedRoles: RoleSelection;
}
```

**Default Initial State**:

```typescript
{
  viewState: 'input',
  projectName: '',
  features: [],
  selectedRoles: {
    systemAnalyst: true,
    qualityAssurance: true,
    backendDeveloper: true,
    clientSupport: true
  }
}
```

**State Transitions**:

- `viewState`: Toggles between `'input'` and `'output'` when user clicks "View Output" or "Back to Input"
- `projectName`: Updates on text input change (controlled component)
- `features`: Updates on add/remove/edit feature actions (immutable array operations)
- `selectedRoles`: Updates on checkbox toggle (immutable object update, validated to keep at least one `true`)

---

## Data Flow

```text
User Input (InputView)
  ↓
App State Updates (setState)
  ↓
Calculation Functions (pure, src/lib/calculations.ts)
  ↓
MandayAllocation Objects (computed)
  ↓
OutputView Rendering (MandayTable)
```

**Key Points**:

- State is the single source of truth (no derived state stored)
- Calculations are performed on-demand when rendering OutputView
- All calculations use pure functions (no side effects, no mutations)
- View state controls which component tree renders (InputView vs OutputView)

---

## Entity Relationships

```text
App (1)
├── projectName: string
├── viewState: 'input' | 'output'
├── features: Feature[] (0..20)
│   └── Feature (1)
│       ├── id: string
│       ├── type: FeatureType (enum)
│       ├── projectCost: number
│       └── hardwareCost: number
└── selectedRoles: RoleSelection (1)
    ├── systemAnalyst: boolean
    ├── qualityAssurance: boolean
    ├── backendDeveloper: boolean
    └── clientSupport: boolean

Derived at Render Time:
MandayAllocation[] (computed from Feature[] + RoleSelection)
TotalSummary (computed from MandayAllocation[])
```

---

## Persistence

**v1.0**: No persistence (frontend-only, all state in memory)

- State is lost on page refresh
- No browser storage (localStorage/sessionStorage) in initial version
- Users must complete estimation in single session

**Future Versions**:

- localStorage for draft projects (auto-save)
- Backend API for saving/loading projects
- Export to PDF/Excel for sharing

---

## Type Safety

All entities are defined as TypeScript interfaces in `src/types/`:

- `Feature.ts` - Feature interface and FeatureType enum
- `Role.ts` - RoleSelection interface
- `MandayAllocation.ts` - MandayAllocation and TotalSummary interfaces

Calculation functions in `src/lib/calculations.ts` use these types for parameters and return values, ensuring compile-time type checking.
