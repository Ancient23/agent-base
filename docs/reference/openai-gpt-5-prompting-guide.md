# GPT-5 Prompting Guide

**Source URL:** https://github.com/openai/openai-cookbook/blob/main/examples/gpt-5/gpt-5_prompting_guide.ipynb
**Fetch Date:** 2025-09-19
**Description:** Comprehensive guide for effective prompting with OpenAI's GPT-5 model

## Introduction

GPT-5 represents a significant advancement in AI capabilities, with notable improvements in:
- **Agentic task performance** - Better autonomous decision-making and task execution
- **Coding abilities** - Enhanced programming and software development capabilities
- **Raw intelligence** - Improved reasoning and problem-solving across domains
- **Steerability** - Better response to user guidance and control

This guide provides practical strategies for maximizing GPT-5's potential through effective prompting techniques.

## 1. Agentic Workflow Predictability

### Controlling Agentic Eagerness

GPT-5 can operate across a spectrum of autonomy, from highly proactive to tightly controlled. The key is calibrating the model's "eagerness" to match your specific use case.

#### Strategies for Reducing Agentic Eagerness

When you need more conservative, controlled behavior:

- **Lower `reasoning_effort`** - Reduces the model's tendency to overthink or explore extensively
- **Define clear context gathering criteria** - Specify exactly what information the model should collect before acting
- **Set fixed tool call budgets** - Limit the number of tool calls or actions the model can take
- **Provide "escape hatch" instructions** - Give the model clear conditions under which it should stop and ask for guidance

Example prompt structure:
```
Before taking any actions, gather only the following specific information: [list criteria]
Limit yourself to maximum 3 tool calls before reporting back
If you encounter any ambiguity or uncertainty, stop and ask for clarification rather than proceeding
```

#### Strategies for Increasing Agentic Eagerness

When you want the model to be more proactive and persistent:

- **Increase `reasoning_effort`** - Encourages deeper analysis and more comprehensive exploration
- **Use persistence-focused prompts** - Encourage the model to work through obstacles
- **Clearly define task completion conditions** - Specify exactly when the task is finished
- **Outline safe vs. unsafe actions** - Define boundaries within which the model can operate freely

Example prompt structure:
```
Work persistently through any obstacles you encounter
Continue until you have fully completed [specific completion criteria]
You may freely use any tools and make multiple attempts
Only stop when you have achieved the complete objective
```

### Tool Preambles

Effective communication during tool usage is crucial for transparency and control.

**Recommended Structure:**
```
Always begin by rephrasing the user's goal in a friendly, clear, and concise manner before calling any tools.

For each step:
1. Explain what you're about to do and why
2. Execute the action
3. Summarize what was accomplished
4. State what comes next
```

This approach provides:
- **Transparency** - User understands what's happening at each step
- **Control** - Opportunities to intervene or provide guidance
- **Context** - Maintains awareness of progress and next steps

## 2. Coding Performance Optimization

### Frontend Development Best Practices

#### Recommended Technology Stack

**Core Frameworks:**
- **Next.js with TypeScript** - Primary recommendation for modern web applications
- **React** - For component-based development
- **HTML** - For simpler projects or specific requirements

**Styling and UI:**
- **Tailwind CSS** - Utility-first CSS framework
- **shadcn/ui** - High-quality component library
- **Icons:** Material Symbols, Heroicons
- **Animation:** Motion (Framer Motion)
- **Typography:** Inter, Geist, Mona Sans fonts

#### Zero-to-One App Generation Strategy

When building applications from scratch, use this self-reflection approach:

```
Before writing any code, create an internal excellence rubric for this application:
1. What makes this type of app excellent?
2. What are the key user experience principles?
3. What technical standards should be maintained?

Then evaluate your initial approach against this rubric and iterate as needed.
```

**Benefits of this approach:**
- Forces deeper analysis of requirements
- Establishes quality standards upfront
- Encourages iterative improvement
- Results in more thoughtful architecture

