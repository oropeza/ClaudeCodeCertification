
# MCP

MCP Servers provide access to data or functionality implemented by outside services. They act as specialized interfaces that expose tools, prompts, and resources in a standardized way.

- Client
- Server


Transport Agnostic
- Standard IO
- HTTP
- WebSockets
- Etc


Communication 
- ListToolsRequest and Response
- CallToolRequest and Response
- ReadResource Request and Response

Use the built-in MCP Inspector with `mcp dev mcp_server.py`

## MCP Server
- The Python MCP SDK makes server creation straightforward.
- MCP Inspector

## MCP Client
- Defining resource.
- Resources and resource templates
- Resources: Resources in MCP servers allow you to expose data to clients, similar to GET request handlers in a typical HTTP server
- Prompts: Prompts define a set of user and assistant messages that clients can use. They should be high-quality, well-tested, and relevant to your MCP server's purpose.


![](../Attachments/Pasted%20image%2020260331172423.png)




## MCP Server primitives

![](../Attachments/Pasted%20image%2020260331173936.png)

@mcp.tool(
@mcp.resource
@mcp.prompt


- **Need to give Claude new capabilities?** Use tools
- **Need to get data into your app for UI or context?** Use resources
- **Want to create predefined workflows for users?** Use prompts


# Core MCP Features



## Sampling
Sampling allows a server to access a language model like Claude through a connected MCP client. Instead of the server directly calling Claude, it asks the client to make the call on its behalf. This shifts the responsibility and cost of text generation from the server to the client.

## Log and progress notifications

## Roots

Roots are a way to grant MCP servers access to specific files and folders on your local machine.



# JSON Message types

## Messages Categories
### Request-Result Messages

These messages always come in pairs. You send a request and expect to get a result back:

- **Call Tool Request** → **Call Tool Result**
- **List Prompts Request → List Prompts Result**
- **Read Resource Request → Read Resource Result**
- **Initialize Request → Initialize Result**

### Notification Messages

These are one-way messages that inform about events but don't require a response:

- **Progress Notification** - Updates on long-running operations
- **Logging Message Notification** - System log messages
- **Tool List Changed Notification** - When available tools change
- **Resource Updated Notification** - When resources are modified



## MCP Transport

### Stdio transport
### StreamableHTTPTransport
- SSE Response - Server Sent Events Connection. Can be held open arbitrarily.

Flags
- stateless_http - horizontal scale
- json_response


stateless:
- get sse response can't be used
- no sampling, progress logging, subscriptions
- client init is no longer required

json_response
- post requests aren't streamed
- Just get final result
