# Prompt engineering

## The Iterative Improvement Process

The approach follows a clear cycle that you can repeat until you achieve your desired results:

1. **Set a goal** - Define what you want your prompt to accomplish
2. **Write an initial prompt** - Create a basic first attempt
3. **Evaluate the prompt** - Test it against your criteria
4. **Apply prompt engineering techniques** - Use specific methods to improve performance
5. **Re-evaluate** - Verify that your changes actually improved the results

You repeat the last two steps until you're satisfied with the performance. Each iteration should show measurable improvement in your evaluation scores.

## Being clear and direct

When crafting that crucial first line, you want to focus on two key principles: clarity and directness. This means using simple language that leaves no room for ambiguity about what you want Claude to do.

### Clear Communication

Being "clear" means:

- Use simple language that anyone can understand
- State exactly what you want without beating around the bush
- Lead with a straightforward statement of Claude's task

Instead of writing something vague like "I need to know about those things people put on their roofs that use sun - those solar panel things, I think they're called," be direct and write: "Write three paragraphs about how solar panels work."

## Direct Instructions

Being "direct" focuses on how you structure your request:

- Use instructions, not questions
- Start with direct action verbs like "Write," "Create," or "Generate"

Rather than asking "I was reading about renewable energy and geothermal energy sounds neat. What countries use it?" try: "Identify three countries that use geothermal energy. Include generation stats for each."

## Being specific

There are two main approaches to being specific in your prompts, and you'll often see them used together in professional applications.

### Output Quality Guidelines

The first type focuses on listing qualities that your output should have. These guidelines help you control:

- Length of the response
- Structure and format
- Specific attributes or elements to include
- Tone or style requirements

### Process Steps

The second type provides specific steps for Claude to follow. This approach is particularly useful when you want Claude to think through a problem systematically or consider multiple perspectives before arriving at a final answer.

Instead of jumping straight to writing, you might ask Claude to:

1. Brainstorm three talents that would create dramatic tension
2. Pick the most interesting talent
3. Outline a pivotal scene that reveals the talent
4. Brainstorm supporting character types that could increase the impact

### When to Use Each Approach

Here's a practical guide for when to use each type of specificity:

#### **Always Use Output Guidelines**

You should include quality guidelines in almost every prompt you write. They're your safety net for getting consistent, useful results.

#### **Use Process Steps For Complex Problems**

Add step-by-step instructions when you're dealing with:

- Troubleshooting complex problems
- Decision-making scenarios
- Critical thinking tasks
- Any situation where you want Claude to consider multiple angles

## Structure with XML tags

When you're building prompts that include a lot of content, Claude can sometimes struggle to understand which pieces of text belong together or what different sections are supposed to represent. **XML tags provide a simple way to add structure and clarity to your prompts**, especially when you're interpolating large amounts of data.

### Custom Tag Names

You don't need to use official XML tags. Create descriptive names that make sense for your content:

- <sales_records> is better than <data>
- <athlete_information> clearly identifies user details
- <my_code> and <docs> separate different types of content

The more specific and descriptive your tag names, the better Claude can understand the purpose of each section.

### When to Use XML Tags

XML tags are most useful when:

- Including large amounts of context or data
- Mixing different types of content (code, documentation, data)
- You want to be extra clear about content boundaries
- Working with complex prompts that interpolate multiple variables

Even for shorter content, XML tags can help serve as delimiters that make your prompt structure more obvious to Claude.

## Providing examples

Providing examples in your prompts is one of the most effective prompt engineering techniques you'll use. This approach, known as "one-shot" or "multi-shot" prompting, involves giving Claude sample input/output pairs to guide its responses.

### Adding Examples to Handle Corner Cases

To solve this, you can add examples that show Claude how to handle tricky cases:

The improved prompt includes:

- A clear positive example: "Great game tonight!" → "Positive"
- A sarcastic example: "Oh yeah, I really needed a flight delay tonight! Excellent!" → "Negative"
- Context explaining why sarcasm should be treated carefully

Notice how the examples are wrapped in XML tags like <sample_input> and <ideal_output>. This structure makes it crystal clear to Claude what each part represents.

### When to Use Examples

Examples are particularly useful for:

- Capturing corner cases or edge scenarios
- Defining complex output formats (like specific JSON structures)
- Showing the exact style or tone you want
- Demonstrating how to handle ambiguous inputs

### One-Shot vs Multi-Shot

One-Shot: Provide a single example to establish the pattern
Multi-Shot: Provide multiple examples to cover different scenarios

Use multi-shot when you need to handle various edge cases or want to show different types of valid responses.

### Finding Good Examples from Evaluations

When running prompt evaluations, look for your highest-scoring outputs to use as examples:

Find responses that scored 10 (or your highest available score) and use those input/output pairs as examples in your prompt. This helps Claude understand what "perfect" output looks like for your specific use case.

### Adding Context to Examples

Don't just provide the input/output pair - explain why the output is good:

```md
<ideal_output>
[Your example output here]
</ideal_output>

This example is well-structured, provides detailed information on food choices and quantities, and aligns with the athlete's goals and restrictions.
```

This additional context helps Claude understand the reasoning behind good responses, not just the format.

### Best Practices

- Always use XML tags to structure your examples clearly
- Be explicit about what you're showing: "Here is an example input with an ideal response"
- Include examples that address your most common failure cases
- Explain why your example outputs are considered ideal
- Keep examples relevant to your specific task

Examples are especially powerful because they show rather than tell. Instead of trying to describe exactly what you want in words, you demonstrate it directly. This makes your prompts much more reliable and helps Claude understand subtle requirements that might be hard to express in instructions alone.
