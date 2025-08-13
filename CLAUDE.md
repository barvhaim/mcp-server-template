# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server template built with FastMCP and Python. The project provides a minimal foundation for creating MCP servers that can expose tools and resources to Claude and other AI assistants.

## Development Commands

- **Install dependencies**: `uv sync`
- **Run the server**: `uv run main.py` or `python main.py`
- **Format code**: `uv run black .`
- **Add dependencies**: `uv add <package>`
- **Add dev dependencies**: `uv add --dev <package>`

## Architecture

The project uses FastMCP, a Python framework for building MCP servers. Key patterns:

- **Server Definition**: The main server is defined in `main.py` using `FastMCP("Demo 🚀")`
- **Tool Registration**: Tools are registered using the `@mcp.tool` decorator
- **Entry Point**: The server runs via `mcp.run()` when executed directly

The current implementation includes a simple `add` tool that demonstrates the basic pattern for exposing functions as MCP tools.

## Dependencies

- **fastmcp**: Core MCP server framework (>=2.11.3)
- **python-dotenv**: Environment variable management (>=1.1.1)
- **black**: Code formatting (dev dependency, >=25.1.0)

## Development Setup

This project uses `uv` for dependency management and Python environment handling. The `uv.lock` file contains pinned versions of all dependencies.