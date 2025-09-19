# Claude Prompt Caching Cookbook

**Source:** https://github.com/anthropics/claude-cookbooks/blob/main/misc/prompt_caching.ipynb
**Fetched:** 2025-09-19
**Description:** Practical Jupyter notebook tutorial demonstrating prompt caching implementation with Claude API

## Overview

This cookbook demonstrates how to use prompt caching through the Claude API, highlighting key benefits:
- Reduce latency by over 2x
- Decrease costs up to 90%
- Improve response quality by including more detailed context

## Setup

### Required Dependencies

```python
%pip install anthropic bs4 --quiet

import anthropic
import time
import requests
from bs4 import BeautifulSoup

client = anthropic.Anthropic()
MODEL_NAME = "claude-3-5-sonnet-20241022"
```

### Required Headers

For prompt caching functionality, you need to include the beta header:

```python
extra_headers = {"anthropic-beta": "prompt-caching-2024-07-31"}
```

## Single Turn Caching Example

### Without Caching

```python
# Fetch book content (Pride and Prejudice example)
url = "https://www.gutenberg.org/files/1342/1342-h/1342-h.htm"
response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')

# Extract text content
book_content = soup.get_text()

# Non-cached API call
start_time = time.time()
response = client.messages.create(
    model=MODEL_NAME,
    max_tokens=1000,
    messages=[{
        "role": "user",
        "content": f"{book_content}\n\nPlease write a short summary of this book."
    }]
)
end_time = time.time()

print(f"Non-cached response time: {end_time - start_time:.2f} seconds")
# Output: ~20.37 seconds
```

### With Caching

```python
# Cached API call using cache_control
start_time = time.time()
response = client.messages.create(
    model=MODEL_NAME,
    max_tokens=1000,
    extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"},
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": book_content,
                "cache_control": {"type": "ephemeral"}
            },
            {
                "type": "text",
                "text": "\n\nPlease write a short summary of this book."
            }
        ]
    }]
)
end_time = time.time()

print(f"Cached response time: {end_time - start_time:.2f} seconds")
# Output: ~2.92 seconds (significant improvement!)
```

## Multi-Turn Conversation Caching

### Progressive Caching Demonstration

```python
# Initialize conversation with cached content
def create_cached_conversation(book_content, new_question):
    return [{
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": book_content,
                "cache_control": {"type": "ephemeral"}
            },
            {
                "type": "text",
                "text": f"\n\n{new_question}"
            }
        ]
    }]

# First turn - establishes cache
start_time = time.time()
messages = create_cached_conversation(book_content, "Who are the main characters?")

response1 = client.messages.create(
    model=MODEL_NAME,
    max_tokens=1000,
    extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"},
    messages=messages
)
end_time = time.time()

print(f"Turn 1 time: {end_time - start_time:.2f} seconds")
print(f"Cache creation tokens: {response1.usage.cache_creation_input_tokens}")
print(f"Cache read tokens: {response1.usage.cache_read_input_tokens}")

# Add assistant response to conversation
messages.append({
    "role": "assistant",
    "content": response1.content[0].text
})

# Second turn - benefits from cache
messages.append({
    "role": "user",
    "content": "What is the main theme of the story?"
})

start_time = time.time()
response2 = client.messages.create(
    model=MODEL_NAME,
    max_tokens=1000,
    extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"},
    messages=messages
)
end_time = time.time()

print(f"Turn 2 time: {end_time - start_time:.2f} seconds")
print(f"Cache creation tokens: {response2.usage.cache_creation_input_tokens}")
print(f"Cache read tokens: {response2.usage.cache_read_input_tokens}")
```

### Expected Performance Results

The cookbook demonstrates progressive improvement:

1. **First turn (cache establishment)**: ~24 seconds
   - High cache creation tokens
   - Zero cache read tokens

2. **Second turn (cache utilization)**: ~11 seconds
   - Zero cache creation tokens
   - High cache read tokens

3. **Subsequent turns**: ~6-7 seconds
   - Nearly 100% of input tokens from cache
   - Dramatic latency reduction

## Key Implementation Insights

### Cache Control Structure

```python
{
    "type": "text",
    "text": "content to cache",
    "cache_control": {"type": "ephemeral"}
}
```

