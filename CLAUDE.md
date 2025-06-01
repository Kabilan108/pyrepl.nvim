# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pyrepl.nvim is a Neovim plugin that provides an interactive Python REPL experience by connecting to a local IPython server. The architecture consists of:

- **Neovim Plugin** (`lua/pyrepl.lua`): Lua module that communicates with the server via HTTP
- **CLI Wrapper** (`bin/pyrepl`): Bash script that manages server startup with package dependencies
- **Python Server** (`bin/server.py`): HTTP server that embeds IPython and executes code

## Development Commands

### Testing the Server
```bash
# Start the server with default settings
./bin/pyrepl

# Start with additional packages
./bin/pyrepl --with-pkgs numpy,pandas

# Start with logging enabled
./bin/pyrepl --log session-name

# Test server health
curl http://localhost:5000/health
```

### Testing the Neovim Plugin
The plugin requires the server to be running. Test by:
1. Starting the server: `./bin/pyrepl`
2. Opening Neovim in a Python file
3. Running `:lua require('pyrepl').setup()`
4. Selecting Python code and running `:RunInPyrepl`

## Architecture Notes

### Communication Flow
1. Neovim plugin captures visual selection via `get_visual_selection()`
2. Code is sent as JSON to `/execute` endpoint using plenary.curl
3. Server executes code in IPython shell with threading for non-blocking operation
4. Results are displayed in terminal and optionally logged to Markdown files

### Server State Management
- Uses `execution_lock` to prevent concurrent code execution
- Returns HTTP 409 when server is busy executing previous code
- `/reset` endpoint clears IPython namespace and execution state

### Key Dependencies
- **uv**: Python package manager for isolated environments
- **plenary.nvim**: Provides HTTP client functionality
- **mini.notify**: Handles user notifications
- **IPython**: Core REPL functionality with autoreload extension

### Configuration
- Server port configurable via `PYREPL_PORT` environment variable (default: 5000)
- Plugin respects `PYREPL_PORT` or uses port 5000
- Logging creates `.pyrepl/` directory in current working directory

## Development Environment

Uses Nix flake for reproducible development environment with Node.js 20 and automatic claude-code installation.

```bash
# Enter development shell
nix develop
```
