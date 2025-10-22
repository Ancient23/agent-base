---
description:  Trace or track the flow of a datatype, interface, variable, etc throughout the codebase
argument-hint: [datatype]
---

# trace-flow

Run the workflow with the variable provided then report output

## Variables

ITEM_TO_TRACK: $1

## Workflow 

Show me how `ITEM_TO_TRACK` flows throughout the application. Detail functions, files and purpose. Summarize the overall flow.

## Report 

1. Bulleted list of: 
`<file>: <function>:<purpose>\n<file>: <function>...`

2. Overall flow summary:
`<function> -> <function> -> <database>`