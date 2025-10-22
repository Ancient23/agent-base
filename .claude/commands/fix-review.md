---
description: Systematically fix all issues from a code review document using parallel agents
argument-hint: [path-to-review] [--pr <number>]
allowed-tools: Read, Task, Bash
---

# Fix Review

Parse a code review document, organize issues into parallelizable tasks with dependency tracking, launch specialized agents to fix each issue, commit changes incrementally, reply to review comments, and generate a comprehensive completion report.

## Variables

REVIEW_PATH: $1
PR_NUMBER: $2
REVIEW_REPLIES_TRACKING_FILE: `/tmp/review-replies-tracking.json`
COMPLETION_REPORT_PATH: `/tmp/fix-review-completion-report.md`

## Workflow

### Phase 1: Parse and Organize

1. **Validate Inputs**
   - If no `REVIEW_PATH` is provided, STOP and ask user for the review document path
   - Read the review document at `REVIEW_PATH`
   - Extract `PR_NUMBER` from the review document URL or from `--pr` argument
   - If no PR number found, STOP and ask user for PR number

2. **Parse Review Document**
   - Extract all issues from the review document by scanning for:
     - Section headers indicating priority (BLOCKING, CRITICAL, Code Quality, etc.)
     - Issue numbers (e.g., #1, #2, #3 or ### 1., ### 2.)
     - Location references (file:line format)
   - For each issue, capture:
     - Issue number/identifier
     - Title/summary
     - Priority level (BLOCKING, CRITICAL, Code Quality, UI/UX, Performance, etc.)
     - Location (file:line references)
     - Current code snippet (code blocks marked with "Current" or "❌")
     - Required fix (code blocks marked with "Required" or "✅")
     - Action items list
     - Reviewer name (if mentioned)
     - Full issue description

3. **Build Dependency Graph**

   Analyze issues to identify dependencies using these rules:

   **Level 0 (No Dependencies) - Execute First:**
   These tasks can run immediately in parallel:
   - Documentation-only changes (comments, markdown files, ESLint explanations)
   - Independent constant definitions (within same file)
   - UI/UX fixes (ARIA attributes, styling, colors)
   - Input validation improvements
   - Type annotations (return types, parameter types)
   - Simple field additions (single-line fixes)

   **Level 1 (Setup/Infrastructure) - Execute Second:**
   These create shared resources that others depend on:
   - Feature flag setup (creates config entries)
   - Shared type definitions (creates files in shared/src/types/)
   - Shared utility functions (creates files in shared/src/utils/)
   - Shared library functions (adds to shared/lib/)
   - Shared constants (creates constants files)

   **Level 1b (Build Shared) - After Level 1:**
   - Run `yarn build:shared` after ALL Level 1 tasks complete
   - This is a single task that depends on all shared package changes

   **Level 2 (Use Shared Resources) - Execute Third:**
   These use resources created in Level 1:
   - Extract duplicate code to shared (imports Level 1 utilities)
   - Refactor to use shared libraries (imports Level 1 functions)
   - Add feature flag guards (uses Level 1 flags)
   - Update components with shared types (imports Level 1 types)
   - Fix import errors (depends on shared build)

   **Level 3 (Testing & Documentation) - Execute Fourth:**
   - Unit tests (tests Level 2 code)
   - Integration tests (tests features with flags)
   - Feature documentation (documents all changes)

   **Level 4 (Verification) - Execute Last:**
   - Build verification (`yarn build`)
   - Test suite execution (`yarn test`)

   **Dependency Detection Algorithm:**
   ```
   For each issue:
     1. Classify issue type based on keywords in title/description:
        - "feature flag" → feature-flag-setup (Level 1)
        - "duplicate" + "type" → shared-type-def (Level 1)
        - "duplicate" + "function" → shared-utility (Level 1)
        - "Bridge API" / "shared/lib" → shared-library (Level 1)
        - "extract to shared" → extract-to-shared (Level 2)
        - "use shared" / "import" → update-with-shared-types (Level 2)
        - "guard" / "wrap" → add-feature-flag-guards (Level 2)
        - "test" → unit-tests or integration-tests (Level 3)
        - "docs" / "documentation" → feature-docs (Level 3)
        - "comment" only → docs-only (Level 0)
        - "ARIA" / "accessibility" → ui-styling (Level 0)
        - "constant" / "magic number" → constants (Level 0)

     2. Determine what it creates:
        - If creates new file in shared/src/types/ → creates: type definitions
        - If creates new file in shared/src/utils/ → creates: utility functions
        - If adds function to shared/lib/ → creates: library function
        - If adds feature flag → creates: feature-flag:<name>

     3. Determine what it requires:
        - If uses "import from @shared" → requires: shared build
        - If wraps with feature flag → requires: feature-flag:<name>
        - If imports type → requires: type definition
        - If imports utility → requires: utility function

     4. Assign to dependency level based on type and requirements
   ```

   **Conflict Detection:**
   - Check if multiple issues modify the same file
   - If conflicts detected: Serialize those issues within their level
   - If multiple issues modify shared package: Run all Level 1 tasks serially

4. **Create Task Execution Plan**
   Generate execution plan organized by dependency levels:
   ```json
   {
     "pr_number": "XXXX",
     "review_path": "docs/reviews/pr-XXXX-....md",
     "total_issues": <count>,
     "execution_levels": [
       {
         "level": 0,
         "description": "Independent fixes",
         "parallel": true,
         "issues": [<issue objects>]
       },
       {
         "level": 1,
         "description": "Create shared infrastructure",
         "parallel": false,
         "issues": [<issue objects>]
       },
       {
         "level": 1.5,
         "description": "Build shared package",
         "parallel": false,
         "tasks": ["yarn build:shared"]
       },
       {
         "level": 2,
         "description": "Use shared infrastructure",
         "parallel": true,
         "issues": [<issue objects>]
       }
     ]
   }
   ```

### Phase 2: Execute Tasks in Parallel Waves

For each dependency level (starting with Level 0):

1. **Launch Parallel Agent Tasks**
   - For each issue in the current level, launch a Task agent with `subagent_type: general-purpose`
   - Provide each agent with a comprehensive prompt using this template:

   ```markdown
   You are fixing issue #{ISSUE_ID} from code review for PR #{PR_NUMBER}.

   ## Issue Summary
   **Title**: {ISSUE_TITLE}
   **Priority**: {PRIORITY}
   **Reviewer**: {REVIEWER}
   **Type**: {ISSUE_TYPE}

   ## Problem Description
   {FULL_ISSUE_DESCRIPTION}

   ## Location
   Files affected:
   {FILE_PATHS_AND_LINES}

   ## Current Code
   ```{LANGUAGE}
   {CODE_SNIPPET_SHOWING_PROBLEM}
   ```

   ## Required Fix
   {DETAILED_FIX_INSTRUCTIONS}

   {IF_APPLICABLE: Code example of required fix}

   ## Action Items
   {LIST_OF_ACTION_ITEMS}

   ## Repository Standards (from AGENTS.md)
   - Always add unit tests for API changes
   - Use constants instead of magic strings/numbers
   - Create DTOs for server-client data transfer
   - No `any` types - search codebase for existing types
   - Follow existing patterns in the codebase
   - Run `yarn build && yarn test` before committing

   ## Your Tasks
   1. Read all affected files
   2. Understand the current implementation
   3. Implement the fix following the provided instructions exactly
   4. If modifying shared package:
      - Run `yarn build:shared` after changes
      - Verify exports in shared/src/index.ts
   5. If creating new files:
      - Follow existing file naming conventions
      - Add proper TypeScript types
      - Include JSDoc comments
   6. Run appropriate validation:
      - `yarn build` to verify no build errors
      - `yarn test <pattern>` for relevant tests (if applicable)
   7. Create a descriptive commit message following this format:
      ```
      {type}(review): {brief description}

      Addresses PR #{PR_NUMBER} review comment #{ISSUE_ID}

      {DETAILED_DESCRIPTION_OF_CHANGES}

      {IF_APPLICABLE: "Reviewer: {REVIEWER_NAME}"}
      ```
      Where type is one of: fix, feat, test, docs, refactor, style
   8. Commit the changes: `git add . && git commit -m "<message>"`
   9. Report back with:
      - Files changed: `git diff --stat HEAD~1`
      - Summary of what was fixed
      - Any issues encountered
      - Test results (if tests were run)

   ## Success Criteria
   {EXPLICIT_SUCCESS_CRITERIA}

   ## Special Instructions for Issue Type: {ISSUE_TYPE}

   {IF_TYPE_IS_extract-to-shared:}
   Before creating new shared utilities:
   1. Search for similar functionality: `rg -i "{function_name_keywords}" shared/src/`
   2. If similar code exists, USE IT - do not duplicate
   3. If no shared utility exists:
      - Find appropriate shared file or create new one
      - Update shared/src/index.ts exports
      - Run `yarn build:shared`
      - Update all consuming files to import from shared
      - Delete duplicate code
      - Verify builds

   {IF_TYPE_IS_feature-flag-setup:}
   1. Check for existing feature flags: `rg -i "{flag_keyword}" shared/src/config/`
   2. If exists, USE IT - note which flag to use
   3. If doesn't exist:
      - Add feature flag to config with proper typing
      - Set default value (usually false for unreleased features)
      - Add to environment variable examples if needed
      - Document flag purpose

   {IF_TYPE_IS_add-feature-flag-guards:}
   1. Identify the feature flag name from Level 1 tasks
   2. Wrap all affected UI/routes with flag checks
   3. Add route guards to prevent direct URL access
   4. Search for all references: `rg -i "{feature_path}" {app_directory}/`

   {IF_TYPE_IS_shared-library:}
   1. Check if function already exists: `rg -i "function {function_name}" shared/lib/`
   2. If doesn't exist, add to appropriate shared/lib/ file
   3. Follow existing patterns in that file exactly
   4. Include proper error handling matching other functions
   5. Add TypeScript types
   6. Add JSDoc documentation

   {IF_TYPE_IS_refactor-to-shared-lib:}
   1. Import the new shared function from Level 1
   2. Replace direct API calls with shared function calls
   3. Remove duplicated error handling
   4. Verify behavior matches original

   {IF_TYPE_IS_shared-type-def:}
   1. Check for existing types: `rg -i "type {type_name}" shared/src/types/`
   2. If doesn't exist, add to shared/src/types/
   3. Create new file or add to existing related file
   4. Export from shared/src/index.ts
   5. Document type with JSDoc

   {IF_TYPE_IS_unit-tests:}
   1. Look at existing test files in same directory for patterns
   2. Create test file following naming convention
   3. Test all required scenarios from issue description
   4. Run tests: `yarn test {test_file_pattern}`
   5. Ensure all tests pass

   {IF_TYPE_IS_feature-docs:}
   1. Check if docs/features/{feature_name}.md exists
   2. If exists, read and update; if not, create following AGENTS.md format
   3. Include:
      - Feature overview
      - Files changed (`git diff --stat`)
      - Testing instructions (screens to load, user actions, expected results)
      - Feature flag configuration (if applicable)
      - Summary of code review fixes applied

   IMPORTANT: Focus ONLY on this specific issue. Do not make additional unrelated changes.
   ```

2. **Monitor Agent Completion**
   - Track which agents have completed: Create tracking file with status
   - Collect results from each agent:
     - Commit SHA
     - Files changed
     - Summary of changes
     - Any errors encountered
   - Store in `REVIEW_REPLIES_TRACKING_FILE`:
     ```json
     {
       "issue_id": "X",
       "title": "...",
       "status": "completed" | "failed",
       "commit_sha": "abc123",
       "files_changed": ["file1", "file2"],
       "error": "..." (if failed),
       "timestamp": "ISO-8601"
     }
     ```

3. **Wait for Level Completion**
   - Do not proceed to next dependency level until ALL tasks in current level complete
   - If any task fails:
     - Stop processing that level
     - Report the failure to user
     - List remaining incomplete tasks
     - Provide manual fix instructions
     - Do NOT proceed to next level

4. **Special Level: Build Shared Package**
   After all Level 1 (shared infrastructure creation) tasks complete:
   ```bash
   cd shared && yarn build
   ```
   - Verify build succeeds
   - Check for TypeScript errors
   - Verify exports are accessible
   - If build fails: STOP and report error

### Phase 3: Post-Task Processing

After each task completes successfully:

1. **Reply to Review Comments (if applicable)**
   If the issue references a specific review comment (identifiable by file:line):

   ```bash
   # Find the review comment
   gh api repos/Oncade/gameservice/pulls/{PR_NUMBER}/comments \
     --jq '.[] | select(.path == "{FILE_PATH}" and .original_line == {LINE})' \
     > /tmp/comment.json

   COMMENT_ID=$(cat /tmp/comment.json | jq -r '.id')

   # Post reply to that comment
   if [ ! -z "$COMMENT_ID" ] && [ "$COMMENT_ID" != "null" ]; then
     gh api repos/Oncade/gameservice/pulls/{PR_NUMBER}/comments/${COMMENT_ID}/replies \
       -X POST \
       -f body="✅ Fixed in commit {COMMIT_SHA}

   {DESCRIPTION_OF_WHAT_WAS_FIXED}

   Files changed:
   {FILE_LIST_WITH_CHANGES}

   🤖 Automated fix by Claude Code"
   fi
   ```

2. **Update Tracking File**
   Append each completion to `REVIEW_REPLIES_TRACKING_FILE`

### Phase 4: Generate Final Report

After ALL issues are processed:

1. **Push All Changes**
   ```bash
   # Push all commits at once
   git push origin HEAD
   ```

2. **Create Completion Report**
   Write to `COMPLETION_REPORT_PATH`:
   ```markdown
   # Code Review Fix Report - PR #{PR_NUMBER}

   **Review Document**: {REVIEW_PATH}
   **Total Issues**: {TOTAL_ISSUES}
   **Issues Fixed**: {FIXED_COUNT}
   **Issues Failed**: {FAILED_COUNT}
   **Total Commits**: {COMMIT_COUNT}
   **Completion Date**: {ISO_TIMESTAMP}

   ---

   ## Fixed Issues

   ### BLOCKING Issues ({COUNT})

   #### ✅ #{ISSUE_ID}: {ISSUE_TITLE}
   - **Priority**: BLOCKING
   - **Commit**: {COMMIT_SHA}
   - **Files Changed**:
     {FILE_LIST}
   - **Changes Summary**: {SUMMARY}
   - **Reviewer**: {REVIEWER}

   [Repeat for each issue organized by priority level]

   ---

   ## Build Verification

   ```bash
   $ yarn build
   {BUILD_OUTPUT_SUMMARY}

   $ yarn test
   {TEST_OUTPUT_SUMMARY}
   ```

   ---

   ## Summary Statistics

   - **Total Files Modified**: {COUNT}
   - **Lines Added**: {COUNT}
   - **Lines Removed**: {COUNT}
   - **New Files Created**: {LIST}
   - **Files Deleted**: {LIST}

   ---

   ## Remaining Work (if any)

   [List any issues that could not be automatically fixed with reasons]

   ---

   ## Next Steps

   1. Review individual commits for quality
   2. Run manual tests as specified in docs/features/
   3. Request re-review from original reviewers: {REVIEWER_LIST}
   4. Monitor CI/CD pipeline
   ```

3. **Update/Create Feature Documentation**
   - Check if feature doc exists in `/docs/features/` (look for related doc)
   - If exists, read and update with:
     - New "Code Review Fixes" section
     - Updated file list
     - Updated test scenarios (if changed)
   - If doesn't exist and review indicates it's needed (check for "Missing Feature Documentation" issue):
     Create following AGENTS.md format:
     ```markdown
     # {Feature Name}

     **Date**: {CURRENT_DATE}
     **PR**: #{PR_NUMBER}

     ## Overview
     {FEATURE_DESCRIPTION_FROM_REVIEW_OR_PR}

     ## Files Changed
     {OUTPUT_OF: git diff --stat main...HEAD}

     ## Testing Instructions

     ### Screens to Load
     1. {URL/PATH}
     2. {URL/PATH}

     ### User Actions
     1. {ACTION_DESCRIPTION}
     2. {ACTION_DESCRIPTION}

     ### Expected Results
     1. {EXPECTED_BEHAVIOR}
     2. {EXPECTED_BEHAVIOR}

     ## Code Review Fixes Applied

     This feature addresses all issues from {REVIEW_PATH}:

     ### BLOCKING Issues
     - ✅ #{ID}: {TITLE} - Fixed in {COMMIT_SHA}

     ### CRITICAL Issues
     - ✅ #{ID}: {TITLE} - Fixed in {COMMIT_SHA}

     [Continue for all priority levels]
     ```

4. **Post Summary Comment to PR**
   ```bash
   gh pr comment {PR_NUMBER} --body "$(cat <<'EOF'
   ## Code Review Fixes Complete

   ### Summary
   - **Total Issues Addressed**: {COUNT}
   - **Commits Created**: {COUNT}
   - **Files Modified**: {COUNT}

   ### Issues Fixed

   #### 🚨 BLOCKING ({COUNT})
   - ✅ #{ID}: {TITLE} - [{COMMIT_SHA}]

   #### ❗ CRITICAL ({COUNT})
   - ✅ #{ID}: {TITLE} - [{COMMIT_SHA}]

   #### Code Quality ({COUNT})
   - ✅ #{ID}: {TITLE} - [{COMMIT_SHA}]

   ### Build Status
   ```
   ✅ yarn build - PASSED
   ✅ yarn test - PASSED
   ```

   ### Next Steps
   - Review the changes in individual commits
   - Run manual tests as specified in docs/features/
   - Request re-review when ready

   **Detailed Report**: See {COMPLETION_REPORT_PATH}

   EOF
   )"
   ```

## Report

Provide a concise summary:

```
Code Review Fixes Completed for PR #{PR_NUMBER}

Review Document: {REVIEW_PATH}
Issues Processed: {TOTAL}
  - BLOCKING: {COUNT} fixed
  - CRITICAL: {COUNT} fixed
  - Code Quality: {COUNT} fixed
  - UI/UX: {COUNT} fixed
  - Other: {COUNT} fixed

Total Commits: {COUNT}
Total Files Changed: {COUNT}

Build Status: ✅ PASSED / ❌ FAILED
Test Status: ✅ PASSED / ❌ FAILED

Completion Report: {COMPLETION_REPORT_PATH}
Feature Docs Updated: {PATHS}

GitHub Updates:
  ✅ PR comment posted
  ✅ {COUNT} review comment replies posted

Next Steps:
1. Review individual commits: git log HEAD~{COUNT}..HEAD
2. Run manual testing per docs/features/
3. Request re-review from original reviewers
```

## Error Handling

If any task fails:
1. Stop processing that dependency level
2. Report the failure with full error details:
   - Issue that failed
   - Error message from agent
   - What was attempted
   - Files that were modified (if any)
3. List remaining incomplete tasks
4. Provide manual fix instructions
5. Create a summary of what was completed successfully
6. Do NOT proceed to next dependency level

User can then fix manually and resume with remaining levels if desired.

## Special Issue Type Handlers

### Extract to Shared
For issues requiring extracting code to shared package:
- Agent must:
  1. Check if similar code already exists, and if so DO NOT DUPLICATE and just use this
  2. If there is no shared utility code for this function yet, find the appropriate shared file for the new function or create a new shared utility file
  3. Update shared/src/index.ts exports
  4. Run `yarn build:shared`
  5. Update all consuming files to import from shared
  6. Delete duplicate code
  7. Verify builds

### Feature Flag
For feature flag issues:
- Agent must:
  1. Check for existing feature flag (or ask if one exists) and either use existing or add a new feature flag to config
  2. Wrap all affected UI/routes with flag checks
  3. Add flag to environment variables (if doesn't yet exist)
  4. Update documentation

### API Pattern Refactor
For issues requiring API pattern changes:
- Agent must:
  1. Add new function to correct shared file (if it doesn't yet exist)
  2. Follow existing patterns in that file
  3. Update route to use new function
  4. Verify error handling matches

### Type Definition
For issues requiring new type definitions:
- Agent must:
  1. Add types to shared/src/types/
  2. Update shared exports
  3. Run `yarn build:shared`
  4. Update consuming files
  5. Remove old type assertions

### Unit Tests
For issues requiring unit tests:
- Agent must:
  1. Look at existing tests for patterns
  2. Create test file with proper naming
  3. Test all scenarios listed in issue
  4. Run tests and verify they pass
  5. Include in commit

### Documentation
For documentation issues:
- Agent must:
  1. Check if doc file exists
  2. Create or update following AGENTS.md format
  3. Include all required sections
  4. Reference all changes made

## Notes

- Each agent operates independently with full context
- Commits are atomic per issue for easy review
- Failed tasks can be manually fixed without blocking other work
- All changes are tracked and reported transparently
- GitHub integration automates review comment replies
- Final report provides complete audit trail
- The command adapts to any review document format
- Works with any PR in any repository following the same standards
