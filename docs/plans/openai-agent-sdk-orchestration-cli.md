# Plan: OpenAI Agent SDK Driven Orchestration CLI

## Context
- Build a terminal-focused orchestration tool that wraps the OpenAI Agent SDK to manage agent definitions, runbooks, and execution flows.
- The CLI should empower developers to set up domain-specific agents, compose multi-step workflows, and observe execution results without relying on a separate UI.
- Leverage existing agent-base repository structure (research → plan → build → document) to keep research, plans, and implementation assets organized.

## Goals
- Provide a discoverable CLI surface for registering, configuring, and invoking OpenAI agents and toolchains.
- Support declarative workflow definitions that map to the Agent SDK orchestration primitives (actions, tool calls, interrupts).
- Enable local development ergonomics: hot reload of agent definitions, environment configuration, dry runs, rich logging and tracing output.
- Offer guardrails for API usage (rate limits, retries, budgeting) and a pathway for integration tests.

## Non-Goals
- Building a graphical UI or web dashboard.
- Implementing custom LLM hosting or alternative model backends beyond OpenAI Agents.
- Providing a full MCP server implementation; focus on orchestrating agents defined via the SDK.

## Assumptions & Constraints
- Primary runtime in Node.js (TypeScript) using the official OpenAI Agent SDK.
- CLI distributed via npm, runnable with `npx` for quick trials.
- Configuration expressed in project-local YAML/JSON files checked into source control.
- Network access is required at runtime; offline mode is limited to validation/linting.

## High-Level Architecture
1. **CLI Layer (Commander / yargs)**: command parsing, help output, interactive prompts.
2. **Agent Registry**: loads agent configurations (YAML/JSON), validates schema, instantiates SDK clients.
3. **Workflow Engine**: translates declarative workflow definitions into Agent SDK calls (start session, send events, handle tool results).
4. **Execution Runtime**: manages run context (API keys, environment variables, tracing, rate limiting).
5. **Observability & DX**: structured logs, optional OpenTelemetry hooks, transcript capture, dry-run analyzers.
6. **Extensibility**: plugin hooks for custom tools, cloud deployment adaptors, and future MCP bridge.

## Implementation Phases

### Phase 0 – Discovery & Foundations
- Audit OpenAI Agent SDK capabilities (sessions, threads, actions, streaming) and document supported features.
- Choose CLI framework (Commander.js vs. yargs) and logging stack (pino/winston).
- Define configuration file format(s) and schema validation approach (zod / JSON schema).
- Deliverable: Technical spike report, foundational decisions recorded in `/docs/research/openai-agent-sdk-cli-foundations.md`.

### Phase 1 – Project Scaffolding
- Initialize TypeScript project with build tooling (tsup/tsup, ESLint, Prettier, Vitest).
- Implement base CLI entry point with version/help commands and environment bootstrap (reads `.env`, validates API key).
- Integrate OpenAI SDK client factory with retry/backoff and API usage budgeting guards.
- Deliverable: `bin/agent-cli` executable, project scripts, CI lint/test workflow skeleton.

### Phase 2 – Agent Registry & Configuration
- Define agent configuration schema (metadata, instructions, tool references, defaults).
- Implement configuration loader with schema validation and helpful error messages.
- Add commands: `agent list`, `agent show`, `agent validate`, `agent init` (scaffold new agent config).
- Deliverable: Config store module, tests covering valid/invalid config scenarios, sample agents under `examples/agents/`.

### Phase 3 – Workflow Definition & Execution
- Design workflow manifest structure (steps, control flow, triggers, tool bindings).
- Build workflow compiler that converts manifests into SDK session flows (start run, emit events, handle interrupts).
- Implement commands: `workflow run <file>`, `workflow dry-run`, `workflow visualize` (outputs step graph / textual tree).
- Deliverable: Workflow engine module with unit tests and snapshot tests for sample workflows.

### Phase 4 – Tooling & Integrations
- Provide scaffolding command `tool init` to register custom tool handlers.
- Support common integrations: shell tool, HTTP fetcher, file read/write with sandbox rules.
- Expose lifecycle hooks for tool execution (before/after) and error handling policies.
- Deliverable: Tool registry, documentation for writing tools, integration tests covering tool execution.

### Phase 5 – Observability & UX Enhancements
- Add structured logging with log levels, JSON/plain output modes, transcript capture to disk.
- Integrate optional OpenTelemetry exporter (config flag) for tracing requests.
- Implement `run status` / `run tail` commands to inspect active or historical executions.
- Deliverable: Logging subsystem, telemetry integration, CLI UX polish (colors, progress indicators, prompts).

### Phase 6 – Documentation, Samples, & Release Readiness
- Write end-to-end tutorial that sets up an example agent + workflow.
- Add reference documentation for all commands under `docs/features/openai-agent-cli.md`.
- Prepare npm packaging, versioning strategy (changesets), and GitHub Actions release workflow.
- Deliverable: Documentation suite, release checklist, initial tagged release candidate.

## Testing Strategy
- Unit tests for config parsers, workflow compiler, tool handlers (Vitest/Jest).
- Integration tests that execute sample workflows against mocked OpenAI SDK (nock/msw).
- Smoke tests hitting the real API in a gated CI job (requires API key secrets).
- CLI snapshot tests for help outputs to prevent regressions.

## Risks & Mitigations
- **SDK evolution**: Track upstream changes; pin SDK version and add nightly compatibility checks.
- **API cost overruns**: Implement budgeting limits and dry-run/simulation modes.
- **Complex workflow debugging**: Provide verbose logging, transcript exports, and `--inspect` flags.
- **Security of custom tools**: Sandboxing policy, explicit allowlists, warn on shell execution.

## Open Questions
- Should workflows support conditional logic/branching out of the gate or defer to later release?
- How to persist run history (local sqlite vs. flat files) for the first milestone?
- Is multi-tenant/team use a requirement (shared config registry) or individual developer focus?
- Do we need a Cloud deployment story (hosted orchestrator) in v1, or is local CLI sufficient?
