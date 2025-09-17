---
description:  Build the codebase based on the plan
argument-hint: [path-to-plan]
allowed-tools: Read, Write, Bash
---

# Build

Follow the `Workflow` to implement the `PATH_TO_PLAN` then `Report` the completed work.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

- If no `PATH_TO_PLAN` is provided, STOP immediately and ask the user to provide it.
- Read the plan at `PATH_TO_PLAN`. Think hard about the plan and implement it into the codebase. 

## Report 

- When adding a new feature or bug fix, create an accompanying markdown file in `/docs/features/<feature-name>.md`
    1. Title of the feature
    2. Date
    3. Report the files and total lines changes with `git diff --stat`
    4. List screens to load
    5. User actions used to test the PR
    6. Expected results.
 
