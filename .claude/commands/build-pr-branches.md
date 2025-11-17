---
description: Build the codebase incrementally in separate branches per PR from a phased plan
argument-hint: [path-to-plan]
allowed-tools: Read, Task, Bash, TodoWrite
---

# Build PR Branches - Sequential PR Implementation

Implements each PR from a phased plan in separate git branches using specialized agents. Each PR is implemented by a dedicated agent that creates a detailed report of its work. Branches are created sequentially (each based on the previous) but are NOT pushed to remote.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

### Phase 1: Setup and Planning

1. **Validate Input**
   - If no `PATH_TO_PLAN` provided, STOP and ask user
   - Verify the plan file exists
   - Read the complete plan

2. **Parse Plan Structure**
   - Extract all PR sections (e.g., "Phase 1.1", "Phase 1.2", "Phase 2.1")
   - For each PR, extract:
     - PR identifier and title
     - PR size and estimated effort
     - Tests to write first (TDD)
     - Implementation details
     - Feature documentation requirements
     - Validation steps
     - Success criteria

3. **Create Todo List**
   - Use TodoWrite to create a todo for each PR
   - Track progress through implementation

4. **Verify Git State**
   - Check current branch
   - Ensure working directory is clean
   - Note the base branch for reference

### Phase 2: Sequential Implementation

For each PR in the plan (in order):

1. **Prepare Branch**
   - Determine branch name: `pr-<phase>-<number>` (e.g., `pr-1-1`, `pr-2-1`)
   - If first PR: base on current branch
   - If subsequent PR: base on previous PR's branch
   - Create branch: `git checkout -b <branch-name> <base-branch>`
   - Mark todo as in_progress

2. **Spawn Implementation Agent**
   - Use Task tool with subagent_type="general-purpose"
   - Provide COMPLETE context (agent has no prior knowledge):

   ```
   # Implementation Task: PR <PR_ID> - <PR_TITLE>

   ## Context
   You are implementing a specific PR as part of a larger implementation plan.

   - **Plan File**: <PATH_TO_PLAN>
   - **Current Branch**: <BRANCH_NAME>
   - **Base Branch**: <BASE_BRANCH>
   - **PR Identifier**: <PR_ID> (e.g., "Phase 1.1")

   ## Important Notes
   - You are working in an ISOLATED environment
   - This is PR <N> of <TOTAL> in the overall plan
   - DO NOT commit changes or create actual GitHub PRs
   - DO NOT push to remote
   - Focus ONLY on this specific PR's scope

   ## Your Implementation Task

   ### PR Details
   - **Size**: <PR_SIZE>
   - **Estimated Effort**: <EFFORT>
   - **Description**: <PR_DESCRIPTION>

   ### Tests to Write FIRST (TDD Approach)
   <FULL_TESTS_SECTION_FROM_PLAN>

   ### Implementation Requirements
   <FULL_IMPLEMENTATION_SECTION_FROM_PLAN>

   ### Feature Documentation
   <FEATURE_DOC_REQUIREMENTS>

   Action: Create or update the feature doc file as specified

   ### Validation Steps
   <VALIDATION_SECTION_FROM_PLAN>

   ### Success Criteria
   <SUCCESS_CRITERIA_FROM_PLAN>

   ## Implementation Instructions

   1. **Write Tests First** (TDD)
      - Create all test files mentioned in "Tests to Write First"
      - Write comprehensive test cases
      - Run tests to verify they fail (as expected before implementation)

   2. **Implement the Feature**
      - Follow the implementation details exactly
      - Make tests pass
      - Follow repository patterns and standards

   3. **Validate Your Work**
      - Run: `yarn build` - must succeed
      - Run: `yarn test` - must pass
      - Follow any additional validation steps

   4. **Create/Update Feature Documentation**
      - Create or update the feature doc as specified
      - Include all required sections

   5. **Generate Statistics**
      - Run: `git diff --stat` to get change statistics
      - Count files created, modified, and tests added

   ## Required Report

   When you finish, create a detailed markdown report at:
   **tmp/pr-<PR_ID>-report.md** (e.g., tmp/pr-1-1-report.md)

   Use this exact template:

   ```markdown
   # PR <PR_ID>: <PR_TITLE>

   **Branch**: <BRANCH_NAME>
   **Date**: <ISO_TIMESTAMP>
   **Status**: ✅ Complete | ⚠️ Partial | ❌ Failed

   ## Summary
   [2-3 sentences describing what was implemented and why]

   ## Changes Made

   ### Files Created
   - `path/to/file1.ts` (123 lines) - Description
   - `path/to/file2.tsx` (456 lines) - Description

   ### Files Modified
   - `path/to/existing1.ts` (+45/-12 lines) - What changed
   - `path/to/existing2.tsx` (+23/-5 lines) - What changed

   ### Tests Added
   1. **Test File**: `path/to/test1.test.ts`
      - Test case 1: Description
      - Test case 2: Description
      - Test case 3: Description
   2. **Test File**: `path/to/test2.test.ts`
      - Test case descriptions...

   ### Feature Documentation
   - Created/Updated: `docs/features/<name>.md`
   - Sections included: [list sections]

   ## Build & Test Results

   ### Build Output
   ```
   [Paste yarn build output - should show success]
   ```

   ### Test Output
   ```
   [Paste yarn test output - show passing tests]
   ```

   ### Git Statistics
   ```
   [Paste output of: git diff --stat]
   ```

   ## Validation Checklist
   - [ ] All tests written first (TDD)
   - [ ] All tests passing
   - [ ] Build successful
   - [ ] Feature documentation created/updated
   - [ ] Code follows repository patterns
   - [ ] Success criteria met

   ## Issues Encountered
   [List any issues and how they were resolved, or "None"]

   ## Notes
   [Any additional observations or recommendations]
   ```

   ## What to Return

   Return ONLY the path to your report file:
   **tmp/pr-<PR_ID>-report.md**

   This allows the parent process to track your work and compile a final summary.
   ```

