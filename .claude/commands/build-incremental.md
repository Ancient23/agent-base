---
description:  Build the codebase incrementally in individual PRs per phase based on a phased plan
argument-hint: [path-to-plan]
allowed-tools: Read, Write, Bash
---

# Build

Follow the `Workflow` to implement the next phase in the `PATH_TO_PLAN` as an indivdual concise PR.

## Variables

PATH_TO_PLAN: $ARGUMENTS

## Workflow

- If no `PATH_TO_PLAN` is provided, STOP immediately and ask the user to provide it.
- Locate the next unfinished phase in the `PATH_TO_PLAN`, follow the `implement_pr_prompt` as the exact prompt for this phase:
    <implement_pr_prompt>
        1. Think hard about the instructions for this phase from the plan, create tests and implement the feature or fix using a test-first approach into the codebase 
        2. Create a `Report` of the completed work 
        3. Update the status of the task in the PLAN, add date modified
        4. Provide a summary with all files changed and location of the report file created for a commit message and commit the changes
        5. Summarize the changes and create a draft PR
    </implement_pr_prompt>

## Report 

- When adding a new feature or bug fix, create an accompanying markdown file in `/docs/features/<feature-name>.md`
    1. Title of the feature
    2. Date
    3. Report the files and total lines changes with `git diff --stat`
    4. List screens to load
    5. User actions used to test the PR
    6. Expected results.
 
