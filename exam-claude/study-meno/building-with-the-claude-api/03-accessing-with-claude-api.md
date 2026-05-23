## Understanding Stream Events

When you enable streaming, Claude sends back several types of events:

**MessageStart** - A new message is being sent
**ContentBlockStart** - Start of a new block containing text, tool use, or other content
**ContentBlockDelta** - Chunks of the actual generated text
**ContentBlockStop** - The current content block has been completed
**MessageDelta** - The current message is complete
**MessageStop** - End of information about the current message

The `ContentBlockDelta` events contain the actual generated text that you'll want to display to users.
