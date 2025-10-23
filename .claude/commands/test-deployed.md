---
description: Run integration tests against deployed systems without cleanup or deployment monitoring
argument-hint: [path-to-script]
---

# Test Deployed

Run integration tests against the deployed system and iterate on fixes if issues are found.

# Variables

INTEGRATION_TEST_SCRIPT = $ARGUMENTS

# Workflow

1. Run the `INTEGRATION_TEST_SCRIPT`
2. Monitor logs in the script output and take note of all issues
3. If AWS logs need to be checked, use AWS CLI to monitor CloudWatch logs
4. If issues were found:
   - Analyze the root cause
   - Fix them in the codebase
   - Redeploy by committing changes to the main branch and pushing to origin/main
   - Wait for deployment to complete
   - Repeat the Workflow
5. If no issues were found:
   - Update the PLAN and any related READMEs with any changes made
   - Complete the Workflow and Report

# Report

Provide a Summary including:
1. Overall test result (passed/failed)
2. Number of fixes, commits, and deployments made
3. A list of issues and fixes in this format:
   - <issue description>: <Fix made>: <Any Technical debt remaining>
4. Any AWS resources updated directly with the AWS CLI
5. Any recommendations for future testing or improvements

