---
description:  Load previosly unfetched documentation from their respective websites into local markdown files our agents can use as context.
---

# Fetch Docs

Load documentation from their respective websites into local markdown files our agents can use as context.

## Variables

URL_REFERENCE_DOC: `/docs/reference/README.md`

## Workflow

1. Read the `URL_REFERENCE_DOC` file
2. See if any reference/<some_filename>.md already exists
    - If it does, skip it
3. For each url in `URL_REFERENCE_DOC` that was not skipped, Use the task tool in parallel and follow the `scrape_loop_prompt` as the exact prompt for each task
    <scrape_loop_prompt>
        Use @docs-scraper agent - pass it the url as the prompt
    </scrape_loop_prompt>
4. After all Tasks are complete, respond in `Report Format`

## Report Format

```
Docs Report
- <Success or Failure>: <url> - <markdown file path>
- ...
```