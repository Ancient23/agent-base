---
description: Build the codebase using Codex MCP for intelligent code generation
argument-hint: [path-to-plan]
allowed-tools: mcp__codex__codex, mcp__codex__codex-reply, Read, Write, Bash
---

# Build with Codex

Implement a development plan using Codex MCP's advanced code generation capabilities. Codex will analyze the plan, understand the codebase context, and generate high-quality implementation code.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

1. **Validate Input**
   - If no `PATH_TO_PLAN` is provided, STOP and request it from user

2. **Prepare Context**
   - Read the plan document at `PATH_TO_PLAN`
   - Extract implementation requirements and phases

3. **Execute with Codex**
   - Invoke mcp__codex__codex with:
     - The implementation plan details
     - Working directory context
     - Sandbox mode: `workspace-write` for safe file modifications
     - Approval policy: `on-failure` to handle errors gracefully

4. **Iterative Implementation**
   - For multi-phase plans, use mcp__codex__codex-reply to continue conversation
   - Guide Codex through each phase sequentially
   - Verify implementation at each phase

5. **Verification**
   - Run tests if specified in plan
   - Check git diff to review changes
   - Ensure all plan requirements are met

## Codex Configuration

When invoking Codex, use these parameters:

```json
{
  "prompt": "[Implementation instructions from plan]",
  "sandbox": "workspace-write",
  "approval-policy": "on-failure",
  "cwd": "[current working directory]",
  "base-instructions": "You are implementing a development plan. Follow the specifications exactly, write clean idiomatic code, and ensure all tests pass."
}
```

## Multi-Phase Handling

For plans with multiple phases:
1. Start with first phase using mcp__codex__codex
2. Save the conversationId from response
3. Continue with subsequent phases using mcp__codex__codex-reply
4. Pass phase-specific instructions in each reply

## Report

After implementation, generate feature documentation:

1. Create `/docs/features/<feature-name>.md` containing:
   - Feature title
   - Implementation date
   - Git diff statistics: `git diff --stat`
   - Files modified/created
   - Testing instructions
   - Expected behavior

2. Provide summary:
```
Build Completed with Codex

Plan: [path to plan]
Feature Doc: /docs/features/<feature-name>.md
Phases Completed: [X of Y]
Files Changed: [count]
Lines Modified: [+additions -deletions]
Codex Sessions: [number of Codex interactions]
Tests Status: [pass/fail/not run]
```

## Error Handling

If Codex encounters issues:
- Review error messages
- Adjust prompt or configuration
- Use mcp__codex__codex-reply to provide corrections
- Fall back to manual implementation if needed