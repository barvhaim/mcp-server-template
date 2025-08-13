# MCP Server Template

A template for building Model Context Protocol (MCP) servers using FastMCP and Python.

## Overview

This template provides a minimal foundation for creating MCP servers that can expose tools and resources to Claude and other AI assistants. The server is built with FastMCP, a Python framework that simplifies MCP server development.

## Features

- Simple MCP server setup with FastMCP
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

## Claude Desktop Integration

To connect your MCP server to Claude Desktop, add this to your `claude_desktop_config.json`:

**macOS/Linux**: `~/.config/claude/claude_desktop_config.json`  
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "uv",
      "args": [
        "--directory",
        "/absolute/path/to/your/server",
        "run",
        "main.py"
      ]
    }
  }
}
```

After configuration:
1. Restart Claude Desktop completely
2. Your server tools will appear in the interface
3. Test by asking Claude to use your tools

## Testing & Debugging

### MCP Inspector
Interactive web interface for testing your server:

```bash
npm install -g @modelcontextprotocol/inspector
npx @modelcontextprotocol/inspector main.py
```

### Manual Testing
Test your server directly with JSON-RPC:

```bash
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | uv run main.py
```

### Common Issues

- **Use absolute paths** in Claude Desktop config
- **Never use print()** in STDIO mode (use logging to stderr)
- **Proper type hints** are required for all tools
- Check Claude Desktop logs: `~/Library/Logs/Claude/mcp.log` (macOS)

## Advanced Features

### Multiple Transport Types

```python
import os
from fastmcp import FastMCP

mcp = FastMCP("MyServer")

if __name__ == "__main__":
    transport = os.getenv("MCP_TRANSPORT", "stdio")
    
    if transport == "http":
        # HTTP transport for web integration
        import uvicorn
        uvicorn.run(mcp.create_fastapi_app(), host="0.0.0.0", port=8000)
    else:
        # Default STDIO for Claude Desktop
        mcp.run()
```

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