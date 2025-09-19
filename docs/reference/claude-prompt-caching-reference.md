# Claude Prompt Caching Reference

**Source:** https://docs.claude.com/en/docs/build-with-claude/prompt-caching
**Fetched:** 2025-09-19
**Description:** Complete reference for Claude API prompt caching functionality

## Overview

Prompt caching is a powerful feature that allows you to cache frequently used prompt prefixes, reducing latency and costs for repetitive API calls. When you send a prompt with cached content, Claude can reuse the cached portion instead of reprocessing it, leading to faster response times and lower token costs.

## How Prompt Caching Works

Prompt caching works by storing the processed state of a prompt prefix. When a new request comes in, Claude checks if the beginning of the prompt matches any cached content. If there's a match, Claude resumes processing from the end of the cached section, significantly reducing processing time and costs.

### Key Benefits

- **Reduced Latency**: Cached content is processed much faster than new content
- **Lower Costs**: Cache hits cost significantly less than processing new tokens
- **Improved Performance**: Especially beneficial for long, repeated content like system instructions or document contexts

## Supported Models

Prompt caching is supported on the following models:

- Claude 3.5 Sonnet
- Claude 3.5 Haiku
- Claude 3 Opus
- Claude 3 Sonnet
- Claude 3 Haiku

## Implementation

### Basic Cache Control

To enable caching for a portion of your prompt, add the `cache_control` parameter to the relevant message or content block:

```json
{
  "model": "claude-3-5-sonnet-20241022",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Here is a large document for analysis:",
          "cache_control": {"type": "ephemeral"}
        },
        {
          "type": "text",
          "text": "Analyze this document and provide insights."
        }
      ]
    }
  ]
}
```

### Cache Control Types

Currently, only `"ephemeral"` cache type is supported:

```json
{
  "cache_control": {
    "type": "ephemeral"
  }
}
```

### Cache Placement Rules

1. **Cache breakpoints can be placed on:**
   - User messages
   - Assistant messages (in multi-turn conversations)
   - Tools in the tools array
   - Tool use content blocks
   - Text content blocks
   - Image content blocks

2. **Cache breakpoints cannot be placed on:**
   - System messages
   - The final assistant message in a conversation

3. **Maximum of 4 cache breakpoints** per request

### Python SDK Example

```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")

# Long document to be cached
long_document = """
[Very long document content that you want to reuse across multiple requests]
"""

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": f"Here is a document to analyze:\n\n{long_document}",
                    "cache_control": {"type": "ephemeral"}
                },
                {
                    "type": "text",
                    "text": "Please provide a summary."
                }
            ]
        }
    ]
)
```

### TypeScript SDK Example

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: 'your-api-key',
});

const longDocument = `
[Very long document content that you want to reuse across multiple requests]
`;

const response = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 1024,
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: `Here is a document to analyze:\n\n${longDocument}`,
          cache_control: { type: 'ephemeral' }
        },
        {
          type: 'text',
          text: 'Please provide a summary.'
        }
      ]
    }
  ]
});
```

## Caching System Messages

While you cannot directly cache system messages, you can move system content to the first user message and cache it there:

```python
# Instead of system message
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "You are an expert legal analyst. Your task is to analyze legal documents and provide comprehensive insights...",
                    "cache_control": {"type": "ephemeral"}
                },
                {
                    "type": "text",
                    "text": "Please analyze this contract: [contract text]"
                }
            ]
        }
    ]
)
```

## Multi-turn Conversations

In multi-turn conversations, you can cache earlier parts of the conversation:

```python
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "Analyze this document: [document]",
                    "cache_control": {"type": "ephemeral"}
                }
            ]
        },
        {
            "role": "assistant",
            "content": "I've analyzed the document. Here are the key findings: [analysis]"
        },
        {
            "role": "user",
            "content": "What are the main risks mentioned?"
        }
    ]
)
```

## Pricing

Prompt caching has the following pricing structure:

### Cache Write Costs
When content is cached for the first time, you pay:
- **25% more** than the standard input token price
- This covers the cost of processing and storing the content

### Cache Read Costs
When cached content is reused, you pay:
- **10% of the standard input token price**
- Significant savings for frequently reused content

### Example Pricing Calculation

For a 10,000 token prompt that's cached:
- First request (cache write): 10,000 tokens × 1.25 × base price
- Subsequent requests (cache read): 10,000 tokens × 0.10 × base price + new tokens × base price

## Monitoring Cache Performance

The API response includes cache-related fields to help you monitor performance:

```json
{
  "id": "msg_123",
  "type": "message",
  "usage": {
    "input_tokens": 1000,
    "cache_creation_input_tokens": 800,
    "cache_read_input_tokens": 200,
    "output_tokens": 150
  }
}
```

### Usage Fields Explained

- **`input_tokens`**: Total input tokens processed
- **`cache_creation_input_tokens`**: Tokens that were cached (first time)
- **`cache_read_input_tokens`**: Tokens that were read from cache
- **`output_tokens`**: Generated response tokens

## Best Practices

### 1. Strategic Cache Placement

Cache stable, reusable content at the beginning of your prompt:
- System instructions
- Large documents or datasets
- Tool definitions
- Context that doesn't change between requests

### 2. Optimal Cache Size

- **Minimum cacheable content**: Varies by model (typically 1024+ tokens)
- **Maximum benefit**: For content that's reused multiple times
- **Consider cache lifetime**: Content is cached for up to 5 minutes by default

### 3. Cache Invalidation

Cached content is invalidated when:
- The exact prompt prefix changes
- Tools array is modified
- System message changes
- More than 5 minutes pass since last use

### 4. Effective Patterns

```python
# Good: Cache stable context at the beginning
{
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": "You are a coding assistant with access to this large codebase: [codebase]",
            "cache_control": {"type": "ephemeral"}
        },
        {
            "type": "text",
            "text": "Help me debug this specific function: [function]"
        }
    ]
}

