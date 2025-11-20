# TypeScript Interfaces: Integration Effort Estimator

This directory contains TypeScript interface definitions for the Integration Effort Estimator application. These interfaces will be implemented in `src/types/` in the actual codebase.

## Files

- `Feature.ts` - Feature entity and FeatureType enum
- `Role.ts` - RoleSelection interface
- `MandayAllocation.ts` - MandayAllocation and TotalSummary interfaces

## Usage

These interfaces define the data contracts between components and calculation functions. All calculation functions in `src/lib/calculations.ts` accept and return values typed with these interfaces.

---

## Feature.ts

```typescript
/**
 * Enum defining the 5 types of integration features.
 * Each type has distinct role allocation percentages.
 */
export enum FeatureType {
  UseCase = "Use Case / API",
  Topic = "Topic / Event",
  DataSync = "Data Synchronization",
  Batch = "Batch / File",
  ESB = "ESB Feature",
}

/**
 * Represents a single integration feature with costs.
 * The totalMD is derived via calculation, not stored.
 */
export interface Feature {
  /** Unique identifier (UUID) for React keys */
  id: string;

  /** Type of integration feature */
  type: FeatureType;

  /** Project cost in THB (non-negative, empty treated as 0) */
  projectCost: number;

  /** Hardware cost in THB (non-negative, empty treated as 0) */
  hardwareCost: number;
}
```

---

## Role.ts

```typescript
/**
 * Represents which team roles are selected for the project.
 * At least one role must be true at all times.
 * All roles default to true on application mount.
 */
export interface RoleSelection {
  /** System Analyst (SA) role selected */
  systemAnalyst: boolean;

  /** Quality Assurance (QA) role selected */
  qualityAssurance: boolean;

  /** Back-End Developer (BE) role selected */
  backendDeveloper: boolean;

  /** Client Support Operation role selected */
  clientSupport: boolean;
}

/**
 * Role allocation percentages for a feature type.
 * Values are decimals from 0.0 to 1.0.
 */
export interface RolePercentages {
  systemAnalyst: number;
  qualityAssurance: number;
  backendDeveloper: number;
  clientSupport: number;
}
```

---

## MandayAllocation.ts

```typescript
/**
 * Represents the calculated manday distribution for a single feature.
 * Role-specific MD fields are optional - only included if role is selected.
 */
export interface MandayAllocation {
  /** Reference to Feature.id */
  featureId: string;

  /** Display name (Feature.type value) */
  featureName: string;

  /** Total mandays for the feature (Math.ceil((costs)/5000)) */
  totalMD: number;

  /** MD allocated to System Analyst (Math.floor(totalMD * 0.xx)) */
  systemAnalyst?: number;

  /** MD allocated to Quality Assurance (Math.floor(totalMD * 0.xx)) */
  qualityAssurance?: number;

  /** MD allocated to Back-End Developer (Math.floor(totalMD * 0.xx)) */
  backendDeveloper?: number;

  /** MD allocated to Client Support Operation (Math.floor(totalMD * 0.xx)) */
  clientSupport?: number;

  /** Unallocated effort (totalMD - sum of selected role MDs) */
  restMD: number;

  /** Sum of clientSupport MD and restMD */
  supportPlusRest: number;
}

/**
 * Represents aggregated manday totals across all features.
 * Displayed in the Total summary row of the output table.
 * clientSupport and restMD are not displayed (grey background cells).
 */
export interface TotalSummary {
  /** Sum of all features' System Analyst MD */
  systemAnalyst?: number;

  /** Sum of all features' Quality Assurance MD */
  qualityAssurance?: number;

  /** Sum of all features' Back-End Developer MD */
  backendDeveloper?: number;

  /** Not displayed in Total row (grey background cell) */
  clientSupport?: number;

  /** Not displayed in Total row (grey background cell) */
  restMD?: number;

  /** Sum of all features' Support+Rest MD (always displayed) */
  supportPlusRest: number;
}
```

---

## Validation Interfaces

```typescript
/**
 * Result of a validation operation.
 * If valid is false, error message is provided.
 */
export interface ValidationResult {
  valid: boolean;
  error?: string;
}

/**
 * Result of cost validation with parsed numeric value.
 */
export interface CostValidationResult extends ValidationResult {
  /** Parsed numeric value (0 if invalid) */
  parsed: number;
}
```

---

## Application State

```typescript
import { Feature } from "./Feature";
import { RoleSelection } from "./Role";

/**
 * Root application state managed by App component.
 */
export interface AppState {
  /** Current view: input form or output table */
  viewState: "input" | "output";

  /** Project name (required before viewing output) */
  projectName: string;

  /** List of integration features (0 to 20) */
  features: Feature[];

  /** Selected team roles (at least one must be true) */
  selectedRoles: RoleSelection;
}
```

---

## Calculation Function Signatures

These are the expected signatures for functions in `src/lib/calculations.ts`:

```typescript
import { Feature, FeatureType } from "../types/Feature";
import { RoleSelection, RolePercentages } from "../types/Role";
import { MandayAllocation } from "../types/MandayAllocation";

/**
 * Calculate total mandays for a feature using round-up formula.
 */
export function calculateTotalMD(
  projectCost: number,
  hardwareCost: number
): number;

/**
 * Get role allocation percentages for a feature type.
 */
export function getRolePercentages(featureType: FeatureType): RolePercentages;

/**
 * Calculate role-specific MD allocation using round-down.
 */
export function calculateRoleMD(totalMD: number, percentage: number): number;

/**
 * Calculate Rest MD (unallocated effort).
 */
export function calculateRestMD(
  totalMD: number,
  allocatedRoles: Partial<Record<string, number>>
): number;

/**
 * Calculate complete manday allocation for a feature.
 */
export function calculateFeatureAllocation(
  feature: Feature,
  selectedRoles: RoleSelection
): MandayAllocation;
```

---

## Type Guards

Utility functions for runtime type checking:

```typescript
/**
 * Type guard to check if a value is a valid FeatureType.
 */
export function isFeatureType(value: any): value is FeatureType {
  return Object.values(FeatureType).includes(value);
}

/**
 * Type guard to check if at least one role is selected.
 */
export function hasAtLeastOneRole(roles: RoleSelection): boolean {
  return Object.values(roles).some((selected) => selected);
}
```

---

## Example Usage

```typescript
import { Feature, FeatureType } from "./types/Feature";
import { RoleSelection } from "./types/Role";
import { calculateFeatureAllocation } from "./lib/calculations";

const feature: Feature = {
  id: "550e8400-e29b-41d4-a716-446655440000",
  type: FeatureType.DataSync,
  projectCost: 20000,
  hardwareCost: 5000,
};

const selectedRoles: RoleSelection = {
  systemAnalyst: true,
  qualityAssurance: true,
  backendDeveloper: false,
  clientSupport: true,
};

const allocation = calculateFeatureAllocation(feature, selectedRoles);
// allocation.totalMD === 5
// allocation.systemAnalyst === 1 (floor(5 * 0.30))
// allocation.qualityAssurance === 1 (floor(5 * 0.20))
// allocation.clientSupport === 1 (floor(5 * 0.30))
// allocation.restMD === 2 (5 - 1 - 1 - 1)
// allocation.supportPlusRest === 3 (1 + 2)
```