### General Coding Principles

- **Adhere to existing codebase standards** - Maintain consistency with established patterns
- **Implement modular, reusable design** - Build components that can be easily extended
- **Focus on clarity and maintainability** - Write code that others can understand and modify
- **Use descriptive naming conventions** - Make intent clear through variable and function names

## 3. Intelligence and Instruction Following

### Core Steering Techniques

#### Verbosity Control

Adjust response length and detail level:
```
Be concise and direct - provide only essential information
```
or
```
Provide comprehensive, detailed explanations with examples
```

#### Precision in Instructions

- **Avoid contradictory instructions** - Review prompts for conflicting requirements
- **Be explicit about priorities** - When trade-offs exist, specify what matters most
- **Use structured formatting** - Organize complex instructions with numbered lists or sections
- **Define success criteria** - Be clear about what constitutes completion

#### Minimal Reasoning Strategies

When you want faster responses with less internal deliberation:

- **Provide brief thought process explanations** instead of extensive reasoning
- **Use descriptive tool-calling preambles** to maintain transparency without extensive analysis
- **Disambiguate instructions clearly** to reduce the need for interpretation
- **Implement thorough upfront planning** to minimize mid-task decision-making

Example minimal reasoning prompt:
```
Execute this task efficiently with brief explanations for each step.
Avoid lengthy deliberation - proceed with the most straightforward approach.
Explain your reasoning in 1-2 sentences per decision.
```

## 4. Advanced Techniques

### Metaprompting

Use GPT-5 to optimize its own prompts for better performance:

**Process:**
1. Present your current prompt to GPT-5
2. Ask for specific improvements
3. Request identification of unclear or suboptimal elements
4. Iterate based on suggestions

**Example metaprompt:**
```
Analyze this prompt and suggest specific improvements:
[your current prompt]

Focus on:
- Clarity and precision of instructions
- Potential ambiguities or contradictions
- Missing context or constraints
- Opportunities for better structure

Provide specific recommendations for phrase additions, deletions, or modifications.
```

### Structured Formatting Strategies

#### XML-Style Instruction Blocks

Use clear delimiters for different types of instructions:

```xml
<task>
Your main objective here
</task>

<constraints>
- Limitation 1
- Limitation 2
</constraints>

<context>
Relevant background information
</context>

<output_format>
Expected format for response
</output_format>
```

#### Hierarchical Organization

Structure complex prompts with clear information hierarchy:

```
PRIMARY GOAL: [main objective]

APPROACH:
1. [Step 1 with details]
2. [Step 2 with details]
3. [Step 3 with details]

REQUIREMENTS:
- [Requirement 1]
- [Requirement 2]

SUCCESS CRITERIA:
- [How to know task is complete]
```

## Best Practices Summary

### Do's
- **Be explicit and specific** - Clear instructions yield better results
- **Provide relevant context** - Help the model understand the full picture
- **Use consistent formatting** - Maintain structure across similar tasks
- **Test and iterate** - Refine prompts based on actual performance
- **Leverage the model's strengths** - Take advantage of GPT-5's enhanced capabilities

### Don'ts
- **Avoid contradictory instructions** - Review for conflicting requirements
- **Don't assume implicit understanding** - State requirements explicitly
- **Don't skip context** - Provide necessary background information
- **Don't ignore feedback patterns** - Learn from consistent issues or successes

## Conclusion

GPT-5's enhanced capabilities in agentic performance, coding, and reasoning create new opportunities for sophisticated AI-assisted workflows. Success depends on thoughtful prompt engineering that leverages these strengths while providing appropriate guidance and constraints.

The key is matching your prompting strategy to your specific use case - whether you need conservative, controlled behavior or proactive, autonomous operation. By applying these techniques systematically, you can achieve more predictable, high-quality results across a wide range of applications.

---

*This guide is based on the OpenAI Cookbook examples and represents best practices as of the model's release. Continue experimenting and adapting these techniques to your specific use cases.*