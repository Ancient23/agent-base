# Introducing Contextual Retrieval

**Source:** https://www.anthropic.com/engineering/contextual-retrieval
**Fetched:** 2025-09-19
**Description:** Anthropic's engineering blog post on Contextual Retrieval technique for improving RAG systems

## Overview

Contextual Retrieval is an advanced technique for improving information retrieval in AI systems, specifically addressing the challenge of maintaining context when extracting relevant information from large knowledge bases. This method significantly improves the accuracy of Retrieval-Augmented Generation (RAG) systems by adding contextual information to document chunks before embedding and indexing.

## Key Problems with Traditional RAG

Traditional Retrieval-Augmented Generation (RAG) approaches often:
- Remove context when encoding information into embeddings
- Fail to retrieve relevant chunks due to lack of contextual understanding
- Split documents into small chunks (typically <200 tokens) that lose important contextual details
- Struggle with precise information retrieval when chunks lack sufficient context

## Contextual Retrieval Technique

### Core Methodology
1. Break knowledge base into text chunks (typically <200 tokens)
2. Generate contextual metadata for each chunk using Claude
3. Prepend contextual metadata to original chunk content
4. Create embeddings and BM25 indexes with contextualized chunks

### Two Sub-Techniques
1. **Contextual Embeddings**: Add context before creating vector embeddings
2. **Contextual BM25**: Add context before creating BM25 (Best Matching 25) keyword indexes

### Example Transformation

```python
original_chunk = "The company's revenue grew by 3% over the previous quarter."

contextualized_chunk = "This chunk is from an SEC filing on ACME corp's performance in Q2 2023; the previous quarter's revenue was $314 million. The company's revenue grew by 3% over the previous quarter."
```

## Performance Improvements

### Retrieval Accuracy Metrics
Experimental results on a comprehensive evaluation dataset showed:
- **Contextual Embeddings alone**: Reduced retrieval failure rate by 35% (5.7% → 3.7%)
- **Contextual Embeddings + Contextual BM25**: Reduced failure rate by 49% (5.7% → 2.9%)
- **With Reranking**: Reduced failure rate by 67% (5.7% → 1.9%)

### Cost Analysis
- **One-time preprocessing cost**: ~$1.02 per million document tokens
- Cost can be reduced using prompt caching for repeated contextualizations
- Investment pays off through significantly improved retrieval accuracy

## Implementation Considerations

1. **Chunk Boundaries**: Carefully experiment with different chunk sizes and boundaries
2. **Embedding Models**: Choose appropriate models (Gemini and Voyage recommended for best performance)
3. **Custom Prompts**: Create domain-specific contextualizer prompts for better results
4. **Chunk Count**: Balance number of chunks (20 chunks found most effective vs traditional 5-10)
5. **Evaluation**: Always run comprehensive evaluations to measure improvement
6. **Prompt Caching**: Leverage prompt caching to reduce preprocessing costs

## Recommended Approach

To maximize retrieval performance:
1. **Use Contextual Embeddings**: Implement with Voyage or Gemini embedding models
2. **Combine with Contextual BM25**: Use both embedding and keyword-based retrieval
3. **Add Reranking**: Include a reranking step for final chunk selection
4. **Optimal Chunk Count**: Pass top 20 chunks to the language model (instead of traditional 5-10)
5. **Preprocessing Pipeline**: Invest in one-time contextual preprocessing for long-term gains

## Practical Implementation

### Tools and Resources
- **Anthropic Cookbook**: [Contextual Embeddings Implementation](https://github.com/anthropics/anthropic-cookbook/tree/main/skills/contextual-embeddings)
- **Evaluation Framework**: Use provided evaluation metrics to measure improvements
- **Prompt Templates**: Leverage Anthropic's contextualizer prompt examples

### Implementation Steps
1. Set up preprocessing pipeline with Claude for context generation
2. Create contextual embeddings using preferred embedding model
3. Build contextual BM25 index alongside embeddings
4. Implement reranking mechanism
5. Evaluate performance against baseline RAG system
6. Fine-tune chunk count and context generation prompts

### Expected Outcomes
With proper implementation, expect:
- 35-67% reduction in retrieval failure rates
- Better semantic understanding of document chunks
- Improved relevance of retrieved information
- More accurate question answering and information synthesis