# Avoid: Caching frequently changing content
{
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": f"Today is {current_date}. Process this: [data]",
            "cache_control": {"type": "ephemeral"}  # Bad: date changes daily
        }
    ]
}
```

## Common Use Cases

### 1. Document Analysis

Cache large documents that need multiple rounds of analysis:

```python
# Cache the document once, ask multiple questions
document_analysis_prompt = f"""
Please analyze this legal document:

{large_legal_document}
"""

# First request
response1 = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": document_analysis_prompt, "cache_control": {"type": "ephemeral"}},
            {"type": "text", "text": "What are the key terms?"}
        ]
    }]
)

# Subsequent requests reuse cached document
response2 = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": document_analysis_prompt, "cache_control": {"type": "ephemeral"}},
            {"type": "text", "text": "What are the potential risks?"}
        ]
    }]
)
```

### 2. Tool-Heavy Applications

Cache tool definitions for applications with many tools:

```python
tools = [
    {
        "name": "web_search",
        "description": "Search the web for information",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            }
        }
    },
    # ... many more tools
]

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    tools=tools,
    tool_choice="auto",
    messages=[{
        "role": "user",
        "content": "Search for information about AI safety",
        "cache_control": {"type": "ephemeral"}  # Cache applied to tools
    }]
)
```

### 3. Conversational Agents

Cache conversation context in long dialogues:

```python
# Cache early conversation context
conversation = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "You are my personal AI assistant. Here's my background: [detailed background]",
                "cache_control": {"type": "ephemeral"}
            }
        ]
    },
    {
        "role": "assistant",
        "content": "I understand your background and I'm here to help."
    },
    # ... more conversation turns
]
```

## Limitations and Considerations

### Technical Limitations

1. **Exact Matching Required**: Cache only works with exact prompt prefix matches
2. **Cache Scope**: Caches are organization-specific, not shared between organizations
3. **Cache Duration**: Default cache lifetime is 5 minutes
4. **Model Restrictions**: Only supported on specific Claude models

### Performance Considerations

1. **Minimum Token Threshold**: Very small content may not benefit from caching
2. **Network Latency**: Cache benefits are most apparent with larger content
3. **Request Patterns**: Most beneficial for repeated requests with same prefix

### Security and Privacy

1. **Data Isolation**: Cached content is isolated per organization
2. **Automatic Expiration**: Cached content expires automatically
3. **No Persistence**: Caches are ephemeral and not permanently stored

## Troubleshooting

### Cache Misses

If you're not seeing expected cache hits:

1. **Verify exact matching**: Ensure the prompt prefix matches exactly
2. **Check cache lifetime**: Verify requests are within 5 minutes
3. **Review cache placement**: Confirm cache_control is on the right content blocks
4. **Monitor usage fields**: Check API response for cache-related metrics

### Performance Issues

1. **Optimize cache size**: Find the right balance between cache size and benefit
2. **Strategic placement**: Cache the most stable, reusable content
3. **Monitor costs**: Track cache creation vs. read costs

## FAQ

### Q: How long does cached content remain available?
A: Cached content remains available for 5 minutes from the last time it was accessed.

### Q: Can I cache system messages?
A: No, but you can move system content to the first user message and cache it there.

### Q: What happens if I modify cached content slightly?
A: Any change invalidates the cache. The content must match exactly for cache hits.

### Q: How many cache breakpoints can I use?
A: You can use up to 4 cache breakpoints per request.

### Q: Do cache costs apply to output tokens?
A: No, caching only affects input tokens. Output token pricing remains the same.

### Q: Can I share caches between different API keys?
A: No, caches are isolated per organization and cannot be shared.

---

*This documentation is based on Claude's official prompt caching documentation and reflects the current capabilities as of the fetch date.*