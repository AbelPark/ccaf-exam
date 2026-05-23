# Prompt evaluation

## Prompt Engineering vs Prompt Evaluation

### Prompt engineering

is your toolkit for crafting effective prompts. It includes techniques like:

- Multishot prompting
- Structuring with XML tags
- Many other best practices

These techniques help Claude understand exactly what you're asking for and how you want it to respond.

### Prompt evaluation

takes a different approach. Instead of focusing on how to write prompts, it's about measuring their effectiveness through automated testing. You can:

- Test against expected answers
- Compare different versions of the same prompt
- Review outputs for errors

## Model based grading

### Types of Graders

**Code graders** - Programmatically evaluate outputs using custom logic
**Model graders** - Use another AI model to assess the quality
**Human graders** - Have people manually review and score outputs

### Code Graders

Code graders let you implement any programmatic check you can imagine. Common uses include:

- Checking output length
- Verifying output does/doesn't have certain words
- Syntax validation for JSON, Python, or regex
- Readability scores

The only requirement is that your code returns some usable signal - usually a number between 1 and 10.

### Model Graders

Model graders feed your original output into another API call for evaluation. This approach offers tremendous flexibility for assessing:

- Response quality
- Quality of instruction following
- Completeness
- Helpfulness
- Safety

### Human Graders

Human graders provide the most flexibility but are time-consuming and tedious. They're useful for evaluating:

- General response quality
- Comprehensiveness
- Depth
- Conciseness
- Relevance

### Defining Evaluation Criteria

Before implementing any grader, you need clear evaluation criteria. For a code generation prompt, you might focus on:

**Format** - Should return only Python, JSON, or Regex without explanation
**Valid Syntax** - Produced code should have valid syntax
**Task Following** - Response should directly address the user's task with accurate code
