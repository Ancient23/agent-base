---
description: Build UI feature with interactive visual testing via Playwright MCP
argument-hint: [feature-description or route-path]
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion, Glob, Grep, mcp__playwright__*
---

# UI Build (Interactive Visual Development)

Build a UI feature with real-time visual feedback using Playwright MCP to see and test your implementation interactively.

## Prerequisites

**IMPORTANT:** This command requires Playwright MCP server to be enabled. If MCP tools are not available:

1. Exit Claude
2. Relaunch with one of the following configs:

**Headless mode (Recommended - Faster):**
```bash
claude --mcp-config .mcp.playwright.headless.json
```
- Browser runs in background
- Faster execution
- Claude sees screenshots but you don't see the browser window
- Best for: Quick iterations, automated testing

**Headed mode (Watch browser in real-time):**
```bash
claude --mcp-config .mcp.playwright.headed.json
```
- Browser window opens and you can watch it
- Slower execution
- Claude still only sees screenshots (not the live window)
- Best for: Debugging complex interactions, watching animations

3. Run this command again

To verify MCP is available, check if `mcp__playwright__*` tools are accessible.

## Variables

FEATURE_INPUT: $ARGUMENTS

## Workflow

### 1. Gather Requirements

- If no `FEATURE_INPUT` is provided, ask the user for:
  - Description of the UI feature to build
  - Which route/path it should be accessible at
- Ask the user which app: `devportal` or `presale`
- Set APP_NAME based on response
- Ask if they have mockup images:
  - If yes, ask for file paths to mockup images (PNG, JPG, etc.)
  - If no, work from text description only
- Set DEV_PORT:
  - devportal: 3001
  - presale: 3002

### 2. Read Mockups (if provided)

If mockup file paths are provided:
- Use Read tool to load each mockup image
- These will be visual references throughout development

### 3. Analyze Existing Codebase

Before implementing, understand the project structure:
- Read AGENTS.md
- Find similar features/components in the codebase using Glob/Grep
- Identify the component architecture (React/Next.js patterns)
- Locate styling approach (CSS modules, Tailwind, styled-components, etc.)
- Find routing structure

### 4. Implement UI Feature

Build the feature following the project's patterns:

**Components:**
- Create necessary React components in appropriate directories
- Follow existing naming conventions
- Implement responsive design (mobile, tablet, desktop)

**Styling:**
- Match the styling approach used in the codebase
- If mockups provided, implement styles to match them
- Ensure dark mode support if the app uses it

**Routing:**
- Add route configuration if needed
- Update navigation/menu if necessary

**States to implement:**
- Default/initial state
- Loading states
- Error states
- Success states
- Interactive states (hover, focus, active)
- Empty states (if applicable)

### 5. Start Development Server

Ensure the dev server is running:

```bash
cd APP_NAME && yarn dev
```

Wait for server to be ready on `http://localhost:DEV_PORT`

### 6. Interactive Visual Testing with MCP

Use Playwright MCP tools to test the implementation:

#### 6.1 Navigate and Capture Initial State

- Navigate to `http://localhost:DEV_PORT/ROUTE_PATH`
- Wait for page load
- Take screenshot of initial state

#### 6.2 Test Responsive Views

For each viewport (desktop, tablet, mobile):
- Set viewport size:
  - Desktop: 1920x1080
  - Tablet: 768x1024
  - Mobile: 375x667
- Navigate to feature
- Take screenshot
- Compare to mockup (if provided)

#### 6.3 Test Interactive States

For each interactive element:
- Hover states: Hover over buttons, links, cards
- Active states: Click and capture active state
- Focus states: Tab through form elements
- Take screenshots of each state

#### 6.4 Test Form Interactions (if applicable)

If the feature includes forms:
- Fill form with valid data → screenshot success state
- Fill form with invalid data → screenshot error state
- Test validation messages → screenshot each error
- Test submit success → screenshot confirmation

#### 6.5 Test Dynamic States

- Loading states: Capture loading spinners/skeletons
- Empty states: Test with no data
- Error states: Test error handling
- Success states: Test successful operations

### 7. Compare and Iterate

After each screenshot:
- **If mockups provided:** Compare screenshot to mockup visually
  - Identify differences in layout, spacing, colors, fonts
  - List specific changes needed
- **If no mockups:** Evaluate against description and best practices
  - Check visual hierarchy
  - Verify responsive behavior
  - Test usability

If differences found:
- Make code changes to fix issues
- Repeat visual testing for changed areas
- Continue until satisfied

### 8. Final Verification

Once implementation looks good:
- Test complete user flow end-to-end via MCP
- Verify all interactive elements work
- Check all viewports (mobile, tablet, desktop)
- Test edge cases (long text, empty data, errors)
- Take final screenshots of all states

### 9. Report

Create a summary of the implementation:

1. **Feature implemented:** Name and route
2. **Files created/modified:** List with brief description
3. **UI states implemented:** List all states (default, loading, error, success, etc.)
4. **Viewports tested:** Desktop, tablet, mobile
5. **Interactive elements tested:** Buttons, forms, navigation, etc.
6. **Screenshots captured:** Reference key screenshots from MCP testing
7. **Mockup comparison:** If mockups were provided, note how well implementation matches
8. **Remaining work:** Any items that need follow-up
9. **Next steps:**
   - Suggest running `/generate-ui-test` to create permanent regression tests
   - Suggest running `yarn build && yarn test` before committing

## Notes

- MCP Playwright runs in isolated mode for clean browser contexts
- Default viewport: 1920x1080 (desktop-first), but Claude changes it programmatically for mobile/tablet testing
- Screenshots are visible to Claude for visual comparison
- This command focuses on development iteration, not permanent test creation
- For permanent visual regression tests, use `/generate-ui-test` after completing this command
- The dev server must be running for MCP to access the UI
- Choose headless mode for speed, headed mode for debugging
- Both configs use the same viewport settings and isolation mode
