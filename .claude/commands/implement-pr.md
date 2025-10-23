---
description: Implement a PR from the approved plan
argument-hint: [PR-number]
---

# Implement PR

You are an expert coding agent working in /Users/filip/Projects/seer-agent. Apply the repository guidelines and implementation rules from AGENTS.md and CLAUDE.md. Then follow the workflow and final response format.

## Variables

PR_NUMBER: $1

## Workflow

  1. **Review and Plan**
     - Review `docs/plans/PLAN.md` and find the PR number from `PR_NUMBER` variable
     - Read any relevant items under `docs/research/` and `docs/reference/` or any other docs the PR points to
     - Determine architecture approach:
       - **Backend (Python):**
         - New standalone service? → FastAPI app in `api/<service>/app.py`
         - Enhancement to existing service? → Router integrated into existing app (`api/tools/` or `api/chat/`)
         - Lambda worker? → New file in `services/<domain>/`
         - Shared utility? → Add to `shared/`
       - **Frontend (TypeScript):**
         - New page? → `web/app/<route>/page.tsx`
         - New component? → `web/components/` or `web/app/components/`
         - API client change? → `web/lib/api.ts`
       - **Infrastructure (CDK):**
         - New stack? → `infra/lib/<name>-stack.ts`
         - Stack modification? → Update existing stack in `infra/lib/`
     - Check existing patterns (API routes, component structure, CDK constructs) for consistency

  2. **Backend Implementation** (if PR involves Python code)
     - Implement every bullet under the PR's Scope/Additional Components/Tests with complete code (no stubs)
     - Extract reusable logic to `shared/`:
       - Helper functions used across multiple endpoints → `shared/<domain>_utils.py`
       - Cross-service data models → `shared/models.py` or `shared/<domain>_schemas.py`
       - Client utilities (filters, queries) → enhance existing `shared/*_client.py`
     - Create constants in appropriate location:
       - Service-specific → `api/<service>/constants.py` or `services/<domain>/constants.py`
       - Cross-service → `shared/models.py` (status constants, etc.)
     - Reuse existing interfaces and patterns from the codebase
     - Add logging with various log levels at important places for proper visibility:
       - Log every exception with full details (exception type, message, traceback)
       - Log every successful response before return
       - Use structured logging with context (videoId, jobId, traceId)
     - Add docstrings with examples for all shared utilities
     - **ECS Deployment Changes:**
       - Update `Dockerfile` if adding system dependencies
       - Test locally: `docker build -t test .`
     - **Lambda Deployment Changes:**
       - Update `lambda-requirements.txt` if adding minimal dependencies
       - Keep Lambda functions small (size limits apply)

  3. **Frontend Implementation** (if PR involves web/ changes)
     - Follow Next.js 15 App Router conventions:
       - Server Components by default (use `'use client'` only when needed)
       - File-based routing in `web/app/`
       - API routes in `web/app/api/` (if needed, though backend APIs preferred)
     - Component structure:
       - Shared UI components → `web/components/`
       - Page-specific components → `web/app/<route>/components/`
       - Use Tailwind CSS for styling
     - API integration:
       - Use `ApiClient` from `web/lib/api.ts` for all backend calls
       - Include authentication headers (Bearer token from Descope)
       - Handle loading states and errors
     - Type safety:
       - Define TypeScript interfaces for API responses
       - Use type guards for runtime validation
     - Build verification:
       - Run `npm run build` in `web/` directory
       - Fix all TypeScript errors and warnings

  4. **Infrastructure Changes** (if PR involves CDK)
     - Current stacks (order matters for deployment):
       1. `SeerAgentBaseStack` - Shared resources (secrets, VPC references)
       2. `SeerAgentApiHttpStack` - ECS Fargate + ALB for FastAPI services
       3. `SeerAgentCloudFrontStack` - CloudFront distribution (HTTPS for API)
       4. `SeerAgentIngestionStack` - Lambda workers + S3 + SQS/SNS + DynamoDB
     - Stack modifications:
       - Add new constructs to existing stacks
       - Update environment variables for ECS tasks or Lambda functions
       - Add IAM permissions as needed
       - Update security groups for new service connections
     - Verify synthesis: `make cdk-synth`
     - Test deployment order matches `.github/workflows/deploy-staging.yml:44-49`

  5. **Testing**
     - **Backend tests:**
       - Add unit tests in `tests/` mirroring source structure
       - Mock external dependencies (Bedrock, Qdrant, Neo4j, DynamoDB, S3)
       - Test both success and error cases
       - Validate request/response schemas with invalid payloads
       - Expected: 168+ tests (update count if adding new tests)
     - **Frontend tests (if applicable):**
       - Component tests using React Testing Library
       - Mock API client responses
       - Test user interactions and state changes
     - **Integration tests:**
       - Add to `tests/integration/` if needed
       - Use local mode (LocalStack, Qdrant `:memory:`)
     - Run full test suite: `make test`

  6. **Documentation Updates**
     - **MANDATORY: Update directory READMEs** for any core directory changes:
       - `agents/README.md` - Agent changes
       - `api/README.md` - API endpoint changes
       - `services/README.md` - Lambda worker changes
       - `shared/README.md` - New utilities/clients
       - `infra/README.md` - CDK stack changes
       - `web/README.md` - Frontend changes
       - `tests/README.md` - New testing patterns
       - Include file:line references for key changes
       - Remove outdated information
     - **Create feature test doc:**
       - File: `docs/features/YYYY-MM-DD-<feature-name>.md`
       - Include: Summary, manual verification steps, expected outcomes, implementation details
     - **Update PLAN.md after implementation:**
       - Mark PR as ✅ completed with date
       - Document architecture decisions
       - Note technical debt and follow-up work
       - Link to feature test doc

  7. **Standard Checks**
     - Run all checks (MANDATORY before commit):
       ```bash
       make dev && make lint && make typecheck && make test && make docs && make cdk-synth
       ```
     - **Web checks** (if web/ changed):
       ```bash
       cd web && npm run build && npm run lint
       ```
     - Fix all failures before proceeding
     - Verify builds:
       - Docker (if Dockerfile changed): `docker build -t test .`
       - CDK synthesis must succeed
       - Web build must succeed (no TypeScript errors)

  8. **Git Workflow**
     - Branch naming: `feature/pr-<N>-<short-name>` or `fix/pr-<N>-<short-name>`
     - Commit message format (follow existing conventions):
       ```
       <type>(<scope>): <subject>

       <body with file:line references>
       ```
       Examples:
       - `feat(api): add video deletion endpoint (PR #X)`
       - `fix(web): resolve chat SSE crash on Safari (PR #X)`
       - `infra(cdk): add CloudFront stack for HTTPS (PR #X)`
     - Stage all relevant files (backend, frontend, infra, tests, docs)
     - Do NOT commit until all checks pass

  9. **Prepare PR Description**
     - Use this template:
       ```markdown
       ## Summary
       <Brief description of changes with key file:line references>

       ## Implementation Details
       ### Backend (if applicable)
       - Changes in `api/`, `services/`, `shared/`
       - File references: `path/to/file.py:123-145`

       ### Frontend (if applicable)
       - Changes in `web/app/`, `web/components/`, `web/lib/`
       - File references: `path/to/file.tsx:67-89`

       ### Infrastructure (if applicable)
       - CDK stack changes
       - New resources: <list>
       - Updated IAM permissions: <list>

       ## Architecture Decisions
       - <Decision 1>: <Rationale>
       - <Decision 2>: <Rationale>

       ## Testing
       - Unit tests: <count> new tests added in `tests/<path>/`
       - Integration tests: <if applicable>
       - Manual verification: See `docs/features/YYYY-MM-DD-<feature>.md`

       **Test Results:**
       ```
       <Summarized pytest output>
       ```

       **Web Build Results** (if applicable):
       ```
       <Summarized npm build output>
       ```

       ## Deployment Considerations
       - Stack deployment order: <if changed>
       - Environment variables: <if new vars added>
       - Database migrations: <if applicable>
       - Rollback plan: <describe>

       ## Technical Debt / Follow-up
       - <Item 1>
       - <Item 2>

       ## Checklist
       - [ ] All standard checks pass (`make dev && make lint && make typecheck && make test && make docs && make cdk-synth`)
       - [ ] Web build passes (if applicable)
       - [ ] READMEs updated for affected directories
       - [ ] Feature test doc created
       - [ ] PLAN.md will be updated after merge
       ```

## Final Response Format

Provide a concise summary with:
  - **PR Number:** <number>
  - **Implementation recap:** Key changes with file:line citations
  - **Architecture decisions:** Standalone vs integrated, schema locations, stack choices
  - **Components modified:**
    - Backend: <files>
    - Frontend: <files> (if applicable)
    - Infrastructure: <stacks> (if applicable)
    - Tests: <files>
    - Docs: <files>
  - **Test results:** Summarized output (pass/fail counts, any failures)
  - **Web build results:** (if applicable)
  - **Git branch:** <branch-name>
  - **Commit message:** <exact message used>
  - **Ready for PR?** Yes/No (if No, list blockers)
  - **Follow-up work:** Technical debt or risks to document