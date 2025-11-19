# rhino-mcp

A Model Context Protocol (MCP) server that enables interaction with Rhinoceros 3D software through Python scripts. This server allows you to execute RhinoPython scripts programmatically via the MCP protocol.

## What This Project Does

This MCP server provides a bridge between MCP-compatible clients (like AI assistants) and Rhinoceros 3D. It allows you to:
- Execute Python scripts that interact with Rhino 3D
- Automate Rhino operations through the MCP protocol
- Control Rhino programmatically from external applications

The server creates a COM connection to Rhino using the `winax` library, executes Python scripts in Rhino's Python environment, and manages temporary script files.

## Dependencies

### Runtime Dependencies
- **@modelcontextprotocol/sdk** (^1.13.2): The Model Context Protocol SDK for building MCP servers. Provides the server infrastructure, tool registration, and stdio transport.
- **winax** (^3.6.2): A Node.js library for creating and interacting with Windows COM objects. Used to connect to Rhino's COM interface.

### Development Dependencies
- **typescript** (^5.8.3): TypeScript compiler for type-safe JavaScript development
- **@types/node** (^24.0.8): TypeScript type definitions for Node.js

## How It Works

1. **Server Initialization**: The server starts as an MCP server named "rhino-server" using stdio transport for communication.

2. **Rhino Connection**: When a tool is called, the server attempts to connect to Rhino via COM:
   - Creates a `Rhino.Application` COM object using `winax`
   - Sets Rhino to visible mode
   - Maintains a singleton instance that is reused across requests

3. **Script Execution**:
   - Receives Python script content via the `rhino_execute` tool
   - Creates a temporary script file in the `temp-scripts` directory
   - Executes the script in Rhino using the `-_RunPythonScript` command
   - Cleans up the temporary file after execution

4. **Tool Registration**: The server registers one tool:
   - `rhino_execute`: Executes a Python script in Rhino and returns the result

## Prerequisites

- **Node.js**: Required to run the server (version compatible with ES2022 modules)
- **Rhinoceros 3D**: Must be installed on your Windows system
- **Windows OS**: Required for COM object support (winax only works on Windows)

## Installation

1. Install dependencies:
```bash
npm install
```

2. Build the TypeScript code:
```bash
npm run build
```

## Running the Server

### Development Mode
Run the server directly with TypeScript (requires `tsx`):
```bash
npm run dev
```

### Production Mode
Build first, then run:
```bash
npm run build
npm start
```

### As a Binary
After building, the server can be run as:
```bash
node dist/index.js
```

## Usage with MCP Clients

To use this server with an MCP client (like Claude Desktop or other MCP-compatible applications), you need to configure the client to connect to this server via stdio.

Example configuration for Claude Desktop (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "rhino-mcp": {
      "command": "node",
      "args": ["path/to/rhino-mcp/dist/index.js"]
    }
  }
}
```

## Available Tools

### rhino_execute
Executes a Python script in Rhino 3D.

**Input Schema:**
- `pythonScript` (string): The Python script to execute in Rhino

**Example Usage:**
```python
import rhinoscriptsyntax as rs

# Create a point at origin
point = rs.AddPoint([0, 0, 0])
print(f"Created point: {point}")
```

## Project Structure

```
rhino-mcp/
├── src/
│   └── index.ts          # Main server implementation
├── dist/                  # Compiled JavaScript output
├── temp-scripts/          # Temporary Python script storage
├── package.json           # Project dependencies and scripts
├── tsconfig.json          # TypeScript configuration
└── README.md              # This file
```

## Scripts

- `npm run build`: Compiles TypeScript to JavaScript in the `dist` directory
- `npm run build:watch`: Watches for changes and rebuilds automatically
- `npm run dev`: Runs the server in development mode using tsx
- `npm start`: Runs the compiled server from `dist/index.js`
- `npm test`: Placeholder test script (currently not implemented)

## Notes

- The server requires Rhino to be installed and accessible via COM
- Scripts are temporarily stored in `temp-scripts/` and cleaned up after execution
- The server maintains a single Rhino instance connection that is reused
- If the Rhino connection is lost, the server will attempt to reconnect on the next request
- This server is Windows-only due to COM object requirements