### Message Structure for Caching

```python
messages = [{
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": cacheable_content,
            "cache_control": {"type": "ephemeral"}
        },
        {
            "type": "text",
            "text": "specific question or instruction"
        }
    ]
}]
```

### Performance Monitoring

```python
# Monitor cache performance through usage metrics
print(f"Total input tokens: {response.usage.input_tokens}")
print(f"Cache creation tokens: {response.usage.cache_creation_input_tokens}")
print(f"Cache read tokens: {response.usage.cache_read_input_tokens}")
print(f"Output tokens: {response.usage.output_tokens}")
```

## Best Practices from the Cookbook

### 1. Strategic Content Organization

- Place cacheable content (large documents, system instructions) at the beginning
- Use cache control on stable, reusable content
- Keep variable content (questions, specific instructions) separate

### 2. Multi-turn Optimization

- Establish cache early in the conversation
- Reuse cached content across multiple turns
- Monitor cache hit rates through usage metrics

### 3. Performance Measurement

- Always measure response times to verify caching benefits
- Track token usage to understand cache efficiency
- Compare cached vs non-cached performance

### 4. Error Handling

```python
try:
    response = client.messages.create(
        model=MODEL_NAME,
        max_tokens=1000,
        extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"},
        messages=messages
    )
except Exception as e:
    print(f"Error with caching: {e}")
    # Fallback to non-cached request
```

## Real-world Applications

### Document Analysis Workflow

```python
def analyze_document_with_caching(document_content, questions):
    """Analyze a large document with multiple questions using caching"""

    responses = []
    conversation = []

    for i, question in enumerate(questions):
        if i == 0:
            # First request establishes cache
            messages = [{
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": document_content,
                        "cache_control": {"type": "ephemeral"}
                    },
                    {
                        "type": "text",
                        "text": question
                    }
                ]
            }]
        else:
            # Subsequent requests reuse cache
            messages = conversation + [{
                "role": "user",
                "content": question
            }]

        start_time = time.time()
        response = client.messages.create(
            model=MODEL_NAME,
            max_tokens=1000,
            extra_headers={"anthropic-beta": "prompt-caching-2024-07-31"},
            messages=messages
        )
        end_time = time.time()

        # Track conversation for context
        if i == 0:
            conversation = messages + [{
                "role": "assistant",
                "content": response.content[0].text
            }]
        else:
            conversation.extend([
                {"role": "user", "content": question},
                {"role": "assistant", "content": response.content[0].text}
            ])

        responses.append({
            "question": question,
            "answer": response.content[0].text,
            "response_time": end_time - start_time,
            "cache_read_tokens": response.usage.cache_read_input_tokens,
            "cache_creation_tokens": response.usage.cache_creation_input_tokens
        })

    return responses
```

## Performance Benchmarks

Based on the cookbook examples:

### Latency Improvements
- **Non-cached**: 20+ seconds for large document processing
- **First cached request**: ~24 seconds (cache establishment overhead)
- **Subsequent cached requests**: 6-11 seconds (60-75% improvement)

### Token Efficiency
- **Cache hit rate**: Up to 100% for repeated content
- **Cost reduction**: Up to 90% for cached tokens
- **Processing speed**: 2x+ faster for cache hits

## Troubleshooting Common Issues

### Cache Not Working
1. Verify beta header is included
2. Check cache_control syntax
3. Ensure content is exactly identical between requests
4. Confirm model supports caching

### Performance Not Improving
1. Measure cache hit rates via usage metrics
2. Verify sufficient cacheable content size
3. Check cache timing (5-minute expiration)
4. Monitor network latency factors

### Integration Challenges
1. Handle cache misses gracefully
2. Implement fallback for non-cached requests
3. Monitor cache efficiency over time
4. Balance cache size vs. performance gains

## Conclusion

This cookbook demonstrates that prompt caching can provide significant performance improvements for applications with:
- Large, reusable content (documents, system instructions)
- Multi-turn conversations
- Repeated analysis tasks
- Tool-heavy applications

The key is strategic placement of cache control on stable content while keeping variable elements separate for optimal cache utilization.

---

*Based on the Claude Cookbooks Jupyter notebook tutorial for practical prompt caching implementation.*