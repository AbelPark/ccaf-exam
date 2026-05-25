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

> **Note: Don't rely on a fixed index like `response.content[1]`.** The position of the `ToolUseBlock` in `response.content` is not guaranteed — it depends on the model and the prompt. Older models (e.g. Sonnet 3.7) often return only a single `ToolUseBlock` (so the tool block is at index `0`), while Claude 4.x models tend to emit a short `TextBlock` first and then the `ToolUseBlock` (so the tool block is at index `1`). Always locate the tool block by its `type` instead:
>
> ```py
> tool_use_block = next(block for block in response.content if block.type == "tool_use")
> tool_use_block.input
> ```
>
> This works regardless of model version, temperature, or whether Claude prepends an explanatory text block.

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

## Multi-turn conversations with tools

### The Multi-Turn Tool Pattern

Here's what happens behind the scenes when Claude needs multiple tools:

1. User asks: "What day is 103 days from today?"
2. Claude responds with a tool use block requesting `get_current_datetime`
3. Your server calls the function and returns the result
4. Claude realizes it needs more information and requests `add_duration_to_datetime`
5. Your server calls that function and returns the result
6. Claude now has enough information to provide the final answer

### Building a Conversation Loop

To handle this pattern, _you need a conversation loop_ that continues until Claude stops requesting tools:

```py
def run_conversation(messages):
    while True:
        response = chat(messages)

        add_assistant_message(messages, response)

        # Pseudo code
        if response isn't asking for a tool:
            break

        tool_result_blocks = run_tools(response)
        add_user_message(messages, tool_result_blocks)

    return messages
```

### Key Improvements

These refactoring steps prepare your code for robust tool handling:

- Flexible message handling - Your helper functions can now work with different message formats
- Tool support in chat - The chat function can receive and pass through tool schemas
- Full message returns - You get complete message objects instead of just text, preserving all blocks
- Text extraction utility - Easy way to get readable text from complex messages

## Implementing multiple turns

Building a conversation system with tools requires implementing a loop that keeps calling Claude until it stops requesting tool usage. When Claude no longer asks for tools, _that signals it has a final response ready for the user._

### Detecting Tool Requests

The key to knowing whether Claude wants to use a tool lies in the `stop_reason` field of the response message. When Claude decides it needs to call a tool, this field gets set to `"tool_use"`. _This gives us a clean way to check if we need to continue the conversation loop:_

```py
if response.stop_reason != "tool_use":
    break  # Claude is done, no more tools needed
```

### The Conversation Loop

### Handling Multiple Tool Calls

Claude can request multiple tools in a single response. The message content contains a list of blocks, and we need to process each tool use block separately:

The `run_tools` function handles this by filtering for tool use blocks and processing each one:

```py
def run_tools(message):
    tool_requests = [
        block for block in message.content if block.type == "tool_use"
    ]
    tool_result_blocks = []

    for tool_request in tool_requests:
        # Process each tool request...
```

### Tool Result Blocks

Each tool use block must be answered with a corresponding tool result block. _The connection between them is maintained through matching IDs:_

```py
tool_result_block = {
    "type": "tool_result",
    "tool_use_id": tool_request.id,
    "content": json.dumps(tool_output),
    "is_error": False
}
```

### Error Handling

Robust tool execution requires handling potential errors. When a tool fails, we still need to provide a result block to Claude:

```py
try:
    tool_output = run_tool(tool_request.name, tool_request.input)
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": json.dumps(tool_output),
        "is_error": False
    }
except Exception as e:
    tool_result_block = {
        "type": "tool_result",
        "tool_use_id": tool_request.id,
        "content": f"Error: {e}",
        "is_error": True
    }
```

### Scalable Tool Routing

To support multiple tools, create a routing function that maps tool names to their implementations:

```py
def run_tool(tool_name, tool_input):
    if tool_name == "get_current_datetime":
        return get_current_datetime(**tool_input)
    elif tool_name == "another_tool":
        return another_tool(**tool_input)
    # Add more tools as needed
```

This approach makes it easy to add new tools without modifying the core conversation logic.

### Complete Workflow

The complete multi-turn conversation works like this:

- Send user message to Claude with available tools
- Claude responds with text and/or tool requests
- Execute all requested tools and create result blocks
- Send tool results back as a user message
- Repeat until Claude provides a final answer

## Using multiple tools

### The Simple Pattern for Adding Tools

Once you have the core tool infrastructure, adding new tools follows this pattern:

1. Create the tool function implementation
2. Define the tool schema
3. Add the schema to the tools list in run_conversation
4. Add a case for the tool in run_tool

This modular approach makes it easy to expand your AI assistant's capabilities without restructuring existing code. Each new tool integrates seamlessly with the existing conversation flow and tool-handling logic.

## Fine grained tool calling

When you combine tool use with streaming in Claude, you get real-time updates as the AI generates tool arguments. This creates a more responsive user experience, but there are some important details to understand about how it works behind the scenes.

### How JSON Validation Works

