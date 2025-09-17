# agent-base

Prompts and utilities for agentic coding environments. This is a template repository that provides structured workflows, specialized agents, and documentation patterns for AI-assisted development.

## Overview

**agent-base** is a meta-development toolkit designed to enhance AI-powered development workflows, particularly for Claude Code interactions. Use this repository as a template to scaffold new projects with built-in AI workflow automation.

### Core Components

1. **Commands** (`.claude/commands/`) - Workflow orchestration scripts that automate complex development processes
2. **Agents** (`.claude/agents/`) - Specialized AI agents for specific technical tasks
3. **Documentation** (`docs/`) - Structured knowledge capture and reference materials
4. **MCP Integration** - Model Context Protocol servers for extended capabilities, not active by default to preserve context

### Environment Variables
1. OPENAI_API_KEY - (Optional - Can also launch codex seperatly first and login) Set this as environment variable to use Codex with a specific API_KEY. 
2. CONTEXT7_API_KEY - (Optional) Context7 works without this but it works better if you have an API Key

### Workflow
Create PRD/Specs -> Gather reference URLs -> generate research docs for features -> generate implementation/test plans from research -> Implement/Test -> Document

#### Research-Plan-Build Lifecycle
1. **Research**: Use reference docs with existing code and patterns to generate research docs
2. **Plan**: Create implementation specifications with detailed steps based on research docs
3. **Build**: Execute plans with comprehensive testing and verification
4. **Document**: Capture all knowledge in structured markdown files

#### Available Commands
- `research [prompt]` - Deep technical investigation using research-specialist
- `plan [requirements] --research [files]` - Create implementation plan with research context
- `quick-plan [requirements]` - Rapid planning without research dependencies
- `build [plan-path]` - Implement plan into codebase
- `build-incremental [plan-path]` - Phased implementation as individual PRs
- `fetch-docs` - Scrape and store documentation locally using docs-scraper agent. Create a README with links in /docs/reference to generate markdown versions from links
- `prime` - Basic codebase understanding and CLAUDE.md generation
- `prime_claude` - Extended codebase analysis for Claude Code improvements

#### Specialized Agents
- **research-specialist** - Deep code analysis, pattern identification, actionable insights
- **docs-scraper** - Documentation extraction and formatting

### Document Organization

All outputs follow structured patterns:
- Research components: `/docs/research/parts/[topic-name].md`
- Research final: `docs/research/[feature-name].md`
- Plans: `/docs/plans/[feature-name].md`
- Features: `/docs/features/[feature-name].md`
- Reference: `/docs/reference/[source]/[doc-name].md`

File naming: kebab-case, descriptive titles

## Using This Template

1. **Create a new repository** from this template
2. **Run the prime command** to prime the context with repository specific information. 
3. **Customize workflows** by modifying or adding commands in `.claude/commands/`
4. **Add specialized agents** in `.claude/agents/` for your domain
5. **Configure MCP servers** in `.mcp.json.*` files for additional tool access

### Integration Points

MCP Servers on-demand to reduce context size. Run these when launching claude to enable MCP server:
`claude --mcp-config <mpc-config-filename>`

- **Upstash Context7**: Long-term memory via `.mcp.json.context7`
- **Playwright**: Browser automation via `.mcp.json.playwright`
- **Codex**: AI code generation via `.mcp.json.codex`
- Additional MCP servers can be configured for extended tool access

### Best Practices
1. Create a PRD/spec and generate important links to up to date docs for all components, features, etc and create /docs/reference/README.md with a list of: <feature name> -> URL, then use the fetch-docs command (prompt) to convert all the references to local markdown files. Run periodically to keep them up to date. Command will replace any doc thats older than 24 hours when run.
2. Pass the PRD to the /research command after reference docs have been generated and let it create a comprehensive research doc 
3. Use the /plan command passing it the PRD and final research doc(s) to generate a phased implementation plan designed to split up work into concise PRs.
4. Use /build-incremental to build out each phase as its own PR.
5. Keep Claude.md small and concise to not bloat context memory and only add things all contexts should know like useful tools or rules of the repository. Use /Prime on fresh contexts to load useful repository information like links or useful data. 

## License

MIT License - Copyright 2025 Dream Awake Solutions
