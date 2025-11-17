---
description: Creates a test-driven development implementation plan based on references (Postman collections, markdown docs) with research-backed recommendations
argument-hint: [user prompt] [--refs <file1,file2,...>]
---

# TDD Plan

Create a comprehensive test-driven development implementation plan that leverages research-specialist agents to understand the current codebase state, identifies shared utilities to reuse, and generates phased implementation steps suitable for small pull requests. Follows all repository rules in AGENTS.md and CLAUDE.md.

## Variables

USER_PROMPT: $1
REFERENCE_FILES: $2
RESEARCH_OUTPUT_DIR: `/docs/research/parts/`
PLAN_OUTPUT_DIRECTORY: `/docs/plans/`
SHARED_DIR: `/shared/`
AGENTS_RULES: `/AGENTS.md`

## Workflow

### Phase 1: Parse and Prepare

1. Parse USER_PROMPT to understand the feature requirements
2. If REFERENCE_FILES are provided (e.g., Postman collections, markdown docs):
   - Read each reference file
   - Extract API endpoints, data structures, and requirements
   - Identify DTOs, models, and types that will be needed
3. If NO REFERENCE_FILES are provided, STOP and ask the user to provide them
4. Read AGENTS_RULES to understand repository coding standards and patterns

### Phase 2: Research Current Implementation

5. Use TodoWrite to create a research plan with the following subtasks:
   - Understand existing API patterns and structure
   - Identify relevant models and DTOs in shared/src/models/
   - Find reusable utilities in shared/src/utils/
   - Review validation patterns in shared/src/validations/
   - Understand error handling patterns (ServiceError implementations)
   - Identify test patterns for similar features
   - Review client/server module boundary patterns

6. For each research subtask, launch Task tool in PARALLEL with subagent_type=research-specialist:
   - Each specialist should output to RESEARCH_OUTPUT_DIR
   - Provide specific context from REFERENCE_FILES to each specialist
   - Request specific findings about:
     * Existing code to reuse (no duplication)
     * Similar patterns already implemented
     * Potential conflicts or breaking changes
     * Test utilities and fixtures available

7. Wait for all research specialists to complete and read their output files

### Phase 3: Analyze Shared Code

8. Review shared/ directory structure:
   - shared/src/models/ - Identify existing DTOs and models
   - shared/src/utils/ - Find reusable utility functions
   - shared/src/validations/ - Check for validation helpers
   - shared/src/errors/ - Review error types (ServiceError pattern)
   - shared/src/constants/ - Check for constants vs magic numbers

9. Create a "Reusable Components" section listing:
   - Models/DTOs that already exist
   - Utilities that can be leveraged
   - Validation functions available
   - Constants that should be used

### Phase 4: Generate TDD Plan

10. Structure the implementation plan with TEST-FIRST approach:

    **For each phase:**
    - Phase N: [Feature Name]
      * Test Files to Create/Update (WRITE THESE FIRST)
        - Unit tests for models/DTOs
        - Unit tests for service layer
        - Unit tests for API endpoints
        - Integration tests if applicable
      * Implementation Files
        - DTOs in shared/src/models/ (following DTO pattern from AGENTS.md)
        - Service layer with ServiceError implementations
        - API endpoints
        - Client components (if needed)
      * Validation
        - Run build, lint, typecheck and tests
        - All tests must pass
        - No type errors allowed
      * Success Criteria
        - Specific test assertions
        - API behavior verification
        - Error handling validation

11. Ensure each phase follows AGENTS.md rules:
    - Use DTOs from shared/src/models/ (not inline types)
    - Implement ServiceError interface for errors
    - Follow client/server boundary patterns (*.types.ts, *.server.ts, *.client.ts)
    - Use constants instead of magic numbers
    - No `any` types
    - POST endpoints return 201 with Location header
    - API calls wrapped in try/catch on client
    - Use useApi hook for API calls
    - Create test documentation in docs/features/

12. Break plan into small phases:
    - Each phase = 1 small PR (easier code review)
    - Each phase is independently testable
    - Each phase adds incremental value
    - Phases build on each other logically

### Phase 5: Document and Save

13. Generate plan document with sections:
    ```markdown
    # [Feature Name] - TDD Implementation Plan

    ## Overview
    - Problem statement
    - Reference files analyzed
    - Key requirements from references

    ## Research Findings
    - Link to research files in RESEARCH_OUTPUT_DIR
    - Summary of current implementation state
    - Identified conflicts or concerns

    ## Shared Code Reuse
    - Existing DTOs/models to use
    - Utilities to leverage
    - Validation helpers available
    - Constants to reference

    ## Repository Rules Checklist
    - [ ] DTOs in shared/src/models/
    - [ ] ServiceError implementations
    - [ ] No `any` types
    - [ ] Constants instead of magic numbers
    - [ ] Client/server boundaries (*.types.ts, *.server.ts)
    - [ ] Unit tests for all layers
    - [ ] API error handling
    - [ ] Documentation in docs/features/

    ## Implementation Phases

    ### Phase 1: [Name]
    **PR Size:** Small | **Estimated Effort:** [Low/Medium/High]

    #### Tests (Write First)
    1. Create test file: path/to/test.spec.ts
       - Test case: should...
       - Test case: should...

    #### Implementation
    1. Create DTO: shared/src/models/Foo.types.ts
    2. Implement service: path/to/foo.server.ts
    3. Create endpoint: path/to/route.ts

    #### Validation
    - Run: `yarn build && yarn test`
    - Verify: All tests pass
    - Check: No type errors

    #### Success Criteria
    - Specific acceptance criteria

    ### Phase 2: [Name]
    ...

    ## Testing Strategy
    - Unit test approach
    - Integration test approach
    - Manual testing steps (docs/features/)

    ## Potential Challenges
    - Challenge 1 and mitigation
    - Challenge 2 and mitigation

    ## References
    - Links to Postman collections
    - Links to markdown docs
    - Links to research files
    ```

14. Generate descriptive kebab-case filename
15. Save to PLAN_OUTPUT_DIRECTORY/<filename>.md

### Phase 6: Report

16. Provide final report in specified format

## Report Format

```
TDD Implementation Plan Created

File: PLAN_OUTPUT_DIRECTORY/<descriptive-name>.md
Topic: <Brief description>
Reference Files: <List of Postman/markdown files analyzed>
Research Files: <List of research outputs from specialists>
Phases: <Number of implementation phases>

Key Findings:
- Shared code to reuse: <summary>
- New DTOs needed: <count>
- New tests needed: <count>
- Repository rules verified: <yes/no>

Research Specialists Used:
- <research area 1>: <output file>
- <research area 2>: <output file>
- ...
```

## Example Usage

```bash
# With Postman collection
/tdd-plan "Implement API" --refs "docs/plans/API.postman_collection.json"

# With multiple references
/tdd-plan "Add user notifications" --refs "docs/api-spec.md,docs/requirements.md"

# With research docs
/tdd-plan "Refactor payment flow" --refs "docs/research/payment-system.md"
```

## Key Principles

1. **Test-First**: Always write tests before implementation
2. **No Duplication**: Reuse shared code aggressively
3. **Small PRs**: Each phase should be independently reviewable
4. **Type Safety**: No `any` types, proper DTOs
5. **Error Handling**: ServiceError pattern throughout
6. **Standards Compliance**: Follow all AGENTS.md rules
7. **Research-Backed**: Use specialists to understand current state
