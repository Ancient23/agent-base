# GPT-5 New Parameters and Tools

**Source**: https://cookbook.openai.com/examples/gpt-5/gpt-5_new_params_and_tools
**Fetched**: 2025-09-19
**Description**: Documentation for GPT-5's four new developer controls including verbosity, free-form function calling, context-free grammar, and minimal reasoning.

## Overview

GPT-5 introduces four key new developer controls that provide enhanced flexibility and performance optimization for various use cases.

## 1. Verbosity Parameter

The verbosity parameter allows developers to control the length and detail level of model responses.

### Values
- `low` - Produces terse, minimal prose responses
- `medium` (default) - Provides balanced detail level
- `high` - Generates verbose responses, ideal for audits and teaching scenarios

### Implementation Example
```python
response = client.responses.create(
    model="gpt-5-mini",
    input=question,
    text={"verbosity": verbosity}
)
```

## 2. Free-Form Function Calling

This feature enables the model to generate raw text payloads directly to custom tools, eliminating the need for JSON wrapping.

### Supported Environments
- Code sandboxes
- SQL databases
- Shell environments
- Configuration generators

### Key Benefits
- More natural tool interactions
- Reduced overhead from JSON formatting
- Direct payload generation

## 3. Context-Free Grammar (CFG)

CFG allows developers to constrain model output to specific syntax rules, ensuring adherence to particular formats or programming language requirements.

### Supported Syntax
- Lark grammar syntax
- Regex syntax

### Use Cases
- Enforce programming language syntax rules
- Create custom output formats
- Ensure structured data generation
- Validate response formatting

## 4. Minimal Reasoning

The minimal reasoning feature is designed to minimize latency and accelerate token generation for simpler tasks.

### Ideal Applications
- Deterministic tasks
- Lightweight operations
- Simple classification problems
- Data extraction and formatting

### Performance Benefits
- Reduced processing time
- Faster token generation
- Lower computational overhead

## Model and API Support

### Supported Models
- gpt-5
- gpt-5-mini
- gpt-5-nano

### Supported API Endpoints
- Responses API
- Chat Completions API

## Best Practices and Recommendations

1. **API Selection**: Use the Responses API for optimal performance
2. **Reasoning Configuration**: Set reasoning effort explicitly based on task complexity requirements
3. **Verbosity Optimization**: Experiment with different verbosity levels to optimize output for your specific use case
4. **Tool Integration**: Leverage free-form function calling for more natural tool interactions
5. **Grammar Constraints**: Implement CFG when strict output formatting is required
6. **Performance Tuning**: Use minimal reasoning for simple, deterministic tasks to improve response times

## Technical Considerations

- These features are designed to work together and can be combined for maximum effectiveness
- Model selection should align with the complexity of your use case
- API endpoint choice affects performance characteristics
- Proper configuration of these parameters can significantly impact both response quality and latency