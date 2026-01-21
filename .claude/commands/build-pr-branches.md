---
description: Build the codebase incrementally in separate branches per PR from a phased plan
argument-hint: [path-to-plan]
allowed-tools: Read, Write, Edit, Bash, TodoWrite, Glob, Grep
---

# Build PR Branches

Implements each PR from a phased plan in separate git branches. Each PR must be fully completed within this context window. DO NOT spawn subagents - implement directly.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

1. **Validate**: If no `PATH_TO_PLAN` provided, STOP and ask user
2. **Read plan** and extract all PR sections
3. **Check git state**: Ensure clean working directory, note current branch as base
4. **Create TodoWrite** with one item per PR

For each PR in order:

1. **Create branch** with descriptive name based on the work:
   - Format: `<type>/<short-description>` (e.g., `feature/user-auth-api`, `fix/cart-total-calc`, `refactor/payment-service`)
   - First branch from base, subsequent from previous PR branch
2. Mark todo as in_progress
3. **Implement the PR directly** (no subagents):
   - Write tests first (TDD)
   - Implement the feature
   - Build and run tests
4. Commit: `git add . && git commit`
5. **Update plan status**: Edit `PATH_TO_PLAN` to mark phase as complete with date
6. Mark todo as completed

After all PRs:
- **Track technical debt**: Append a `## Technical Debt` section to the plan with any shortcuts, TODOs, or improvements deferred during implementation
- **Inform user**: List the technical debt items and recommend addressing them in follow-up PRs

## Codebase Rules (from AGENTS.md)

You MUST follow these rules during implementation:

- **No `type any`** - search codebase for suitable types
- **No magic numbers** - use constants (exception: test files)
- **Unit tests required** for API and smart contract changes
- **DTOs for client-server data** - use shared/src/models/<Model>.types.ts pattern
- **Constants over string checks** - create constants instead of checking string values
- **Use hooks/useApi** for API calls (bundles auth)
- **No ts-nocheck**
- **Wrap client API calls in try-catch**
- **POST endpoints return 201** with Location header
- **Keep endpoint docs updated** in devportal/content/api-endpoints
- **Client/Server boundaries**: use `*.server.ts` with `import 'server-only'` for server logic

## Context Window Limits

**CRITICAL**: Each PR must be completed within this context window.

If you cannot complete the current PR:
1. **STOP immediately**
2. Commit any completed work with `[WIP]` prefix
3. Output this continuation prompt for the user:

```
---CONTINUATION PROMPT---
Continue implementing PR <PR_ID> from plan: <PATH_TO_PLAN>

Current state:
- Branch: <current-branch>
- Completed: <what was done>
- Remaining: <what still needs to be done>
- Last file touched: <filename>

Resume from: <specific next step>
---END PROMPT---
```

## Commit Message Format

```
<type>: <short description>

- <brief change summary>
```

Types: `feat`, `fix`, `refactor`, `test`, `chore`

## Constraints

- DO NOT push branches to remote
- DO NOT create GitHub PRs
- DO NOT generate summary.md files
- DO keep all work local
- DO let other agents handle PR review
