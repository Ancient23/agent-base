# Split Plan into Small PRs

You are helping restructure an implementation plan into small, reviewable, testable PRs.

## Input
The user will provide a plan file path. Read and analyze the plan.

## Your Task
Restructure the plan to break each phase into smaller PRs following these principles:

### PR Organization Principles

1. **Backend/Shared First, UI Later**
   - PR 1: Shared functions, constants, API routes
   - PR 2: Reusable UI components (if needed)
   - PR 3: Integration into screens

2. **Each PR Must Be:**
   - **Independently testable** - Has its own test suite
   - **Small** - Focused on one concern (backend, component, or integration)
   - **Reviewable** - Clear scope, easy to review
   - **Deployable** - Can be merged without breaking existing functionality

3. **Avoid These:**
   - ❌ Separate "documentation" PRs
   - ❌ PRs without tests
   - ❌ Large PRs mixing backend + UI + integration
   - ❌ Dependencies between PRs in same phase

### Feature Doc Format (for each PR)

Each PR should have a simple **Feature Doc** section:

```markdown
**Feature Doc:** `docs/features/feature-name.md` (create or update)
- **Shared/Backend:** List new shared functions, utilities, or backend changes
- **Constants:** New constants added (if applicable)
- **API Routes:** New or updated API endpoints
- **Screens/Components:** UI components or screens modified
- **User Actions:** How to manually test this PR (clear steps)
- **Testing:** Quick summary of automated tests
```

### Feature Doc Content

Feature docs should be **concise and practical**:
- **Focus:** What code changed and how to test it
- **Format:** Bullet points listing files and changes
- **Testing:** Clear user actions to verify the feature works
- **Avoid:** Screenshots, lengthy explanations, marketing language

### Example PR Structure

**Phase X: [Feature Name]**

#### Phase X.1: Add Backend API and Shared Functions
**PR Size:** Small | **Estimated Effort:** Low

**Tests (Write First):**
- Create test file: `path/to/test.ts`
  - Test case 1
  - Test case 2

**Implementation:**
- Add function: `shared/src/lib/module.ts` - code example
- Add constant: `shared/src/constants/module.ts` - code example
- Create API route: `app/api/route/route.ts` - code example

**Feature Doc:** `docs/features/feature-name.md`
- **Shared:** `shared/src/lib/module.ts` - New function description
- **API Routes:** `app/api/route/route.ts` - What it does
- **Testing:** How to call the API and expected response

**Validation:**
- Run: build, lint, typecheck and tests
- Test: Specific manual test step

**Success Criteria:**
- All tests pass
- API route works as expected

---

#### Phase X.2: Create UI Component
**PR Size:** Small | **Estimated Effort:** Low

**Tests (Write First):**
- Create test file: `path/to/component.test.tsx`

**Implementation:**
- Create component: `app/components/Component.tsx`

**Feature Doc:** `docs/features/feature-name.md` (update)
- **Components:** `app/components/Component.tsx` - What it renders
- **Testing:** Component renders with expected props

**Validation:**
- Run: build, lint, typecheck and tests
- Test: Component renders correctly

**Success Criteria:**
- All tests pass
- Component is reusable

---

#### Phase X.3: Integrate Component into Screen
**PR Size:** Tiny | **Estimated Effort:** Very Low

**Implementation:**
- Update: `app/screens/Screen.tsx` - Add component

**Feature Doc:** `docs/features/feature-name.md` (update)
- **Screens:** `app/screens/Screen.tsx` - Component added
- **User Actions:**
  - Navigate to screen
  - Perform action X
  - See result Y

**Validation:**
- Run build, lint, typecheck and tests
- Test: Full end-to-end flow

**Success Criteria:**
- Feature works end-to-end
- UI properly integrated

## Output

Restructure the plan with:
1. Clear PR boundaries (X.1, X.2, X.3, etc.)
2. Simple feature doc sections for each PR
3. Test-first approach
4. Backend → Component → Integration order
5. Independent, small, testable PRs

Keep existing code examples and implementation details but organize them into smaller PRs.
