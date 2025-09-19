# Claude Prompt Caching Documentation

**Source:** https://docs.claude.com/en/docs/build-with-claude/prompt-caching
**Fetched:** 2025-09-19
**Description:** Complete documentation for Claude's prompt caching feature, including implementation details, pricing, and best practices.

---

## Overview

Prompt caching is a powerful feature that allows you to cache the prefix of your prompt, enabling you to resume from specific points in your conversation. This optimization reduces both processing time and costs for repetitive tasks or when working with large, stable contexts.

### Key Benefits

- **Reduced Latency**: Cached content processes faster than new content
- **Cost Optimization**: Lower costs for cached token processing
- **Improved Efficiency**: Ideal for repetitive tasks with stable content

## Supported Models

Prompt caching is available for the following Claude models:

- Claude Opus 4.1
- Claude Opus 4
- Claude Sonnet 4
- Claude Sonnet 3.7
- Claude Haiku 3.5
- Claude Haiku 3

## Technical Specifications

### Cache Breakpoints
- Support for up to **4 cache breakpoints** per prompt
- Each breakpoint marks a position where caching can resume

### Cache Lifetime Options
- **Default**: 5 minutes
- **Extended**: 1 hour (optional, with different pricing)

### Minimum Token Requirements
- **Most models**: 1024 tokens minimum for cacheable content
- **Haiku models**: 2048 tokens minimum for cacheable content

## Implementation

### Basic Cache Control Structure

To enable prompt caching, add the `cache_control` parameter to your content:

```json
{
  "type": "text",
  "text": "Your content here",
  "cache_control": {"type": "ephemeral"}
}
```

### System Message Caching Example

```curl
curl https://api.anthropic.com/v1/messages \
  -H "content-type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -d '{
    "model": "claude-opus-4-1-20250805",
    "max_tokens": 1024,
    "system": [
      {
        "type": "text",
        "text": "You are an AI assistant analyzing literary works. You have extensive knowledge of classic literature, literary devices, themes, and critical analysis techniques.",
        "cache_control": {"type": "ephemeral"}
      }
    ],
    "messages": [
      {
        "role": "user",
        "content": "Analyze the themes in Pride and Prejudice."
      }
    ]
  }'
```

### Multi-Part System with Caching

```json
{
  "model": "claude-opus-4-1-20250805",
  "max_tokens": 1024,
  "system": [
    {
      "type": "text",
      "text": "You are an expert data analyst."
    },
    {
      "type": "text",
      "text": "[Large dataset or context that you want to cache]",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": "Analyze this data for trends."
    }
  ]
}
```

### User Message Caching

```json
{
  "role": "user",
  "content": [
    {
      "type": "text",
      "text": "Here is a large document to analyze:"
    },
    {
      "type": "text",
      "text": "[Very long document content]",
      "cache_control": {"type": "ephemeral"}
    },
    {
      "type": "text",
      "text": "Please summarize the key points."
    }
  ]
}
```

## Pricing Structure

Prompt caching uses a tiered pricing model with different rates for various cache operations:

### Claude Opus 4.1 Example Pricing
- **Base input tokens**: $15.00 per million tokens
- **5-minute cache writes**: $18.75 per million tokens (1.25x base rate)
- **1-hour cache writes**: $30.00 per million tokens (2x base rate)
- **Cache hits & refreshes**: $1.50 per million tokens (0.1x base rate)
- **Output tokens**: $60.00 per million tokens

### Claude Sonnet 4 Example Pricing
- **Base input tokens**: $3.00 per million tokens
- **5-minute cache writes**: $3.75 per million tokens
- **1-hour cache writes**: $6.00 per million tokens
- **Cache hits & refreshes**: $0.30 per million tokens
- **Output tokens**: $15.00 per million tokens

*Note: Pricing may vary by model. Check the latest pricing documentation for current rates.*

## Best Practices

### 1. Cache Stable Content
- System instructions that don't change
- Large reference documents
- Code bases or documentation
- Examples and few-shot prompts

### 2. Optimal Cache Placement
- Place cached content at the beginning of your prompt
- Cache the largest, most stable portions first
- Ensure cached content meets minimum token requirements

### 3. Strategic Cache Usage
- Use caching for repetitive tasks with consistent context
- Cache content that will be reused within the cache lifetime
- Consider the 5-minute vs 1-hour cache trade-offs

### 4. Cost Optimization
- Balance cache write costs against cache hit savings
- Monitor usage patterns to optimize cache strategy
- Use shorter cache durations for frequently changing workflows

## Cache Behavior

### Cache Hits
When cached content is detected, the API will:
- Resume processing from the cache breakpoint
- Apply reduced pricing for cached tokens
- Maintain the same response quality

### Cache Misses
When content doesn't match cached content:
- Full processing occurs for all tokens
- Standard pricing applies
- New cache entries may be created

### Cache Expiration
- **5-minute cache**: Automatically expires after 5 minutes of inactivity
- **1-hour cache**: Automatically expires after 1 hour of inactivity
- Expired caches require full reprocessing

## Common Use Cases

### 1. Document Analysis
Cache large documents for multiple analysis requests:
```json
{
  "system": [
    {
      "type": "text",
      "text": "You are a document analyzer.",
      "cache_control": {"type": "ephemeral"}
    }
  ]
}
```

### 2. Code Review
Cache codebases for iterative review sessions:
```json
{
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Here's the codebase to review:",
          "cache_control": {"type": "ephemeral"}
        }
      ]
    }
  ]
}
```

### 3. Few-Shot Learning
Cache examples for consistent few-shot prompting:
```json
{
  "system": [
    {
      "type": "text",
      "text": "You are a text classifier. Here are examples:\n[Large set of examples]",
      "cache_control": {"type": "ephemeral"}
    }
  ]
}
```

## Troubleshooting

### Common Issues
1. **Content too short**: Ensure cached content meets minimum token requirements
2. **No cache hits**: Verify content exactly matches previous requests
3. **Unexpected costs**: Check cache write vs hit ratios

### Monitoring Cache Performance
- Track cache hit rates in your application
- Monitor cost savings from cache usage
- Analyze cache expiration patterns

## API Response Headers

Successful cache operations include response headers indicating cache status:
- Cache hit confirmations
- Cache creation notifications
- Cache usage statistics

---

*This documentation covers the essential aspects of Claude's prompt caching feature. For the most up-to-date information and additional details, refer to the official Anthropic documentation.*