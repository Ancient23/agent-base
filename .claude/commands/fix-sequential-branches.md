---
description: Fix an issue in a sequential branch tree and propagate the fix to all dependent branches
---

# Fix Sequential Branches

You are tasked with fixing an issue in a sequential branch tree where each branch is based on the previous one.

## Input Required

The user will provide:
1. **Error log or issue description**
2. **Branch tree structure** (or you should analyze git branches to determine it)

## Workflow

### Step 1: Analyze the Error

1. Read the error log carefully
2. Identify:
   - Root cause (file, function, line number)
   - Which PR/branch introduced the issue
   - What fix is needed

### Step 2: Identify the Source Branch

From the error, determine which branch introduced the bug:
- Check git blame or commit history
- Look at file modification timestamps
- Trace the feature implementation through the branch tree

### Step 3: Apply the Fix

1. Checkout the branch where the bug was introduced
2. Apply the fix to the relevant file(s)
3. Rebuild if necessary (e.g., `yarn build:shared` for shared package changes)
4. Run tests to verify the fix
5. Commit with descriptive message:
   ```
   Fix: [Brief description]

   [Detailed explanation of what was wrong]
   [What the fix does]
   [Error it resolves]
   ```

### Step 4: Propagate to Dependent Branches

For each branch that was based on the fixed branch (in order):

1. Checkout the dependent branch
2. Rebase onto its parent: `git rebase <parent-branch>`
3. Handle any conflicts if they arise
4. Continue to next branch

Example for a chain: `A → B → C → D`
- Fix applied to branch `B`
- Rebase `C` onto `B`
- Rebase `D` onto `C`

### Step 5: Verify

1. Rebuild shared packages if applicable
2. Run tests on the final branch
3. Verify the error is resolved

## Important Rules

- **Never amend commits** - Always create new fix commits
- **Rebase, don't merge** - Use `git rebase` to propagate changes
- **Test after each step** - Ensure nothing breaks during rebasing
- **Document the fix** - Create a summary in `tmp/` if the fix is significant

## Output

Provide a summary showing:
1. ✅ Branch where fix was applied
2. ✅ All branches that were updated
3. ✅ Test results
4. ✅ Next steps for the user

## Example Execution

```bash
# Step 1: Fix on source branch
git checkout branch-b
# Apply fix
git commit -m "Fix: [description]"

# Step 2: Rebase dependent branches
git checkout branch-c
git rebase branch-b

git checkout branch-d
git rebase branch-c

# Step 3: Verify
yarn build:shared
yarn test
```

## Tips

- If the branch tree is complex, draw it out first
- Check `git log --graph --oneline` to visualize relationships
- Keep fixes atomic - one logical change per commit
- If multiple files need fixes, consider if they should be separate commits
