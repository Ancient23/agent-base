---
description: Rebase all stacked PRs after updating the foundation PR
argument-hint: [pr-number-that-changed]
allowed-tools: Read, Bash, TodoWrite
---

# Rebase Stack

Follow the `Workflow` to rebase all child PRs after changes to a parent PR, then `Report` the results.

## Variables

CHANGED_PR: $ARGUMENTS (required - the PR number that was updated)

## Workflow

### Step 1: Discover Stack Structure

1. **Get information about the changed PR:**
   ```bash
   gh pr view $CHANGED_PR --json number,headRefName,baseRefName,title
   ```

2. **Discover dependent PRs:**
   - Search for PRs that depend on the changed PR by checking which PRs have `baseRefName` matching the changed PR's `headRefName`
   - Build the dependency tree by recursively finding PRs that depend on each discovered PR
   - Use:
     ```bash
     gh pr list --json number,headRefName,baseRefName,title --limit 100
     ```
   - Filter and map the relationships to build a tree structure

3. **Verify current branch:**
   ```bash
   git branch --show-current
   ```

4. **Check for uncommitted changes:**
   ```bash
   git status --porcelain
   ```
   - If there are uncommitted changes, stash them before proceeding

### Step 2: Determine Rebase Order

Based on the dependency tree discovered in Step 1:

1. **Build rebase order** by traversing the dependency tree:
   - Start with direct children of `CHANGED_PR`
   - For each child, recursively add its children
   - Maintain topological order (parents before children)

2. **Create a todo list** with all PRs that need rebasing in the correct order

3. **Display the rebase plan** to show the user:
   ```
   Rebase Order:
   1. PR #<num>: <branch-name> (depends on <parent-branch>)
   2. PR #<num>: <branch-name> (depends on <parent-branch>)
   ...
   ```

### Step 3: Execute Rebase Cascade

For each PR in the rebase order (from the todo list):

1. **Fetch latest changes:**
   ```bash
   git fetch origin
   ```

2. **Switch to the branch:**
   ```bash
   git checkout <branch-name>
   ```

3. **Ensure branch is up to date with remote:**
   ```bash
   git pull origin <branch-name>
   ```

4. **Rebase onto parent branch:**
   ```bash
   git rebase <parent-branch-name>
   ```

   - If rebase succeeds, continue to next step
   - If conflicts occur:
     - Run `git status` to see conflicting files
     - Ask user: "Conflicts detected in <file-list>. How would you like to resolve these?"
     - Options:
       a) Abort rebase and let user handle manually
       b) Show conflict details and get guidance
       c) Auto-resolve if conflicts are simple (imports, formatting)

5. **Verify the rebase:**
   - Check that the branch history looks correct with `git log --oneline -5`
   - Show the commit count difference: `git rev-list --count HEAD ^origin/<branch-name>`

6. **Push the rebased branch:**
   ```bash
   git push --force-with-lease origin <branch-name>
   ```

7. **Mark todo as completed** and move to next branch

### Step 4: Handle Special Cases

**For PRs with multiple parent branches** (merge commits):
- If a PR has multiple base branches (e.g., merges two feature branches):
  1. Rebase onto the primary parent first
  2. Re-merge the secondary parent: `git merge <secondary-parent> --no-edit`
  3. Verify all parent changes are present: `git log --oneline --graph -15`

**For PRs with merge conflicts:**
- If conflicts are detected, provide clear guidance on which files conflict
- Show the conflict markers and let user decide how to proceed
- After manual resolution, continue: `git rebase --continue`

### Step 5: Validate All Branches

After rebasing all branches:

1. **Verify git history for each rebased branch:**
   ```bash
   git log --oneline --graph -10
   ```

2. **Check that each branch is ahead of its remote:**
   ```bash
   git status
   # Should show: "Your branch is up to date with 'origin/<branch>'"
   ```

3. **Optional: Run project-specific validation** (if build/test commands are available):
   - Look for package.json, Makefile, or other build scripts
   - If found, ask user if they want to run validation
   - Examples: `npm run build`, `make test`, `npm test`, etc.

