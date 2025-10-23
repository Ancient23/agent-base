---
description:  Run full integration tests against deployed systems
argument-hint: [path-to-script]
---

# Test Deployment

Run the workflow and repeat until al issues are fixed and the test runs completly with no issues.

# Variables

INTEGRATION_TEST_SCRIPT = $ARGUMENTS

# Workflow

1. Monitor recent deployment triggered via CI on the main branch of this repository and wait for completion
2. Clear any previously uploaded test videos, dynamodb jobs, locks, database entries, and any other resources created or uploaded during the last integration test run(s).
3. Run the `INTEGRATION_TEST_SCRIPT`. 
4. Monitor Logs on AWS using the AWS CLI, and take note of all issues.
5. Monitor logs in the script, and take note of all issues.
6. If issues were found: 
    - Fix them
    - Redploy by committing changes to the main branch and pushing to origin/main
    - Repeat the Workflow 
7. If no issues were found:
    - Update the PLAN and any related READMEs with any changes made
    - Complete the Workflow and Report

# Report 

Provide a Summary including:
1. Overall number of fixes, commits, deployment
2. a list of issues and fixes in this format:
- <issue description>: <Fix made>: <Any Technical debt remaining>
3. Any AWS resources updated directly with the AWS CLI
4. Anything else that might be important to note 

