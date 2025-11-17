---
description:  Debug and fix a failed AWS deployment triggered via CI
---

# Fix Deployment

Execute the `Workflow`, and `Report` sections to debug and fix a current failed deployment on github CI.

## Workflow

1. Execute the .claude/commands/prime_deploy.md command. 
2. Check recent deployment triggered via CI on the main branch of this repository. If failed, fix any issues, commit, push and monitor deployment. Repeat until deployment succeeds.

## Report 

Generate a final concise report summarizing and listing the issues and fixes