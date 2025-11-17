---
description: Generate or update a README for a given directory
argument-hint: [DIRECTORY_NAME]
---

# Create Readme

Create or update a README for the [DIRECTORY_NAME]/ directory by utilizing knowledge from 
  docs/features/ and any other relevant documentation in docs/. ALWAYS use the code in 
  [DIRECTORY_NAME]/ as the source of truth.

  The README should include:
  1. Overview - What the directory does and its purpose in the project
  2. Directory structure - Layout of files and subdirectories
  3. Core components - Detailed descriptions of main modules/files
  4. Installation/setup - Prerequisites and setup instructions if applicable
  5. Usage examples - Both basic and advanced usage patterns
  6. Configuration - Environment variables, settings, and options
  7. Key concepts - Important patterns or architectural decisions
  8. Testing - How to test the code in this directory
  9. Troubleshooting - Common issues and solutions
  10. Related documentation - Links to other relevant docs
  11. Quick reference - Command table or summary if applicable
  12. Any extra guidance for agents reviewing the file

  When writing:
  - Be comprehensive but concise
  - Use actual code references (file:line format) where relevant
  - Include practical examples from the codebase
  - Cross-reference other README files where appropriate
  - Verify all information against the actual source code
  - Include both local development and deployed environment usage
  - Add architecture diagrams (ASCII) where helpful
  - Keep tone technical and direct

  If updating rather than creating: 
  - Look for outdated or obsolete information, code references, or doc references and fix update or remove them
  - If there are links to other docs, check the other docs and make sure any back references are updated
  - Look for ways to make existing docs more human readable and concise
  - Look for ways to make the guide more impactful and accurate for agents
