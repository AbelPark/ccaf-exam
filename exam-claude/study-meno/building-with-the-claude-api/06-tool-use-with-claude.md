# Tool use with Claude

Tools allow Claude to access information from the outside world, extending its capabilities beyond what it learned during training. By default, **Claude only knows information from its training data** and can't access current events, real-time data, or external systems. **Tool use solves this limitation by creating a structured way for Claude to request and receive fresh information.**

## Introducing tool use

### How Tool Use Works

Tool use follows a specific back-and-forth pattern between your application and Claude. Here's the complete flow:

1. Initial Request: You send Claude a question along with instructions on how to get extra data from external sources
2. Tool Request: Claude analyzes the question and decides it needs additional information, then asks for specific details about what data it needs
3. Data Retrieval: Your server runs code to fetch the requested information from external APIs or databases
4. Final Response: You send the retrieved data back to Claude, which then generates a complete response using both the original question and the fresh data

### Key Benefits

- Real-time Information: Access current data that wasn't available during Claude's training
- External System Integration: Connect Claude to databases, APIs, and other services
- Dynamic Responses: Provide answers based on the latest available information
- Structured Interaction: Claude knows exactly what information it needs and how to ask for it

## Project overview

We're going to build a practical project that teaches Claude how to set reminders for future dates. This might sound simple at first, but it reveals several interesting challenges that we'll solve using custom tools.

### Why This Is Challenging

While Claude knows the current date, there are three specific problems we need to solve:

- Limited time awareness: Claude might know the current date, but not the exact time
- Date calculation issues: Claude doesn't always handle time-based addition well, especially when looking many days into the future
- No reminder capability: Claude doesn't know how to set a reminder - it has no built-in mechanism for this

Each of these limitations represents a gap between what Claude can do naturally and what we need for our reminder system. Tools are how we bridge these gaps.

### Tools We Need

We'll create three separate tools to handle each challenge:

- Get the current date time: Claude needs to know the current date and time precisely
- Add duration to date time: Claude isn't perfect with date time addition, so we'll give it a reliable tool for this
- Set a reminder: We need a way to actually set a reminder in the system

## Tool functions

### What Are Tool Functions?

A tool function is a plain Python function that gets executed automatically when Claude decides it needs extra information to help a user. For example, if someone asks "What time is it?", Claude would call your date/time tool to get the current time.

### Best Practices for Tool Functions

When writing tool functions, follow these guidelines:

- Use descriptive names: Both your function name and parameter names should clearly indicate their purpose
- Validate inputs: Check that required parameters aren't empty or invalid, and raise errors when they are
- Provide meaningful error messages: Claude can see error messages and might retry the function call with corrected parameters

**The validation is particularly important because Claude learns from errors.** If you raise a clear error like "Location cannot be empty", Claude might try calling the function again with a proper location value.

## Tool schemas

After writing your tool function, the next step is creating a JSON schema that tells Claude what arguments your function expects and how to use it. This schema acts as documentation that Claude reads to understand when and how to call your tools.

### Understanding JSON Schema

JSON Schema isn't specific to AI or tool calling - it's a widely-used data validation specification that's been around for years. The AI community adopted **it because it's a convenient way to describe function parameters and validate data.**

The complete tool specification has three main parts:

- name - A clear, descriptive name for your tool (like "get_weather")
- description - What the tool does, when to use it, and what it returns
- input_schema - The actual JSON schema describing the function's arguments

### Writing Effective Descriptions

Your tool description is crucial for helping Claude understand when to use your function. Best practices include:

- Aim for 3-4 sentences explaining what the tool does
- Describe when Claude should use it
- Explain what kind of data it returns
- Provide detailed descriptions for each argument

### Implementing the Schema in Code

Once Claude generates your schema, copy it into your code file. Here's a good naming pattern to follow:

Use the pattern of `function_name` followed by `function_name_schema` to keep your schemas organized and easy to match with their corresponding functions. (eg. `get_current_datetime`, `get_current_datetime_schema`)

