---
description:  Fix all review issues in parallel subtree tasks and merge them together 
argument-hint: [review issues] 
allowed-tools: Read, Write, Bash
---

# Fix Review Issues

Follow the workflow to create parallel tasks in git subtrees to Fix the code review issues 

## VARIABLES 

ISSUE_LIST = $ARGUMENTS

## WORKFLOW

1. Parse each issue from `ISSUE_LIST` into a seperate issue
2. Organize the issues so they can be parallelized with minimal conflicts when merging, or group them if dependant.
3. For each issue or set of dependant issues, use the task tool in parallel and create a prompt for each subagent task that understands the rules in AGENTS.md and understands everything it needs to complete its work given a fresh context. Each task should commit its changes and ONLY its changes, but not push The task should report its work back to you so you can summarize all its work in a final report.
4. When complete, merge all changes back to this branch
5. Run tests and make sure everything passes.
6. Report in a list of all changes made. 