3. **Wait for Agent Completion**
   - Agent creates: `tmp/pr-<PR_ID>-report.md`
   - Agent returns: report file path
   - Read the report to verify completion

4. **Commit Changes**
   - Stage all changes: `git add .`
   - Extract summary from agent report
   - Create commit message:
     ```
     PR <PR_ID>: <PR_TITLE>

     <Summary from agent report>

     Changes:
     - Files created: <count>
     - Files modified: <count>
     - Tests added: <count>

     Part of: <PATH_TO_PLAN>
     ```
   - Commit: `git commit -m "<message>"`
   - DO NOT PUSH

5. **Update Progress**
   - Mark current PR todo as completed
   - Move to next PR (which will branch from this one)

### Phase 3: Final Report

After all PRs are implemented, generate comprehensive final report.

1. **Collect All PR Reports**
   - Read all `tmp/pr-*-report.md` files
   - Extract statistics from each

2. **Generate Branch Tree Visualization**
   - Show branch hierarchy
   - Indicate which branch is based on which

3. **Create Final Summary**
   - Create: `tmp/build-pr-branches-final-report.md`
   - Use template below

4. **Display Summary to User**
   - Show branch tree
   - Show statistics table
   - Provide next steps

## Templates

### Final Report Template

File: `tmp/build-pr-branches-final-report.md`

```markdown
# Build PR Branches - Final Report

**Plan**: <PATH_TO_PLAN>
**Execution Date**: <ISO_TIMESTAMP>
**Total PRs**: <COUNT>
**Overall Status**: ✅ All Complete | ⚠️ Partial | ❌ Some Failed

---

## Branch Tree

```
<BASE_BRANCH> (e.g., main or staging, or current branch)
└── pr-1-1: Phase 1.1 - <Title>
    └── pr-1-2: Phase 1.2 - <Title>
        └── pr-2-1: Phase 2.1 - <Title>
            └── pr-2-2: Phase 2.2 - <Title>
                └── pr-3-1: Phase 3.1 - <Title>
                    └── ...
```

---

## Implementation Summary by Phase

### Phase 1: <Phase Title>
**Status**: ✅ Complete

- **PR 1.1**: <Title> - ✅ Complete
  - Files: X created, Y modified
  - Tests: Z added
  - [View Report](./pr-1-1-report.md)

- **PR 1.2**: <Title> - ✅ Complete
  - Files: X created, Y modified
  - Tests: Z added
  - [View Report](./pr-1-2-report.md)

### Phase 2: <Phase Title>
**Status**: ✅ Complete

- **PR 2.1**: <Title> - ✅ Complete
  - [View Report](./pr-2-1-report.md)

[Continue for all phases...]

---

## Statistics Overview

| Metric | Count |
|--------|-------|
| Total PRs Implemented | <COUNT> |
| Branches Created | <COUNT> |
| Files Created | <COUNT> |
| Files Modified | <COUNT> |
| Total Tests Added | <COUNT> |
| Total Lines Added | <COUNT> |
| Total Lines Removed | <COUNT> |
| Build Status | ✅ All Passed |
| Test Status | ✅ All Passed |

---

## Detailed PR Summary

| PR ID | Title | Status | Files Changed | Tests | Branch |
|-------|-------|--------|---------------|-------|--------|
| 1.1 | <Title> | ✅ | 5 (3 new, 2 mod) | 12 | pr-1-1 |
| 1.2 | <Title> | ✅ | 3 (1 new, 2 mod) | 8 | pr-1-2 |
| 2.1 | <Title> | ✅ | 7 (4 new, 3 mod) | 15 | pr-2-1 |
| ... | ... | ... | ... | ... | ... |

---

## Individual PR Reports

1. [PR 1.1: <Title>](./pr-1-1-report.md)
2. [PR 1.2: <Title>](./pr-1-2-report.md)
3. [PR 2.1: <Title>](./pr-2-1-report.md)
[Continue for all PRs...]

---

## Next Steps

### 1. Review Individual Branches

Each branch can be reviewed independently:

```bash
# Review first PR
git checkout pr-1-1
git diff <base-branch>