Here's where things get interesting. The Anthropic API doesn't immediately send you every chunk as Claude generates it. Instead, it buffers chunks and validates them first.

The API waits for complete top-level key-value pairs before sending anything. For example, if your tool expects this structure:

The API will:

1. Wait until the entire abstract value is complete
2. Validate that key-value pair against your schema
3. Send all the buffered chunks for abstract at once
4. Repeat the process for the meta object

_This validation process explains why you see delays followed by bursts of text, even with streaming enabled._ The chunks are being held back until a complete, valid top-level key-value pair is ready.

### Fine-Grained Tool Calling

If you need faster, more granular streaming - perhaps to show users immediate updates or start processing partial results quickly - you can enable fine-grained tool calling.

Fine-grained tool calling does one main thing: _it disables JSON validation on the API side._ This means:

- You get chunks as soon as Claude generates them
- No buffering delays between top-level keys
- More traditional streaming behavior
- Critical: _JSON validation is disabled_ - **your code must handle invalid JSON**

Enable it by adding `fine_grained=True` to your API call:

```py
run_conversation(
    messages,
    tools=[save_article_schema],
    fine_grained=True
)
```

With fine-grained tool calling, you might receive a word_count value much earlier in the stream, without waiting for the entire meta object to be completed.

### Handling Invalid JSON

When using fine-grained tool calling, Claude might generate invalid JSON like `"word_count": undefined` instead of a proper number. Your application needs to handle these cases gracefully:

```py
try:
parsed_args = json.loads(chunk.snapshot)
except json.JSONDecodeError: # Handle invalid JSON appropriately
print("Received invalid JSON, continuing...")
```

Without fine-grained tool calling, the API's validation would catch this error and potentially wrap problematic values in strings, which might not match your expected schema.

### When to Use Fine-Grained Tool Calling

Consider enabling fine-grained tool calling when:

- You need to show users real-time progress on tool argument generation
- You want to start processing partial tool results as quickly as possible
- The buffering delays negatively impact your user experience
- You're comfortable implementing robust JSON error handling

For most applications, the default behavior with validation is perfectly adequate. But when you need that extra responsiveness, fine-grained tool calling gives you the control to get chunks as fast as Claude can generate them.

## The text edit tool

Claude comes with one built-in tool that you don't need to create from scratch: the text editor tool. This tool gives Claude the ability to work with files and directories just like you would in a standard text editor.

### What the Text Editor Tool Can Do

The text editor tool provides Claude with a comprehensive set of file manipulation capabilities:

- View file or directory contents
- View specific ranges of lines in a file
- Replace text in a file
- Create new files
- Insert text at specific lines in a file
- Undo recent edits to files

### Understanding the Implementation Requirements

Here's where things get a bit confusing: while the tool schema is built into Claude, _you still need to provide the actual implementation._
Think of it this way - _Claude knows how to ask for file operations_, **but you need to write the code that actually performs those operations.**

When you use other tools, you write both the JSON schema and the function implementation. With the text editor tool, _Claude provides the schema knowledge_, but **you must write functions to handle Claude's requests to create files, read directories, replace text, and so on.**

### Why Use the Text Editor Tool?

You might wonder why this tool exists when modern code editors already have AI assistants built in. The text editor tool becomes valuable in scenarios where:

- You're building applications that need to programmatically edit files
- You're working in environments without access to full-featured code editors
- You want to integrate file editing capabilities directly into your Claude-powered applications

## The web search tool

_Claude includes a built-in web search tool_ that lets it search the internet for current or specialized information to answer user questions. Unlike other tools where you need to provide the implementation, Claude handles the entire search process automatically - _you just need to provide a simple schema to enable it._

### Setting Up the Web Search Tool

To use the web search tool, you create a schema object with these required fields:

```py
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5
}
```

The `max_uses` field limits how many searches Claude can perform. Claude might do follow-up searches based on initial results, so this prevents excessive API calls. A single search returns multiple results, but Claude may decide additional searches are needed.

### How the Response Works

When Claude uses the web search tool, the response contains several types of blocks:

- Text blocks - Claude's explanation of what it's doing
- ServerToolUseBlock - Shows the exact search query Claude used
- WebSearchToolResultBlock - Contains the search results
- WebSearchResultBlock - Individual search results with titles and URLs
- Citation blocks - Text that supports Claude's statements

### Restricting Search Domains

You can limit searches to specific domains using the `allowed_domains` field. This is particularly useful when you want reliable, authoritative sources:

```py
web_search_schema = {
    "type": "web_search_20250305",
    "name": "web_search",
    "max_uses": 5,
    "allowed_domains": ["nih.gov"]
}
```

### Rendering Search Results

The different block types in the response are designed for specific UI rendering:

- Render text blocks as regular content
- Display web search results as a list of sources at the top
- Show citations inline with the text, including the source domain, page title, URL, and quoted text

### Practical Usage

The web search tool works best for:

- Current events and recent developments
- Specialized information not in Claude's training data
- Fact-checking and finding authoritative sources
- Research tasks requiring up-to-date information