## Report

After completing the rebase cascade, provide a summary:

1. **Branches Rebased:**
   - List each PR and branch that was rebased
   - Show before/after commit hashes
   - Note any conflicts that were resolved

2. **Rebase Statistics:**
   For each rebased branch, show new commits:
   ```bash
   git log --oneline origin/<branch-name>..HEAD
   ```

3. **Action Items:**
   - List any manual steps the user needs to take
   - Note if any conflicts required manual resolution
   - Recommend which PRs need re-review due to significant changes

4. **Verification Steps:**
   - Confirm all rebases completed successfully
   - Suggest testing each updated PR locally if needed
   - Recommend adding PR comments to notify reviewers of the rebase

## Example Output

```markdown
# Rebase Cascade Complete

## Summary
Rebased all dependent PRs after changes to PR #<changed-pr> (<branch-name>).

## Branches Updated
✅ PR #<num> (<branch-name>): abc1234 → def5678 (3 commits ahead)
✅ PR #<num> (<branch-name>): ghi9012 → jkl3456 (5 commits ahead)
✅ PR #<num> (<branch-name>): mno7890 → pqr1234 (4 commits ahead)

## Conflicts Resolved
[If any] Conflicts were encountered in:
- PR #<num>: <file1>, <file2> (resolved manually)

[If none] No conflicts encountered during rebase.

## Validation
✅ All branches successfully rebased
✅ All branches pushed to remote
✅ Git history is clean

## Next Steps
1. Add comment to each PR notifying reviewers of rebase
2. Re-run CI/CD pipelines if they were interrupted
3. Test PRs locally if significant conflicts were resolved

## Recommended PR Comment Template
> **Rebased onto updated PR #<changed-pr>**
>
> This PR has been rebased to include the latest changes from PR #<changed-pr>.
>
> [If conflicts] Conflicts resolved in: <file-list>
>
> Please re-review if needed.
```

## Error Handling

If errors occur during the workflow:

1. **Conflict during rebase:**
   - Stop the workflow immediately
   - Show the conflicting files with `git status`
   - Provide guidance: "Conflicts detected in: <files>. Options:"
     ```bash
     # Option 1: Resolve conflicts manually
     # Edit conflicting files, then:
     git add <files>
     git rebase --continue

     # Option 2: Abort and handle later
     git rebase --abort

     # After resolution, you can re-run /rebase-stack with remaining PRs
     ```

2. **Validation failure after rebase:**
   - Show the error message
   - Suggest: "Validation failed. This may indicate a breaking change. Review the error before pushing."
   - Offer to abort the rebase: `git rebase --abort`

3. **Force-push rejected:**
   - Show the error
   - Suggest: "Push rejected. Someone may have pushed to this branch. Options:"
     ```bash
     # Fetch and review changes
     git fetch origin
     git log HEAD..origin/<branch-name>

     # If safe, force push (use with caution)
     git push --force origin <branch-name>
     ```

4. **PR dependency discovery fails:**
   - If unable to fetch PR information via `gh` CLI
   - Ensure GitHub CLI is authenticated: `gh auth status`
   - Suggest manual dependency tree input from user

## Safety Checks

**Before rebasing each branch:**
- Verify we're on the correct branch: `git branch --show-current`
- Verify the parent branch exists locally: `git rev-parse --verify <parent-branch>`
- Check for uncommitted changes: `git status --porcelain`
- Stash uncommitted changes if needed: `git stash push -m "rebase-stack auto-stash"`

**After rebasing each branch:**
- Verify commit count looks reasonable
- Check that rebase didn't lose commits (compare with remote)
- Ensure the branch is ahead of its remote counterpart
- Verify no duplicate commits were created

## Notes

- This command uses `--force-with-lease` for safety (never `--force`)
- Each rebase is done individually to catch errors early
- The workflow maintains topological order based on PR dependencies
- Validation steps are optional to keep the command fast
- User can abort at any point and handle rebases manually if needed
- Works with any repository that uses GitHub Pull Requests
- Requires GitHub CLI (`gh`) to be installed and authenticated
