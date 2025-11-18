# Quick Start Guide: Integration Effort Estimator

**Feature**: 001-effort-estimator  
**Last Updated**: 2025-11-18  
**Prerequisites**: Node.js 18+, npm 9+, modern browser (Chrome/Firefox/Safari/Edge)

---

## 1. Initial Setup (5 minutes)

### Clone & Install

```bash
# Navigate to project root
cd /Users/mrcha/Documents/AIS/Spec-kit/spec-effort-estimator

# Checkout feature branch
git checkout 001-effort-estimator

# Install dependencies
npm install
```

**Expected Output**:

```
added 234 packages in 12s
```

**Dependencies Installed** (package.json):

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/user-event": "^14.5.0",
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "jsdom": "^23.0.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

---

## 2. Development Server (Start Coding)

### Start Dev Server

```bash
npm run dev
```

**Expected Output**:

```
  VITE v5.0.8  ready in 287 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**Browser**: Open http://localhost:5173/

**Hot Reload**: Changes to .tsx/.css files auto-refresh browser (<500ms)

---

## 3. Project Structure

```
spec-effort-estimator/
├── src/
│   ├── components/
│   │   ├── App.tsx              # Root component (project name + tables)
│   │   ├── ProjectForm.tsx      # Project name input
│   │   ├── FeatureInputTable.tsx # Feature inputs wrapper
│   │   ├── FeatureRow.tsx       # Single feature input row
│   │   └── MandayTable.tsx      # Calculated mandays display
│   ├── lib/
│   │   ├── calculations.ts      # Pure functions (calculateEstimatedMD, etc.)
│   │   └── constants.ts         # ROLE_ALLOCATIONS, FEATURE_TYPE_OPTIONS
│   ├── types.ts                 # All TypeScript interfaces
│   ├── App.css                  # Global styles
│   └── main.tsx                 # React entry point
├── tests/
│   ├── unit/
│   │   └── calculations.test.ts # Pure function tests
│   └── components/
│       └── FeatureRow.test.tsx  # Component tests
├── public/                      # Static assets (favicon, etc.)
├── package.json                 # Dependencies + scripts
├── tsconfig.json                # TypeScript config (strict mode)
├── vite.config.ts               # Vite + Vitest config
└── specs/
    └── 001-effort-estimator/
        ├── spec.md              # Requirements
        ├── plan.md              # Technical design
        ├── tasks.md             # Task list
        ├── research.md          # Technical decisions
        ├── data-model.md        # Entity definitions
        └── quickstart.md        # This file
```

---

## 4. Running Tests

### All Tests (Unit + Component)

```bash
npm test
```

**Expected Output**:

```
 ✓ tests/unit/calculations.test.ts (12 tests)
 ✓ tests/components/FeatureRow.test.tsx (8 tests)

 Test Files  2 passed (2)
      Tests  20 passed (20)
   Start at  10:23:45
   Duration  1.24s
```

### Watch Mode (TDD Workflow)

```bash
npm test -- --watch
```

**Behavior**: Tests re-run automatically when files change

### Test UI (Visual Mode)

```bash
npm test -- --ui
```

**Opens**: http://localhost:51204/**vitest**/

**Features**: Visual test runner, coverage reports, file explorer

### Coverage Report

```bash
npm test -- --coverage
```

**Expected Output**:

```
 % Coverage report from v8
--------------------|---------|----------|---------|---------|
File                | % Stmts | % Branch | % Funcs | % Lines |
--------------------|---------|----------|---------|---------|
All files           |   100   |   100    |   100   |   100   |
 lib/calculations.ts|   100   |   100    |   100   |   100   |
--------------------|---------|----------|---------|---------|
```

**HTML Report**: Open `coverage/index.html` for detailed view

---

## 5. Build for Production

### Create Optimized Build

```bash
npm run build
```

**Expected Output**:

```
vite v5.0.8 building for production...
✓ 234 modules transformed.
dist/index.html                  0.45 kB
dist/assets/index-a3b2c1d4.css   1.20 kB │ gzip: 0.58 kB
dist/assets/index-e4f5g6h7.js  142.34 kB │ gzip: 45.12 kB
✓ built in 2.18s
```

**Output Directory**: `dist/` (ready to deploy)

### Preview Production Build

```bash
npm run preview
```

**Opens**: http://localhost:4173/

**Purpose**: Test production build locally before deployment

---

## 6. Common Development Tasks

### Add a New Component

1. Create file: `src/components/MyComponent.tsx`
2. Define TypeScript interface for props
3. Export component
4. Import in parent component

**Example**:

```typescript
// src/components/MyComponent.tsx
import { FC } from "react";

