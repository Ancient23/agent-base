---
description: Develop new UI feature with visual testing against mockups
argument-hint: [route-path]
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
---

# Generate UI Test

Create visual regression tests for a new UI feature, generate baselines, and iterate with visual feedback.

## Variables

ROUTE_PATH: $ARGUMENTS

## Workflow

### 1. Setup

- If no `ROUTE_PATH` is provided, STOP immediately and ask the user to provide the route path for the new feature (e.g., `/dashboard/settings` or `/presale/buy-tokens`)
- Ask the user which app this feature is for: `devportal` or `presale`
- Set APP_NAME based on user response
- Set TEST_FILE based on APP_NAME:
  - devportal: `tests/visual/devportal.visual.spec.js`
  - presale: `tests/visual/presale.visual.spec.js`

### 2. Read Existing Test Structure

- Read the existing test file at `TEST_FILE` to understand the current test patterns
- Identify the naming convention for snapshots

### 3. Create UI Test

Add a new test suite to `TEST_FILE` for the feature with the following tests:

```javascript
test.describe('Feature Name', () => {
  test.beforeEach(async ({ page, baseURL }) => {
    await page.goto(baseURL || '/');
  });

  // Desktop view
  test('feature-name - desktop view', async ({ page }) => {
    await page.goto('ROUTE_PATH');
    await page.waitForLoadState('networkidle');
    await expect(page).toHaveScreenshot('feature-name-desktop.png', {
      fullPage: true,
      animations: 'disabled',
    });
  });

  // Mobile view
  test('feature-name - mobile view', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('ROUTE_PATH');
    await page.waitForLoadState('networkidle');
    await expect(page).toHaveScreenshot('feature-name-mobile.png', {
      fullPage: true,
      animations: 'disabled',
    });
  });

  // Tablet view
  test('feature-name - tablet view', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    await page.goto('ROUTE_PATH');
    await page.waitForLoadState('networkidle');
    await expect(page).toHaveScreenshot('feature-name-tablet.png', {
      fullPage: true,
      animations: 'disabled',
    });
  });
});
```

Replace `feature-name` with an appropriate name derived from ROUTE_PATH.

### 4. Generate Initial Baseline

Run the test to generate initial baseline snapshots:

```bash
yarn test:visual:update:APP_NAME
```

This creates baseline snapshots in `.playwright/baseline/APP_NAME/` that represent the current state of your UI.

### 5. Interactive Development

Inform the user they can now iterate on their UI with these commands:

**To see visual diffs as you code:**
```bash
yarn test:visual:APP_NAME
```

**To view the diff report:**
```bash
yarn test:visual:report
```

**To update baseline after making changes:**
```bash
yarn test:visual:update:APP_NAME
```

**To use Playwright UI mode for interactive debugging:**
```bash
yarn test:visual:ui
```

### 6. Optional: Test UI States

Ask the user if they want to add tests for specific UI states (hover, active, error, loading, etc.). If yes:

- Ask what interactive elements or states need testing
- Add additional test cases for each state
- Example patterns:
  - Hover states: `await page.hover('.selector')`
  - Active/clicked states: `await page.click('.selector')`
  - Form error states: Invalid input + submit
  - Loading states: Mock slow API responses
  - Dark mode: Toggle theme if applicable

### 7. Report

Create a summary markdown file at `/docs/features/<feature-name>-visual-testing.md`:

1. Feature name and route
2. Date created
3. List of visual test cases added
4. Viewports tested (desktop/mobile/tablet)
5. UI states covered
6. Location of baseline snapshots: `.playwright/baseline/APP_NAME/chromium/APP_NAME.visual.spec.js-snapshots/`
7. Commands to run:
   - Test: `yarn test:visual:APP_NAME`
   - Update: `yarn test:visual:update:APP_NAME`
   - Report: `yarn test:visual:report`

## Notes

- The first test run will ALWAYS fail (no baseline exists yet)
- Update baselines with `yarn test:visual:update:APP_NAME` after implementing the feature
- Visual tests run automatically in CI on PRs via GitHub Actions
- Baselines can be updated in CI by manually triggering the Playwright workflow with `update_snapshots=true`
- 10% pixel difference tolerance is configured to handle font rendering differences across platforms
