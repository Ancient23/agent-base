# Claude Code: Best Practices

**Source:** https://www.anthropic.com/engineering/claude-code-best-practices
**Fetched:** 2025-09-19
**Description:** Official best practices guide for Claude Code from Anthropic Engineering

## Introduction

Claude Code is a command line tool for agentic coding, designed to provide Anthropic engineers and researchers with a native way to integrate Claude into their coding workflows. The tool is intentionally low-level and unopinionated, offering flexible and customizable access to the model without forcing specific workflows.

## 1. Customize Your Setup

### Create CLAUDE.md Files

A `CLAUDE.md` file is automatically pulled into context when starting a conversation. Use it to document:

- Common bash commands
- Core files and utility functions
- Code style guidelines
- Testing instructions
- Repository etiquette
- Developer environment setup
- Unexpected behaviors or warnings
- Other important project information

#### Example CLAUDE.md

```markdown
# Bash commands
- npm run build: Build the project
- npm run typecheck: Run the typechecker

# Code style
- Use ES modules (import/export) syntax, not CommonJS (require)
- Destructure imports when possible

# Workflow
- Typecheck after code changes
- Prefer running single tests for performance
```

#### Placement of CLAUDE.md Files

You can place `CLAUDE.md` files in:
- Root of the repo
- Any parent directory
- Any child directory
- Home folder (`~/.claude/CLAUDE.md`)

### Tune CLAUDE.md Files

- Refine `CLAUDE.md` like any frequently used prompt
- Use the `#` key to quickly add instructions
- Occasionally run files through a prompt improver
- Add emphasis to improve instruction adherence

### Manage Allowed Tools

Claude Code is conservative by default. You can manage tool permissions through:
- Selecting "Always allow" during a session
- Using the `/permissions` command
- Manually editing `.claude/settings.json`
- Using the `--allowedTools` CLI flag

## 2. Give Claude More Tools

### Use Bash Tools

- Claude inherits your bash environment
- Provide instructions for custom tools
- Tell Claude to use `--help` for documentation
- Document frequently used tools in `CLAUDE.md`

### Use MCP (Model Context Protocol)

Claude Code can connect to MCP servers via:
- Project config
- Global config
- Checked-in `.mcp.json` files

This allows Claude to access additional capabilities like databases, APIs, and custom tools.

### Create Custom Slash Commands

For repeated workflows, create custom slash commands that can encapsulate complex operations and make them easily reusable.

## 3. Common Workflows

### Explore, Plan, Code, Commit

1. **Explore**: Read relevant files to understand the codebase
2. **Plan**: Create a detailed implementation plan
3. **Code**: Implement the solution step by step
4. **Commit**: Commit changes with descriptive messages

### Test-Driven Development

1. Write tests first
2. Confirm tests fail as expected
3. Implement code to make tests pass
4. Commit incrementally
5. Refactor if needed

### Visual Design Iteration

1. Take screenshots of current state
2. Provide visual mockups or design references
3. Implement design changes
4. Take new screenshots to compare
5. Iterate until satisfied
6. Commit final implementation

## 4. Workflow Optimization Strategies

### Be Specific in Instructions

- Provide clear, detailed requirements
- Specify exact file paths and function names
- Include context about expected behavior
- Mention edge cases and error handling

### Use Images and Visual References

- Take screenshots for UI work
- Provide mockups and design references
- Use visual comparisons to guide implementation
- Include before/after examples

### Mention Specific Files

- Reference exact file paths
- Point to relevant functions and classes
- Include line numbers when helpful
- Provide context about file relationships

### Course-Correct Early

- Monitor Claude's progress closely
- Provide feedback and corrections promptly
- Redirect if the approach isn't working
- Ask clarifying questions when needed

### Use `/clear` to Maintain Context Focus

- Clear context when switching tasks
- Remove irrelevant information
- Keep conversations focused on current objectives
- Start fresh when appropriate

### Leverage Checklists and Scratchpads

- Create task lists for complex work
- Track progress systematically
- Use scratchpads for planning and notes
- Break down large tasks into smaller steps

## 5. Advanced Techniques

### Multi-Claude Workflows

- Use multiple Claude instances for parallel work
- Divide complex tasks across sessions
- Coordinate between different workstreams
- Merge results systematically

### Parallel Task Processing

- Handle independent tasks simultaneously
- Use background processes effectively
- Monitor multiple operations
- Synchronize results when needed

### Headless Mode Automation

- Automate repetitive tasks
- Create scripted workflows
- Run operations without interaction
- Integrate with CI/CD pipelines

### Git Worktree Management

- Use git worktrees for parallel development
- Manage multiple branches simultaneously
- Isolate experimental changes
- Coordinate across different features

## Safety Recommendations

### Permission Management

- Use `--dangerously-skip-permissions` cautiously
- Review tool permissions regularly
- Understand security implications
- Maintain least-privilege access

### Environment Isolation

- Prefer containerized environments
- Use virtual environments when possible
- Isolate development from production
- Maintain clean separation of concerns

### Oversight and Monitoring

- Maintain careful oversight of Claude's actions
- Review changes before committing
- Test thoroughly before deployment
- Monitor for unexpected behaviors

## Best Practices Summary

1. **Start with good setup**: Create comprehensive `CLAUDE.md` files
2. **Provide tools**: Give Claude access to the tools it needs
3. **Be specific**: Clear instructions lead to better results
4. **Use visuals**: Screenshots and mockups improve understanding
5. **Iterate quickly**: Course-correct early and often
6. **Stay organized**: Use checklists and clear workflows
7. **Experiment**: Find workflows that suit your specific needs
8. **Stay safe**: Maintain oversight and use appropriate permissions

## Tools and Integrations

### Recommended Tools

- **GitHub CLI (`gh`)**: For repository management, issues, and pull requests
- **Docker**: For containerized development environments
- **ripgrep (`rg`)**: For fast code searching
- **MCP servers**: For extended functionality and integrations

### Useful MCP Servers

- Database connectors for direct database access
- API clients for external service integration
- Development tool integrations
- Custom business logic servers

## Tips for Effective Usage

### Communication Best Practices

- Be explicit about your goals and constraints
- Provide context about the broader project
- Share relevant examples from your codebase
- Ask for explanations when Claude's approach is unclear

### Workflow Optimization

- Start conversations with clear objectives
- Use structured task breakdown for complex work
- Leverage Claude's ability to read and understand large codebases
- Integrate testing and validation into your workflow

### Troubleshooting Common Issues

- If Claude is making unexpected changes, provide more specific guidance
- Use screenshots for visual debugging
- Reference existing patterns in your codebase for consistency
- Break down complex tasks into smaller, manageable pieces

## Conclusion

The article emphasizes that Claude Code is a research project designed for experimentation. Users are encouraged to find personalized workflows that suit their individual and team needs while maintaining appropriate safety measures.

The key to success with Claude Code is thoughtful customization, clear communication, and iterative improvement of your workflows. Start with basic usage patterns and gradually incorporate advanced techniques as you become more familiar with the tool's capabilities.