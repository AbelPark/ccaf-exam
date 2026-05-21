## The Five-Step Request Flow

request to server > request to Anthropic API >model processing > response to server > and response to client

## Why You Need a Server

You should never make requests to the Anthropic API directly from client-side code. Here's why:
API requests require a secret API key for authentication
Exposing this key in client code creates a serious security vulnerability
Anyone could extract the key and make unauthorized requests

## Making API Requests

When your server contacts the Anthropic API, you can use either an official SDK or make plain HTTP requests.

Every request must include these essential fields:

**API Key** - Identifies your request to Anthropic
**Model** - Name of the model to use (like "claude-3-sonnet")
**Messages** - List containing the user's input text
**Max Tokens** - Limit for how many tokens Claude can generate

## Inside Claude's Processing

Once Anthropic receives your request, Claude processes it through four main stages
tokenization > embedding > contextualization > generation

### Tokenization

Claude first breaks your input text into smaller chunks called tokens.

### Embedding

Each token gets converted into an embedding.

### Contextualization

Claude refines each embedding based on surrounding words to determine the most likely meaning in context.

### Generation

The contextualized embeddings pass through an output layer that calculates probabilities for each possible next word. _Claude doesn't always pick the highest probability word_ - it uses a mix of probability and controlled randomness to create natural, varied responses.

## When Claude Stops Generating

Max tokens reached - Has it hit the limit you specified?
Natural ending - Did it generate an end-of-sequence token?
Stop sequence - Did it encounter a predefined stop phrase?

## The API Response

**Message** - The generated text
**Usage** - Count of input and output tokens
**Stop Reason** - Why generation ended
