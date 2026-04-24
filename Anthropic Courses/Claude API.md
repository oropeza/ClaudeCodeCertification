
# Claude API

## Model Family

![](../Attachments/Pasted%20image%2020260405183307.png)
## Claude processing

Tokenization, embedding, contextualization, and generation.

![](../Attachments/Pasted%20image%2020260404150450.png)

After each token, Claude checks several conditions to decide whether to continue:

- **Max tokens reached** - Has it hit the limit you specified?
- **Natural ending** - Did it generate an end-of-sequence token?
- **Stop sequence** - Did it encounter a predefined stop phrase?


## Create function

```
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "What is quantum computing? Answer in one sentence"
        }
    ]
)
```


- **model** - The name of the Claude model you want to use
- **max_tokens** - A safety limit on response length (not a target)
- **messages** - The conversation history you're sending to Claude

Response: `message.content[0].text`


**Claude doesn't store any of your conversation history**. Each request you make is completely independent, with no memory of previous exchanges.

This means if you want to have a multi-turn conversation where Claude remembers context from earlier messages, you need to handle the conversation state yourself.


### System Prompts

System prompts provide Claude with guidance on how to respond. You define them as plain strings and pass them into the create function call. The key benefits are:

- System prompts provide Claude guidance on how to respond
- Claude will try to respond in the same way someone in the specified role would respond
- Helps keep Claude on task

Here's the basic structure:

```
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""

client.messages.create(
    model=model,
    messages=messages,
    max_tokens=1000,
    system=system_prompt
)
```


### Temperature

Temperature is a decimal value between 0 and 1 that directly influences these selection probabilities. It's like adjusting the "creativity dial" on Claude's responses.

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623340%2F03_-_008_-_Temperature_06.1748623340446.png)

At low temperatures (near 0), Claude becomes very deterministic - it almost always picks the highest probability token. At high temperatures (near 1), Claude distributes probability more evenly across options, leading to more varied and creative outputs.

![](../Attachments/Pasted%20image%2020260404153758.png)



```
def chat(messages, system=None, temperature=1.0):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature
    }
    
    if system:
        params["system"] = system
    
    message = client.messages.create(**params)
    return message.content[0].text
```

### Response streaming
#### Stream Events

When you enable streaming, Claude sends back several types of events:

- **MessageStart** - A new message is being sent
- **ContentBlockStart** - Start of a new block containing text, tool use, or other content
- **ContentBlockDelta** - Chunks of the actual generated text.
- **ContentBlockStop** - The current content block has been completed
- **MessageDelta** - The current message is complete
- **MessageStop** - End of information about the current message


## Simplified Text Streaming

```
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="")
```


## Getting the Complete Message

```
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages
) as stream:
    for text in stream.text_stream:
        # Send each chunk to your client
        pass
    
    # Get the complete message for database storage
    final_message = stream.get_final_message()
```


### Structured data 

Assistant Message Prefilling + Stop Sequences

**Assistant Message Prefilling** refers to the practice of providing the AI assistant (like Claude) with a partial or template message as a starting point for its response. This “prefilled” message is included in the prompt sent to the model, and the model is expected to continue or complete it.

**Stop Sequences** are specific strings or tokens that tell the model when to stop generating text. When the model outputs a stop sequence, it halts further generation. 

# Prompt evaluation

![](../Attachments/Pasted%20image%2020260404191640.png)


# Prompt Engineering

The approach follows a clear cycle that you can repeat until you achieve your desired results:

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748623585%2F05_-_001_-_Prompt_Engineering_01.1748623585186.png)


* Being Clear and Direct
* Being specific: 
	* Qualities of output
	* Steps to follow
* Provide Structure
	* XML tags to separate portions of the prompt
* Provide examples 
	* Examples, input or output


# Tool Use


1. Tool function
2. Write JSON Schema
	1. name, description: what it does
	2. describe arguments for function
3. Call claude with JSON schema
4. Run Tool

### Multi-Block messages
- Text Block
- ToolUse Block
- ToolResult Block
	- is_error
* ServerToolUseBlock
* ServerToolResultBlock

