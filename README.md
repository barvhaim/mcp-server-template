# MCP Server Template

A template for building Model Context Protocol (MCP) servers using FastMCP and Python.

## Overview

This template provides a minimal foundation for creating MCP servers that can expose tools and resources to Claude and other AI assistants. The server is built with FastMCP, a Python framework that simplifies MCP server development.

## Features

- Simple MCP server setup with FastMCP
- Streamable HTTP transport (runs on `127.0.0.1:5555`)
- Example tool implementation (`add` function)
- Modern Python dependency management with `uv`
- Code formatting with Black

## Quick Start

1. **Install dependencies**:
   ```bash
   uv sync
   ```

2. **Run the server**:
   ```bash
   uv run main.py
   ```

   The server will start on `http://127.0.0.1:5555` using the Streamable HTTP transport.

## Development

### Adding Tools

Tools are registered using the `@mcp.tool` decorator:

```python
@mcp.tool
def your_tool(param: str) -> str:
    """Tool description"""
    return f"Result: {param}"
```

### Commands

- **Install dependencies**: `uv sync`
- **Add dependencies**: `uv add <package>`
- **Format code**: `uv run black .`
- **Run server**: `uv run main.py`

## Dependencies

- **fastmcp** (>=2.11.3): Core MCP server framework
- **python-dotenv** (>=1.1.1): Environment variable management
- **black** (>=25.1.0): Code formatting (dev dependency)

## What is MCP?

Model Context Protocol (MCP) is a universal standard for connecting AI applications to external tools and data sources. Think of it as "USB-C for AI" - it solves the N×M complexity problem where every AI application needs custom integrations for every tool.

### Core Concepts

- **Tools**: Functions that AI can call to perform actions (like API endpoints)
- **Resources**: Read-only data sources that AI can access for context
- **Prompts**: Reusable interaction templates for common tasks
- **Transport**: STDIO (local), HTTP, or Streamable HTTP connections

## Connecting Clients

This server uses the **Streamable HTTP** transport, so clients connect over HTTP rather than spawning the process via STDIO. Start the server first (`uv run main.py`), then connect from your client.

### Claude Code (CLI)

Claude Code supports Streamable HTTP natively. Register the server with:

```bash
claude mcp add --transport http your-server-name http://127.0.0.1:5555/mcp
```

### Claude Desktop

Claude Desktop's `claude_desktop_config.json` does **not** natively support Streamable HTTP — it only spawns STDIO servers. Use the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge to proxy STDIO to your HTTP endpoint:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "npx",
      "args": ["mcp-remote", "http://127.0.0.1:5555/mcp"]
    }
  }
}
```

After configuration:
1. Make sure the server is running (`uv run main.py`)
2. Restart Claude Desktop completely
3. Your server tools will appear in the interface
4. Test by asking Claude to use your tools

### Other clients

Clients that support Streamable HTTP natively (Cursor, MCP Inspector, custom clients) can connect directly to `http://127.0.0.1:5555/mcp` without a bridge.

## Testing & Debugging

### MCP Inspector
Interactive web interface for testing your server. Start the server first, then point Inspector at the HTTP endpoint:

```bash
uv run main.py  # in one terminal
npx @modelcontextprotocol/inspector  # in another, then connect to http://127.0.0.1:5555/mcp
```

### Manual Testing
MCP uses JSON-RPC 2.0 as its message format across all transports. With Streamable HTTP, you send those JSON-RPC messages as HTTP POST bodies:

```bash
curl -X POST http://127.0.0.1:5555/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
```

### Common Issues

- **Server must be running** before clients can connect (HTTP transport, not STDIO)
- **Port conflicts**: change the `port` argument in `main.py` if `5555` is in use
- **Proper type hints** are required for all tools
- Check Claude Desktop logs: `~/Library/Logs/Claude/mcp.log` (macOS)

## Advanced Features

### Transport Configuration

This template runs with the Streamable HTTP transport:

```python
from fastmcp import FastMCP

mcp = FastMCP("Demo 🚀")

if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="127.0.0.1", port=5555)
```

To switch to STDIO (for direct Claude Desktop spawning), replace the `mcp.run(...)` call with `mcp.run()`.

### Adding Resources and Prompts

```python
@mcp.resource("config://server-info")
def get_server_info() -> dict:
    """Server information resource."""
    return {
        "name": "MyServer",
        "version": "1.0.0",
        "status": "running"
    }

@mcp.prompt()
def create_summary(text: str) -> str:
    """Generate summarization prompt."""
    return f"Please summarize this text:\n\n{text}\n\nSummary:"
```

## Project Structure

```
├── main.py          # Main server implementation
├── pyproject.toml   # Project configuration and dependencies
├── uv.lock         # Locked dependency versions
├── CLAUDE.md        # Claude Code guidance
└── README.md       # This file
```

## Resources

- **FastMCP Documentation**: [github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)
- **MCP Specification**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **uv Package Manager**: [astral.sh/uv](https://astral.sh/uv)
- **Claude Desktop**: [claude.ai](https://claude.ai)