interface MyComponentProps {
  value: string;
  onChange: (value: string) => void;
}

export const MyComponent: FC<MyComponentProps> = ({ value, onChange }) => {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
};
```

### Add a Pure Function

1. Add function to `src/lib/calculations.ts`
2. Export with explicit return type
3. Create test file: `tests/unit/myFunction.test.ts`
4. Write tests first (TDD)

**Example**:

```typescript
// src/lib/calculations.ts
export function myCalculation(input: number): number {
  return Math.floor(input / 100);
}

// tests/unit/myCalculation.test.ts
import { describe, it, expect } from "vitest";
import { myCalculation } from "../../src/lib/calculations";

describe("myCalculation", () => {
  it("divides input by 100 and floors result", () => {
    expect(myCalculation(250)).toBe(2);
  });
});
```

### Update Styling

Edit `src/App.css` and save → browser auto-reloads

**CSS Variables** (optional):

```css
:root {
  --primary-color: #1976d2;
  --border-color: #ddd;
}
```

### Debugging

**Browser DevTools**:

- React DevTools: Inspect component state/props
- Console: All `console.log()` statements visible
- Network: Check for CORS issues (none expected in frontend-only app)

**Vite Error Overlay**:

- TypeScript errors appear in browser as overlay
- Click error to open file in editor (if configured)

---

## 7. Key Commands Reference

| Command                  | Purpose                    | When to Use                        |
| ------------------------ | -------------------------- | ---------------------------------- |
| `npm install`            | Install dependencies       | First setup, after pulling changes |
| `npm run dev`            | Start dev server           | Daily development                  |
| `npm test`               | Run all tests              | Before committing code             |
| `npm test -- --watch`    | Run tests in watch mode    | TDD workflow                       |
| `npm test -- --coverage` | Generate coverage report   | Before PR review                   |
| `npm run build`          | Create production build    | Before deployment                  |
| `npm run preview`        | Preview production build   | QA testing                         |
| `npm run type-check`     | Check TypeScript errors    | CI/CD pipeline                     |
| `npm run lint`           | Run linter (if configured) | Code quality checks                |

---

## 8. First Feature Implementation (Example Workflow)

### Task: Implement calculateEstimatedMD()

**Step 1**: Read the task (from tasks.md)

```
T012: Implement calculateEstimatedMD() function
- Input: projectCost (number), hardwareCost (number)
- Output: number (floor((projectCost + hardwareCost) / 5000))
- Test: Division by zero (expect 0), negative costs (expect floor of result)
```

**Step 2**: Write test first

```typescript
// tests/unit/calculations.test.ts
import { describe, it, expect } from "vitest";
import { calculateEstimatedMD } from "../../src/lib/calculations";

describe("calculateEstimatedMD", () => {
  it("calculates MD from costs using floor division", () => {
    expect(calculateEstimatedMD(15000, 5000)).toBe(4); // (20000 / 5000) = 4
  });

  it("returns 0 for zero costs", () => {
    expect(calculateEstimatedMD(0, 0)).toBe(0);
  });

  it("floors fractional results", () => {
    expect(calculateEstimatedMD(7000, 6000)).toBe(2); // (13000 / 5000) = 2.6 → 2
  });
});
```

**Step 3**: Run test (should fail)

```bash
npm test -- calculations.test.ts
```

**Expected**: `Error: calculateEstimatedMD is not defined`

**Step 4**: Implement function

```typescript
// src/lib/calculations.ts
export function calculateEstimatedMD(
  projectCost: number,
  hardwareCost: number
): number {
  const total = projectCost + hardwareCost;
  return Math.floor(total / 5000);
}
```

**Step 5**: Run test again (should pass)

```bash
npm test -- calculations.test.ts
```

**Expected**: `✓ tests/unit/calculations.test.ts (3 tests) 1.02s`

**Step 6**: Use function in component

```typescript
// src/components/App.tsx
import { calculateEstimatedMD } from "../lib/calculations";

