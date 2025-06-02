# QUESTION: I have heard that SSE is being used in MCP. Can you tell me more in this context? How do MCP client and server talks with each other?

The Model Context Protocol (MCP) uses Server-Sent Events (SSE) as one of its transport mechanisms to enable real-time communication between AI applications and external data sources or tools. However, it's important to note that **SSE transport in MCP has been deprecated as of March 26, 2025, in favor of the new HTTP Stream Transport**[2].

## How MCP Works with SSE

MCP over SSE implements a **hybrid communication pattern** that overcomes SSE's unidirectional limitation by combining it with HTTP POST requests. The architecture uses two separate endpoints to achieve bidirectional communication[3]:

**SSE Endpoint (`/sse`)**: Used for server-to-client communication
**Messages Endpoint (`/messages`)**: Used for client-to-server communication via HTTP POST

## Communication Flow

### Initial Connection Setup

When an MCP client connects to an SSE-based server, the following sequence occurs[3]:

1. **Client connects** to the `/sse` endpoint on the server
2. **Server responds** with an "endpoint event" telling the client what URI (e.g., `/messages?client_id=123`) to use for sending messages back
3. **Client stores** this message endpoint for future communication
4. **Server maintains** the SSE connection for streaming events and messages to the client

```javascript
// Client establishes SSE connection
const eventSource = new EventSource('http://localhost:8080/sse');

// Handle endpoint URL from server
eventSource.addEventListener('endpoint', (event) => {
    const messageEndpoint = event.data;
    // Store messageEndpoint for sending messages
});

// Handle incoming messages
eventSource.addEventListener('message', (event) => {
    const message = JSON.parse(event.data);
    // Process MCP message
});
```

### Bidirectional Communication Pattern

**Server-to-Client (via SSE)**:
- Real-time tool responses
- Resource updates
- Protocol notifications
- Error messages

**Client-to-Server (via HTTP POST)**:
- Tool execution requests
- Resource queries
- Protocol initialization messages

```python
# Server-side endpoints
@app.get("/sse")
async def sse_endpoint(request: Request):
    endpoint_event = f"event: endpoint\ndata: /messages?client_id={client_id}\n\n"
    return StreamingResponse(
        endpoint_event,
        media_type="text/event-stream",
        headers={
            'Cache-Control': 'no-cache',
            'Connection': 'keep-alive',
        }
    )

@app.post("/messages")
async def receive_message(request: Request):
    data = await request.json()
    client_id = request.query_params.get("client_id")
    # Process MCP JSON-RPC message
    return {"jsonrpc": "2.0", "id": data.get("id"), "result": {"status": "received"}}
```

## MCP Protocol Over SSE

MCP uses **JSON-RPC 2.0** as its underlying protocol, with three main message types[7]:

- **Requests**: Messages to initiate operations (e.g., tool calls)
- **Responses**: Replies to requests
- **Notifications**: One-way messages that don't require responses

### Example Tool Execution Flow

1. **Client sends HTTP POST** to `/messages` with a tool call request:
```json
{
    "jsonrpc": "2.0",
    "id": "req-001",
    "method": "tools/call",
    "params": {
        "name": "get_weather",
        "arguments": {
            "city": "Sydney"
        }
    }
}
```

2. **Server processes** the request and sends response via SSE:
```json
{
    "jsonrpc": "2.0",
    "id": "req-001",
    "result": {
        "content": [
            {
                "type": "text",
                "text": "The weather in Sydney is sunny, 24°C"
            }
        ]
    }
}
```

## Implementation Architecture

MCP servers using SSE typically implement **asynchronous streams**[6]:

- **Instream**: Receives messages from clients (via HTTP POST to `/messages`)
- **Outstream**: Sends JSON-RPC responses and events to clients (via SSE)

This creates a **two-way communication channel** where SSE handles server-to-client streaming and HTTP POST handles client-to-server requests[6].

## Limitations and Deprecation

The SSE transport approach had several limitations that led to its deprecation[3]:

- **Dual connection complexity**: Requires maintaining two separate connections and endpoints
- **Stateful nature**: Makes stateless implementations difficult
- **Infrastructure compatibility**: Limited compatibility with some middleware and proxies
- **Connection resilience**: Lacks resumability in case of network disruptions

These challenges made SSE-based MCP servers difficult to deploy and maintain in production environments, especially for remote servers accessible over the internet. The new HTTP Stream Transport addresses these limitations while maintaining the real-time capabilities that MCP requires.

Despite its deprecation, understanding SSE in MCP context provides valuable insight into how real-time AI tool communication protocols evolved and the engineering challenges involved in creating robust client-server architectures for AI applications.

Citations:
[1] https://github.com/yunusemredilber/express-mcp-sse-server
[2] https://mcp-framework.com/docs/Transports/sse/
[3] https://blog.christianposta.com/ai/understanding-mcp-recent-change-around-http-sse/
[4] https://auth0.com/blog/an-introduction-to-mcp-and-authorization/
[5] https://www.youtube.com/watch?v=kJ6EbcWvgYU
[6] https://www.youtube.com/watch?v=Oddi6ihrTJI
[7] https://betterstack.com/community/guides/ai/mcp-explained/
[8] https://mcp.so/server/MCP-SSE-Server-Sample/edom18
[9] https://modelcontextprotocol.io/docs/concepts/transports
[10] https://github.com/sidharthrajaram/mcp-sse
[11] https://learn.microsoft.com/en-us/microsoftteams/platform/teams-ai-library/csharp/in-depth-guides/ai/mcp/mcp-client
[12] https://docs.chainlit.io/advanced-features/mcp
[13] https://mcpmarket.com/server/browser-use-14
[14] https://mcpmarket.com/server/fastapi-sse
[15] https://webdeveloper.beehiiv.com/p/how-server-sent-events-work
[16] https://www.securityhq.com/services/managed-security-service-edge-sse/
[17] https://www.maximilian-schwarzmueller.com/articles/server-sent-events-sse-the-champion-no-one-knows
[18] https://www.zscaler.com/resources/security-terms-glossary/what-is-security-service-edge-sse
[19] https://workos.com/blog/how-mcp-servers-work

---
Answer from Perplexity: pplx.ai/share