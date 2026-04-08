# Cheat Sheet: Getting Started with MCP

**Estimated time needed:** 10 minutes

This module explored the basics of creating and configuring MCP servers. Use this cheat sheet to quickly review and refresh the concepts you learned.

## MCP Basics

| Topic | Description |
|---|---|
| What is MCP? | Model Context Protocol (MCP) is an open-source standard for enabling AI applications with the capabilities of external systems. Using MCP, large language models (LLMs) like ChatGPT can execute tasks like sending an email, retrieve data like reading from the local file system, and perform specialized workflows like iterative code review. |
| Analogy | A simple analogy for MCP is to think of it like a USB port for AI applications. MCP provides a standardized interface that lets AI applications connect with external systems, just like how USB offers a universal way for devices to connect and communicate. |
| Why MCP? | **Standardized integration:** MCP sets a standard protocol for connecting AI to external systems, making development more consistent, understandable, and scalable. **Interoperability:** It supports many clients and AI models and is modular. **Security:** It uses OAuth 2.0 and token-based authentication. **Minimizes hallucinations:** It connects to external systems that stay up to date, helping provide cleaner and more accurate data. **Agentic workflow support:** It enables AI agents to perform multi-step, dynamic tasks. |

## MCP Architecture

| Topic | Description |
|---|---|
| Three-layer architecture | MCP uses bidirectional communication with three message types in JSON-RPC 2.0. These messages are sent over STDIO or HTTP, which are transports you will learn more about in the next module. |
| Initialization | **1. Initialization**: The client sends an `initialize` request with protocol version and capabilities. The server responds with its capabilities. The client sends an `initialized` notification to complete the handshake. **2. Operation**: Bidirectional communication begins. The client discovers and invokes tools, resources, and prompts provided by the server. The server can also send notifications, logging messages, and other requests to the client. **3. Shutdown**: The client sends a `shutdown` request, the server responds, and the connection closes. |
| Multi-server client | The host process manages multiple MCP client instances. Each client handles its own session with a server, including lifecycle and capabilities. The host process aggregates capabilities and routes tool results to the correct client. |

## Interaction with an MCP Server

Use the FastMCP library to create a simple client that can call tools from an MCP server.

First, import the client and transport classes:

```python
from fastmcp import Client
from fastmcp.client.transports import StreamableHttpTransport, StdioTransport
Then create the transports. You can use either STDIO or HTTP:

stdio_transport = StdioTransport(
    command="npx",
    args=["-y", "@upstash/context7-mcp"]
)

http_transport = StreamableHttpTransport(
    url="https://mcp.context7.com/mcp"
)
Here, both transports connect to the same Context7 MCP server. They are configurable to connect to any MCP server that supports the given transport type.

Next, create a client for each transport:

stdio_client = Client(stdio_transport)
http_client = Client(http_transport)
Finally, open an asynchronous context manager for the client and call tools using client.list_tools().

This lists all tools with their name, description, and metadata, including the input schema.

You can also use client.call_tool("tool_name", {"key": "value"}) to call a tool with input parameters.

Example using STDIO
async with stdio_client as client:
    tools = await client.list_tools()
    response = await client.call_tool(
        "resolve-library-id",
        {"libraryName": "fastmcp"}
    )
Example using HTTP
async with http_client as client:
    tools = await client.list_tools()
    response = await client.call_tool(
        "resolve-library-id",
        {"libraryName": "fastmcp"}
    )