### Adding Type Safety

For better type checking, import and use the `ToolParam` type from the Anthropic library:

```py
from anthropic.types import ToolParam

get_current_datetime_schema = ToolParam({
    "name": "get_current_datetime",
    "description": "Returns the current date and time formatted according to the specified format",
    # ... rest of schema
})
```

**While not strictly necessary for functionality**, this prevents type errors **when you use the schema with Claude's API and makes your code more robust**.

## Handling message blocks

When working with Claude's tool functionality, you'll encounter a new type of response structure that's different from the simple text responses you've seen before. Instead of just getting back a single text block, **Claude can now return multi-block messages that contain both text and tool usage information.**

### Making Tool-Enabled API Calls

To enable Claude to use tools, you need to include a `tools` parameter in your API call. Here's how to structure the request:

```py
messages = []
messages.append({
    "role": "user",
    "content": "What is the exact time, formatted as HH:MM:SS?"
})

response = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema],
)
```

The `tools` parameter takes a list of JSON schemas that describe the available functions Claude can call.

### Understanding Multi-Block Messages

A multi-block message typically contains:

- **Text Block** - Human-readable text explaining what Claude is doing (like "I can help you find out the current time. Let me find that information for you")
- **ToolUse Block** - Instructions for your code about which tool to call and what parameters to use

The ToolUse block includes:

- An ID for tracking the tool call
- The name of the function to call (like "get_current_datetime")
- Input parameters formatted as a dictionary
- The type designation "tool_use"

### Managing Conversation History with Multi-Block Messages

Remember that _Claude doesn't store conversation history_ - **you need to manage it manually.** When working with tool responses, **you must preserve the entire content structure, including all blocks.**

Here's how to properly append a multi-block assistant message to your conversation history:

```py
messages.append({
    "role": "assistant",
    "content": response.content
})
```

This preserves both the text block and the tool use block, which is crucial for maintaining the conversation context when you make subsequent API calls.

### The Complete Tool Usage Flow

The tool usage process follows this pattern:

1. Send user message with tool schema to Claude
2. Receive assistant message with text block and tool use block
3. Extract tool information and execute the actual function
4. Send tool result back to Claude along with complete conversation history
5. Receive final response from Claude

Each step requires careful handling of the message structure to ensure Claude has the full context it needs to provide accurate responses.

## Sending tool results

After Claude requests a tool call, _you need to execute the function and send the results back._ This completes the tool use workflow by providing Claude with the information it requested.

### Running the Tool Function

When Claude responds with a tool use block, you extract the input parameters and call your function. Here's how to access the tool parameters:

```py
response.content[1].input
```

This gives you a dictionary of the arguments Claude wants to pass to your function. Since your function expects keyword arguments rather than a dictionary, you use Python's unpacking syntax:

```py
get_current_datetime(**response.content[1].input)
```

### Tool Result Block

After running the tool function, _you need to send the results back to Claude using a tool result block._ This block goes inside a user message and tells Claude _what happened when you executed the tool._

The tool result block has several important properties:

- tool_use_id - Must match the id of the ToolUse block that this ToolResult corresponds to
- content - Output from running your tool, serialized as a string
- is_error - True if an error occurred

### Handling Multiple Tool Calls

Each tool call gets a unique ID, and _you must match these IDs when sending back results._

### Building the Follow-up Request

Your follow-up request to Claude must include the complete conversation history plus the new tool result. Here's the structure:

```py
messages.append({
    "role": "user",
    "content": [{
        "type": "tool_result",
        "tool_use_id": response.content[1].id,
        "content": "15:04:22",
        "is_error": False
    }]
})
```

The complete message history now contains:

- Original user message
- Assistant message with tool use block
- User message with tool result block

### Making the Final Request

When sending the follow-up request, **you must still include the tool schema even though you're not expecting Claude to make another tool call.** _Claude needs the schema to understand the tool references in your conversation history._

```py
client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    tools=[get_current_datetime_schema]
)
```