# Review second PR
git checkout pr-1-2
git diff pr-1-1

# Review third PR
git checkout pr-2-1
git diff pr-1-2
```

### 2. Create Actual Pull Requests (If Desired)

Since PR numbers in the plan are sequential and NOT real GitHub PR numbers:

```bash
# For each branch, push and create PR:
git checkout pr-1-1
git push -u origin pr-1-1
gh pr create --base main --title "Phase 1.1: <Title>" --body "$(cat tmp/pr-1-1-report.md)"

git checkout pr-1-2
git push -u origin pr-1-2
gh pr create --base main --title "Phase 1.2: <Title>" --body "$(cat tmp/pr-1-2-report.md)"

# Continue for each PR...
```

**Note**: Each PR should be created against `staging` (or your base branch), NOT against the previous PR branch.

### 3. Merge Strategy Options

**Option A: Sequential Merging**
1. Merge `pr-1-1` into `staging`
2. Rebase `pr-1-2` onto `staging`, then merge
3. Rebase `pr-2-1` onto `staging`, then merge
4. Continue for each PR

**Option B: Merge as Cumulative**
- Since each branch includes all previous changes, you could merge just the final branch
- But this loses the ability to review changes incrementally

**Option C: Cherry-pick**
- Review the commits from each branch
- Cherry-pick only the relevant commits to main

### 4. Testing the Full Implementation

```bash
# Switch to the last branch (has all changes)
git checkout <last-pr-branch>
```

- Run tests according to repository rules

---

## Troubleshooting

### If a PR Failed

1. Check the PR's report in `tmp/pr-<id>-report.md`
2. Review the error messages
3. Checkout the branch: `git checkout pr-<id>`
4. Fix the issues manually
5. Update the report
6. Commit the fixes

### If You Need to Restart from a Specific PR

1. Checkout the last successful PR's branch
2. Delete failed branch: `git branch -D pr-<failed-id>`
3. Re-run this command with `--start-from=<pr-id>` flag (if implemented)
4. Or manually continue from that point

---

## Plan Reference

**Original Plan**: <PATH_TO_PLAN>

All implementation details, test requirements, and success criteria can be found in the original plan document.

---

**Generated**: <TIMESTAMP>
**Build Command**: `/build-pr-branches <PATH_TO_PLAN>`
```

---

## Error Handling

### If Agent Fails to Complete PR

1. Create partial report noting:
   - What was completed
   - What failed
   - Error messages
   - Partial changes made

2. Mark as ⚠️ Partial or ❌ Failed

3. Commit what exists (if anything valuable)

4. Continue with next PRs (note: they may also fail due to dependencies)

5. Include failure details in final report

### If Build/Tests Fail

1. Agent should include full error output in report
2. Mark PR as failed
3. DO NOT proceed to commit
4. Continue with remaining PRs (document cascade failures)

### If Git Operations Fail

1. Log the git error
2. Attempt to recover (e.g., stash changes, retry)
3. If unrecoverable, halt and report to user

---

## Important Constraints

- ❌ DO NOT push any branches to remote
- ❌ DO NOT create actual GitHub PRs
- ❌ DO NOT modify the plan file
- ✅ DO keep all work local for review
- ✅ DO create comprehensive reports
- ✅ DO use TDD (tests first)
- ✅ DO run build and tests for each PR
- ✅ DO provide complete context to each agent

---

## Performance Notes

- Each agent runs independently (stateless)
- Agents run sequentially (not parallel) due to git branching dependencies
- Expected time: <effort-from-plan> per PR
- Total time: Sum of all PR efforts

---

## Example Execution

```bash
# User runs:
/build-pr-branches docs/plans/PLAN.md

# System creates branches:
pr-1-1  (from main)
pr-1-2  (from pr-1-1)
pr-2-1  (from pr-1-2)
pr-2-2  (from pr-2-1)
...

# System generates reports:
tmp/pr-1-1-report.md
tmp/pr-1-2-report.md
tmp/pr-2-1-report.md
tmp/pr-2-2-report.md
...
tmp/build-pr-branches-final-report.md

# User can then review:
- Each branch independently
- Each PR report
- Final summary report
```
