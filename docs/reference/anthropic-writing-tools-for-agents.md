# Writing Effective Tools for AI Agents—Using AI Agents

**Source:** https://www.anthropic.com/engineering/writing-tools-for-agents
**Author:** Ken Aizawa (with contributions from multiple Anthropic teams)
**Published:** September 11, 2025
**Fetched:** 2025-09-19

## Overview

This engineering blog post explores strategies for developing high-quality tools for AI agents, emphasizing an iterative, evaluation-driven approach that treats AI agents as collaborative partners in the development process.

## What is a Tool?

Tools represent a new category of software that bridges deterministic and non-deterministic systems. Unlike traditional software designed for human users, tools must be specifically crafted for AI agents' unique capabilities and interaction patterns.

Key characteristics of AI agent tools:
- Interface between structured, deterministic functions and flexible AI reasoning
- Must account for AI agents' different interaction patterns compared to humans
- Require careful design consideration for optimal performance

## How to Write Tools

The recommended process follows three main phases:

### 1. Build a Prototype
Start with a basic implementation to establish core functionality and understand the tool's role in the agent workflow.

### 2. Run Comprehensive Evaluations
Conduct thorough testing to identify performance bottlenecks, edge cases, and areas for improvement.

### 3. Collaborate with AI Agents
Use AI agents as partners in the development process to refine and optimize tool design.

## Principles for Writing Effective Tools

### Choose Tools Carefully and Purposefully
- Build few, targeted tools that solve specific workflows
- Avoid creating tools for every possible function
- Focus on high-impact capabilities that enhance agent performance

### Use Clear Namespacing
- Implement logical organization for tool functions
- Group related capabilities under coherent namespaces
- Make tool discovery and usage intuitive for agents

### Return Meaningful, Contextually Relevant Information
- Provide information that directly supports the agent's current task
- Filter out irrelevant data that could confuse decision-making
- Structure responses to highlight the most important details

### Optimize for Token Efficiency
- Minimize token usage in tool responses
- Implement response format options for different verbosity levels
- Balance information completeness with computational efficiency

Example ResponseFormat implementation:
```typescript
enum ResponseFormat {
  DETAILED = "detailed",
  CONCISE = "concise"
}
```

### Carefully Engineer Tool Descriptions
- Write clear, actionable descriptions that help agents understand tool capabilities
- Include relevant context about when and how to use each tool
- Provide examples of appropriate use cases

## Best Practices

### Consolidate Functionality
- Combine related functions into unified tools where possible
- Reduce the cognitive load of managing multiple similar tools
- Create coherent interfaces for related operations

### Use Natural Language Identifiers
- Prefer descriptive names over cryptic codes or abbreviations
- Make tool functions self-documenting through clear naming
- Enable agents to understand tool purposes without extensive documentation

### Implement Flexible Response Formats
- Provide options for different levels of detail in responses
- Allow agents to request the appropriate amount of information for their context
- Support both comprehensive and summary response modes

### Create Clear Error Messages
- Provide actionable feedback when tools encounter issues
- Help agents understand what went wrong and how to correct it
- Include suggestions for alternative approaches when appropriate

### Limit Tool Response Tokens
- Set reasonable bounds on response length to maintain efficiency
- Implement truncation strategies that preserve the most important information
- Monitor token usage and optimize accordingly

## Development Approach

The article emphasizes treating AI agents as collaborative partners in tool development rather than passive users. This approach involves:

- Iterative refinement based on agent feedback and performance
- Evaluation-driven development cycles
- Continuous optimization based on real-world usage patterns
- Recognition that tool design for AI agents requires different considerations than traditional software

## Key Takeaways

1. **Purpose-driven design**: Every tool should solve a specific, well-defined problem
2. **Agent-first thinking**: Design tools specifically for AI agent capabilities and limitations
3. **Iterative improvement**: Use evaluation and agent collaboration to continuously refine tools
4. **Efficiency focus**: Optimize for token usage and response relevance
5. **Clear communication**: Invest in tool descriptions and error messaging that agents can easily understand

This approach represents a fundamental shift in software development, requiring developers to think about how AI agents interact with and benefit from tools in ways that may differ significantly from human usage patterns.