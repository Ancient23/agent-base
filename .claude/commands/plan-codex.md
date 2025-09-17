---
allowed-tools: mcp__codex__codex, mcp__codex__codex-reply, Read, Write
description: Creates implementation plans using Codex MCP for advanced code analysis and generation
argument-hint: [user prompt] [--research <filenames>]
---

# Plan with Codex

Create a detailed phased implementation plan using Codex MCP's advanced capabilities. Leverages Codex for deep code analysis, pattern recognition, and intelligent planning based on the user's requirements and optional research files.

## Variables

USER_PROMPT: $1
RESEARCH_FILES: $2
PLAN_OUTPUT_DIRECTORY: `/docs/plans/`

## Instructions

- Parse the user's requirements from USER_PROMPT
- If RESEARCH_FILES are provided, read and analyze them first
- Use Codex MCP to:
  - Analyze the codebase structure and patterns
  - Generate a comprehensive implementation plan
  - Split work into distinct PR-sized phases
  - Include code examples and technical details
- Save the generated plan to `PLAN_OUTPUT_DIRECTORY/<descriptive-name>.md`

## Workflow

1. **Prepare Context**
   - Read any provided RESEARCH_FILES
   - Extract key requirements from USER_PROMPT

2. **Invoke Codex**
   - Use mcp__codex__codex with a structured prompt that includes:
     - User requirements
     - Research context (if available)
     - Request for phased implementation plan
     - Emphasis on PR-sized work chunks

3. **Process Codex Output**
   - Review and structure the Codex-generated plan
   - Ensure it includes:
     - Clear problem statement
     - Technical approach and architecture
     - Step-by-step implementation per phase
     - Testing strategy per phase
     - Success criteria

4. **Save Plan**
   - Generate descriptive kebab-case filename
   - Write to `PLAN_OUTPUT_DIRECTORY/<filename>.md`

## Codex Prompt Template

When invoking Codex, use this prompt structure:

```
Create a detailed phased implementation plan for the following requirements:

[USER_PROMPT]

[If RESEARCH_FILES exist:]
Based on the following research:
[RESEARCH_CONTENT]

Please provide:
1. Problem statement and objectives
2. Technical approach and architecture decisions
3. Implementation phases (each suitable for a PR):
   - Phase description
   - Files to modify/create
   - Step-by-step implementation
   - Testing approach
   - Success criteria
4. Potential challenges and solutions
5. Code examples where helpful

Format as a structured markdown document.
```

## Report

After creating the plan, provide:

```
Implementation Plan Created with Codex

File: PLAN_OUTPUT_DIRECTORY/<descriptive-name>.md
Topic: <Brief description>
Research Files: <List if any>
Phases: <Number of implementation phases>
Codex Analysis: <Key insights from Codex>
```