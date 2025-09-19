# GPT-5 Prompt Optimization Cookbook

**Source URL:** https://cookbook.openai.com/examples/gpt-5/prompt-optimization-cookbook
**Fetch Date:** 2025-09-19
**Content:** OpenAI GPT-5 prompt engineering and optimization techniques

## Overview

This cookbook provides comprehensive guidance on optimizing prompts for GPT-5, demonstrating systematic approaches to improve AI model performance through two detailed case studies: coding tasks and financial question answering.

## Key Optimization Principles

### Systematic Approach
- Identify baseline prompt limitations
- Use OpenAI's Prompt Optimizer tool
- Apply iterative improvements
- Evaluate results quantitatively and qualitatively

### Core Optimization Techniques
- Remove prompt contradictions
- Specify exact output requirements
- Create structured decision frameworks
- Minimize model uncertainty
- Tighten constraints and reduce ambiguity

## Case Study 1: Coding Task Optimization (Top-K Frequent Tokens)

### Baseline Prompt Issues
- Contained contradictory instructions
- Allowed ambiguous implementation choices
- Risked variable performance across runs

### Baseline Prompt Example
```python
baseline_prompt = """
Write Python to solve the task on a MacBook Pro (M4 Max). Keep it fast and lightweight.

- Prefer the standard library; use external packages if they make things simpler.
- Stream input in one pass to keep memory low; reread or cache if that makes the solution clearer.
- Aim for exact results; approximate methods are fine when they don't change the outcome in practice.
- Avoid global state; expose a convenient global like top_k so it's easy to check.
- Keep comments minimal; add brief explanations where helpful.
- Sort results in a natural, human-friendly way; follow strict tie rules when applicable.

Output only a single self-contained Python script inside one Python code block, with all imports, ready to run.
"""
```

### Optimized Prompt
```python
optimized_prompt = """
# Objective
Generate a single, self-contained Python script that exactly solves the specified task on a MacBook Pro (M4 Max).

# Hard requirements
- Use only Python stdlib. No approximate algorithms.
- Tokenization: ASCII [a-z0-9]+ on the original text; match case-insensitively and lowercase tokens individually.
- Exact Top‑K semantics: sort by count desc, then token asc.
- Define `top_k` as a list of (token, count) tuples with length = min(k, number of unique tokens).
- Performance & memory constraints
- Do NOT materialize the entire token stream or any large intermediate list.
- Target peak additional memory beyond the counts dict to O(k).

# Guidance
- Build counts via a generator over re.finditer
- Prefer heapq.nsmallest for exact selection without full sort
- Keep comments minimal; include complexity analysis
"""
```

### Optimization Results
- **Runtime improvement:** -0.929s average reduction
- **Memory reduction:** -3048.8 KB significant decrease
- **Code quality improvement:** +0.16 increase
- **LLM adherence improvement:** +0.50 increase

### Key Improvements Made
1. **Explicit constraints:** Removed contradictory guidance about external packages vs standard library
2. **Precise tokenization rules:** Specified exact ASCII pattern matching
3. **Clear sorting semantics:** Defined exact tie-breaking rules
4. **Memory constraints:** Specified O(k) memory target
5. **Algorithm guidance:** Recommended specific approaches (heapq.nsmallest)

## Case Study 2: Financial Question Answering (FailSafeQA Benchmark)

### Optimization Focus Areas
- Improved context grounding
- Enhanced robustness to query variations
- Developed clear refusal policies
- Standardized answer formatting

### Optimized Prompt Structure
The financial QA optimization involved creating a comprehensive system prompt with:

1. **Behavioral Priorities**
   - Strict context grounding requirements
   - Evidence verification protocols

2. **Evidence Checking Mechanisms**
   - Clear guidelines for information validation
   - Handling of conflicting information

3. **Noise Handling Strategies**
   - OCR artifact management
   - Query ambiguity resolution

4. **Explicit Refusal Guidelines**
   - Clear policies for insufficient information
   - Structured refusal responses

5. **Precise Answer Formatting**
   - Standardized response structure
   - Consistent output format

### Evaluation Methodology
- Used FailSafeQA benchmark for testing
- Focused on prompt robustness across variations
- Measured compliance rates and accuracy improvements

## Technical Implementation Details

### Optimization Tools
- **OpenAI Prompt Optimizer:** Primary tool for systematic improvement
- **Quantitative Metrics:** Performance, memory, and compliance measurements
- **Qualitative Assessment:** Code quality and adherence evaluation

### Performance Measurement
- Runtime analysis across multiple runs
- Memory usage profiling
- Code quality scoring
- Model adherence tracking

## Best Practices for Prompt Optimization

### 1. Constraint Specification
- Use explicit, unambiguous requirements
- Avoid contradictory instructions
- Define clear success criteria

### 2. Output Format Control
- Specify exact output structure
- Include format examples where helpful
- Define edge case handling

### 3. Context Management
- Provide clear context boundaries
- Establish information verification processes
- Handle incomplete or ambiguous inputs

### 4. Iterative Refinement
- Start with baseline measurements
- Apply systematic improvements
- Validate changes through testing

### 5. Evaluation Framework
- Establish quantitative metrics
- Include qualitative assessments
- Test across diverse scenarios

## Key Takeaways

1. **Systematic Approach:** Prompt optimization requires methodical analysis and iterative improvement
2. **Precision Matters:** Specific constraints and clear requirements significantly improve performance
3. **Tool Integration:** OpenAI's optimization tools provide valuable systematic improvement capabilities
4. **Measurement is Critical:** Both quantitative and qualitative evaluation is essential for validation
5. **Domain-Specific Considerations:** Different tasks (coding vs QA) require tailored optimization strategies

## Conclusion

The GPT-5 Prompt Optimization Cookbook demonstrates that careful prompt engineering can yield significant improvements in model performance, reliability, and efficiency. The systematic approach outlined provides a framework for optimizing prompts across various domains and use cases.

The key insight is that precise, unambiguous instructions with clear constraints and well-defined success criteria enable AI models to perform more consistently and effectively. This methodology can be applied broadly to improve prompt effectiveness across different applications and domains.