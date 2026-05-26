# RAG and Agentic Search

## Introducing Retrieval Augmented Generation

Retrieval Augmented Generation (RAG) is a technique that _helps you work with large documents that are too big to fit into a single prompt._ Instead of cramming everything into one massive prompt, RAG breaks documents into chunks and only includes the most relevant pieces when answering questions.

### The Problem with Large Documents

#### Option 1: Include Everything in the Prompt

This approach has serious limitations:

- There's a hard limit on prompt length - your document might be too long
- Claude becomes less effective with very long prompts
- Larger prompts cost more to process
- Larger prompts take longer to process

#### Option 2: Break Documents into Chunks

RAG takes a smarter approach. First, you break the document into smaller chunks during a preprocessing step. Then, when a user asks a question, _you find the chunks most relevant to their question and only include those in your prompt._

Here's how it works: if someone asks "What risks does this company face?" you'd search through your chunks, find the "Risk Factors" section, and include just that relevant chunk in your prompt.

### Benefits of RAG

- Claude can focus on only the most relevant content
- Scales up to very large documents
- Works with multiple documents
- Smaller prompts cost less and run faster

### Challenges with RAG

- Requires a preprocessing step to chunk documents
- Need a search mechanism to find "relevant" chunks
- Included chunks might not contain all the context Claude needs
- Many ways to chunk text - which approach is best?

For example, you could split documents into equal-sized portions, or you could create chunks based on document structure like headers and sections. Each approach has trade-offs you'll need to evaluate for your specific use case.

## Text chunking strategies

Text chunking is one of the most critical steps in building a RAG (Retrieval Augmented Generation) pipeline. How you break up your documents directly impacts the quality of your entire system. A poor chunking strategy can lead to irrelevant context being inserted into your prompts, causing your AI to give completely wrong answers.

### Size-Based Chunking

Size-based chunking is the simplest approach - _you divide your text into strings of equal length._ If you have a 325-character document, you might split it into three chunks of roughly 108 characters each.

This method is easy to implement and works with any type of document, but it has clear downsides:

- Words get cut off mid-sentence
- Chunks lose important context from surrounding text
- Section headers might be separated from their content

To address these issues, _you can add overlap between chunks._ This means each chunk includes some characters from the neighboring chunks, providing better context and ensuring complete words and sentences.

### Structure-Based Chunking

Structure-based chunking divides text based on the document's natural structure - headers, paragraphs, and sections. This works great when you have well-formatted documents like Markdown files.

This approach gives you _the cleanest, most meaningful chunks because each one represents a complete section._ However, **it only works when you have guarantees about your document structure.** Many real-world documents are plain text or PDFs without clear structural markers.

### Semantic-Based Chunking

Semantic-based chunking is _the most sophisticated approach._ You divide text into sentences, then use natural language processing to determine how related consecutive sentences are. You build chunks from groups of related sentences.

This method is computationally expensive but produces the most relevant chunks. _It requires understanding the meaning of individual sentences and is more complex to implement than the other strategies._

### Sentence-Based Chunking

A practical middle ground is chunking by sentences. You split the text into individual sentences using regular expressions, then group them into chunks with optional overlap:

### Choosing Your Strategy

Your choice depends entirely on your use case and document guarantees:

- Structure-based: Best results when you control document formatting (like internal company reports)
- Sentence-based: Good middle ground for most text documents
- Size-based: Most reliable fallback that works with any content type, including code

_Size-based chunking with overlap is often the go-to choice in production_ because **it's simple, reliable, and works with any document type.** While it may not give perfect results, it consistently produces reasonable chunks that won't break your pipeline.

## Text embeddings
