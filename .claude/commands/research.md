---
description:  Research the given prompt and produce research docs which understand how the system works, finds all relevent files and links to them, and potentially explore causes of a bug if specified.
---

# Research

Research the given prompt and produce research docs which understand how the system works, finds all relevent files and links to them, and potentially explore causes of a bug if specified.

## Variables

REFERENCE_DIR: `/docs/reference/`

## Workflow

1. Break down user's query into composable research areas
2. Take time to ultrathink about the underlying patterns, connections, and architectural implications the user might be seeking.
3. Identify specific components, patterns, or concepts to investigate.
4. Read the `REFERENCE_DIR` directory and find relevant reference docs or review any docs provided.
5. Create a research plan using TodoWrite to track all subtasks.
6. Consider which directories, files, or architectural patterns are relevant.
7. For each subtask, Use the task tool in parallel and follow the `research_specialist_prompt` as the exact prompt for each task
    <research_specialist_prompt>
        Use @research-specialist agent to research the specific area providing context and any specific reference docs identified in the previous step. The sub-agent should output its research to the /docs/research/parts/ directory and provide the file name when complete. 
    </research_specialist_prompt>
8. After all tasks are complete, review all research docs produced by the specialist agents and produce a final research doc in the /research directory.
7. After all Tasks are complete, respond in `Report Format`

## Report Format

```
Docs Report
- Final doc produced: <filename>.md
- individual research areas: 
-- <Success or Failure>: <task> - <markdown file path>
- ...
```