const estimatedMD = calculateEstimatedMD(
  feature.projectCost,
  feature.hardwareCost
);
```

**Step 7**: Verify in browser

1. Open http://localhost:5173/
2. Enter costs: Project = 15000, Hardware = 5000
3. Check output: Estimated MD = 4

---

## 9. Troubleshooting

### Dev Server Won't Start

**Symptom**: `Error: Cannot find module 'vite'`

**Solution**:

```bash
rm -rf node_modules package-lock.json
npm install
npm run dev
```

### TypeScript Errors Not Showing

**Check**: `tsconfig.json` has `"strict": true`

**Solution**: Restart VS Code / restart Vite server

### Tests Failing with "Cannot find module"

**Solution**:

```bash
npm install --save-dev jsdom @testing-library/react
```

### Hot Reload Not Working

**Solution**:

1. Check no syntax errors in console
2. Hard refresh browser (Cmd+Shift+R on Mac, Ctrl+Shift+R on Windows)
3. Restart dev server

### Port Already in Use

**Symptom**: `Error: Port 5173 is already in use`

**Solution**:

```bash
# Kill process on port 5173
lsof -ti:5173 | xargs kill -9

# Or use different port
npm run dev -- --port 3000
```

---

## 10. Next Steps

**Phase 1** (Setup): Complete tasks T001-T008 (project scaffolding)

**Phase 2** (Foundational): Complete tasks T009-T017 (types, constants, pure functions)

**Phase 3** (US1 MVP): Complete tasks T018-T058 (single feature input + display)

**Phase 4** (US2): Complete tasks T059-T074 (multiple features + totals)

**Phase 5** (US3): Complete tasks T075-T095 (validation + error handling)

**Phase 6** (Polish): Complete tasks T096-T116 (accessibility + documentation)

**Detailed Task List**: See `specs/001-effort-estimator/tasks.md`

---

## 11. Resources

### Documentation

- React: https://react.dev/
- TypeScript: https://www.typescriptlang.org/docs/
- Vite: https://vitejs.dev/guide/
- Vitest: https://vitest.dev/guide/
- Testing Library: https://testing-library.com/react

### Project Documents

- Requirements: `specs/001-effort-estimator/spec.md`
- Technical Plan: `specs/001-effort-estimator/plan.md`
- Task Breakdown: `specs/001-effort-estimator/tasks.md`
- Constitution: `.specify/memory/constitution.md`

### Team Contacts (Example)

- Tech Lead: [Name]
- QA Lead: [Name]
- Product Owner: [Name]

---

## 12. Development Workflow Summary

```
┌─────────────────┐
│ git checkout    │
│ 001-effort-...  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ npm install     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌──────────────────┐
│ npm run dev     │────▶│ Browser: :5173   │
└────────┬────────┘     └──────────────────┘
         │                      ▲
         ▼                      │
┌─────────────────┐             │ Hot Reload
│ Edit src/*.tsx  │─────────────┘
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ npm test --watch│◀─── Test-Driven Development
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ git commit      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ npm run build   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Deploy dist/    │
└─────────────────┘
```

---

## Quick Reference Card

```
DEV SERVER:   npm run dev        → http://localhost:5173
TEST:         npm test           → Run once
TEST WATCH:   npm test -- -w    → Auto-rerun
COVERAGE:     npm test -- --cov → Coverage report
BUILD:        npm run build     → dist/ folder
PREVIEW:      npm run preview   → Test production build

KEY FILES:
  src/components/App.tsx         → Root component
  src/lib/calculations.ts        → Pure functions
  src/types.ts                   → TypeScript interfaces
  tests/unit/calculations.test.ts → Unit tests
  specs/001-effort-estimator/    → All documentation
```

---

**Ready to start?** Run `npm run dev` and open http://localhost:5173/

**Questions?** Check `specs/001-effort-estimator/plan.md` for detailed technical design.