To enable Claude to use tools, you need to include a `tools` parameter in your API call. Here's how to structure the request:

```
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

The `tools` parameter takes a list of JSON schemas that describe the available functions Claude can call.

**Why you need to keep the schema:**

When Claude sees a `tool_use` or `tool_result` block in the conversation history, it needs the tool schema to understand:

- What that tool was supposed to do
- What parameters it accepted
- The context of why it was called

**What about adding/removing tools then?**

You can still add and remove tools, but:

- **Keep schemas for any tool referenced in the conversation history**
- **Add new tools** as needed for new functionality
- **Remove tools** that have never been used in this conversation
### Follow-up Request

Your follow-up request to Claude must include the complete conversation history plus the new tool result. Here's the structure:

```
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

# Multi-Turn conversation with tools


**Building a Conversation Loop**

To handle this pattern, you need a conversation loop that continues until Claude stops requesting tools


```
def run_conversation(messages):
    while True:
        response = chat(messages)
        
        add_user_message(messages, response)
        
        # Pseudo code
        if response isn't asking for a tool:
            break
            
        tool_result_blocks = run_tools(response)
        add_user_message(tool_result_blocks)
        
    return messages
```

We will keep running conversations until Claude isn't asking for a tool use

stop_reason = tool_use


![](../Attachments/Pasted%20image%2020260404205642.png)

When using stream:
*FineToolCalling* will disable JSON validation in Claude.

## Claude Tools
- TextEditor Tool
- WebSearch Tool

# RAG - Retrieval Augmented Generation

Retrieval Augmented Generation (RAG) is a technique that helps you work with large documents that are too big to fit into a single prompt. Instead of cramming everything into one massive prompt, RAG breaks documents into chunks and only includes the most relevant pieces when answering questions.

Chunking strategies: 
- Size based
- Structure based
- Semantic Based


Semantic Search => Text embeddings
VoyageAI for Embeddings
Vector Databases
Searches similarity

BM25 Lexical search

![](../Attachments/Pasted%20image%2020260404221502.png)



# Features of Claude

## Extended thinking

Extended thinking is Claude's advanced reasoning feature that gives the model time to work through complex problems before generating a final response. Think of it as Claude's "scratch paper" - you can see the reasoning process that leads to the answer, which helps with transparency and often results in better quality responses.

- ThinkingBlock
- RedactedThinkingBlock

## Image support

* ImageBlock

## PDF Support

## Citations

## Prompt Caching

- Cache breakpoints

You can set multiple cache breakpoints in a single request. The order matters:

1. Tools (if provided)
2. System prompt (if provided)
3. Messages

## Code execution

ServerToolUseBlock
- name: code_execution
- type: server_tool_use


Key characteristics of the code execution environment:

- Runs in an isolated Docker container
- No network access (can't make external API calls)
- Claude can execute code multiple times during a single conversation
- Results are captured and interpreted by Claude for the final response


# MCP

![](../Attachments/Pasted%20image%2020260405132422.png)

# Anthropic Apps

- Claude Code
- Computer Use
- Agents


# Workflows and Agents

## When to Use Workflows vs Agents

![](https://everpath-course-content.s3-accelerate.amazonaws.com/instructor%2Fa46l9irobhg0f5webscixp0bs%2Fpublic%2F1748543103%2F11_-_001_-_Agents_and_Workflows_01.1748543103044.jpg)

The decision comes down to how well you understand the task:

- **Use workflows** when you can picture the exact flow or steps that Claude should go through to solve a problem, or when your app's UX constrains users to a set of tasks
- **Use agents** when you're not sure exactly what task or task parameters you'll give to Claude


Workflow:
- The Evaluator-Optimizer Pattern
- Parallelization
- Chaining Workflows
- Routing Workflows

Agents

Instead of defining a rigid sequence, you give Claude a goal and a set of tools, then let it figure out how to combine those tools to achieve the objective.

This flexibility makes agents attractive for building applications that need to handle varied, unpredictable tasks.

- Use tools in different combinations
- Tools should be abstract -> general

![](../Attachments/Pasted%20image%2020260405181202.png)



