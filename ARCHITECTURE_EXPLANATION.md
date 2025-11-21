# How This Repository Works: Architecture Explained

## Important: This is NOT a Traditional Web App!

This repository is **NOT** a front-end/back-end web application. Instead, it's an **MCP (Model Context Protocol) Server** - a special type of backend service that communicates with AI assistants.

## The Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  Claude Desktop │  <--->  │  This MCP Server │  <--->  │  Rhino 3D App   │
│   (The Client)  │  MCP    │  (This Repo)     │  COM    │  (3D Software)  │
│                 │ Protocol│                  │         │                 │
└─────────────────┘         └──────────────────┘         └─────────────────┘
     Front-End?                    Back-End                    External App
```

## What Each Component Does

### 1. **Claude Desktop** (The "Front-End" / Client)
- **What it is**: The user interface you interact with
- **Role**: MCP Client - it talks to MCP servers
- **What it does**:
  - You type messages to Claude
  - Claude decides when to use tools
  - Claude sends MCP protocol messages to this server
  - Claude displays results back to you

### 2. **This MCP Server** (The Back-End - This Repo)
- **What it is**: A Node.js server that implements the MCP protocol
- **Role**: MCP Server - provides tools to MCP clients
- **What it does**:
  - Listens for MCP protocol messages (via stdin/stdout)
  - Provides a tool called `rhino_execute`
  - When the tool is called, it connects to Rhino 3D
  - Executes Python scripts in Rhino
  - Returns results back to Claude Desktop

### 3. **Rhino 3D** (External Application)
- **What it is**: 3D modeling software
- **Role**: The actual application being controlled
- **What it does**:
  - Runs Python scripts via its COM interface
  - Creates 3D geometry, manipulates models, etc.

## How Data Flows

### Step-by-Step Example: "Create a point in Rhino at [0, 0, 0]"

1. **You** type in Claude Desktop: "Create a point in Rhino at [0, 0, 0]"

2. **Claude Desktop** analyzes your request and decides to use the `rhino_execute` tool

3. **Claude Desktop** sends an MCP protocol message to **This Server**:
   ```json
   {
     "method": "tools/call",
     "params": {
       "name": "rhino_execute",
       "arguments": {
         "pythonScript": "import rhinoscriptsyntax as rs\npoint = rs.AddPoint([0, 0, 0])"
       }
     }
   }
   ```

4. **This Server** receives the message and:
   - Connects to Rhino via COM (using `winax`)
   - Creates a temporary Python file
   - Executes the script in Rhino
   - Gets the result

5. **This Server** sends response back to **Claude Desktop**:
   ```json
   {
     "content": [{
       "type": "text",
       "text": "Python script executed successfully. Result: Script completed"
     }]
   }
   ```

6. **Claude Desktop** displays the result to **You**

## Key Concepts

### MCP (Model Context Protocol)
- **What**: A standardized protocol for AI assistants to interact with external tools
- **How**: Uses JSON-RPC messages over stdio (standard input/output)
- **Why**: Allows AI assistants to use tools without custom integrations

### stdio Transport
- **What**: Communication via standard input/output streams
- **How**: 
  - Server reads from `stdin` (standard input)
  - Server writes to `stdout` (standard output)
  - Errors/logs go to `stderr` (standard error)
- **Why**: Simple, universal, works with any process manager

### COM (Component Object Model)
- **What**: Windows technology for inter-process communication
- **How**: This server uses `winax` library to create COM objects
- **Why**: Rhino 3D exposes its functionality via COM interface

## Code Structure Breakdown

### `src/index.ts` - The Main Server File

```typescript
// 1. Create MCP Server
const server = new McpServer({ name: "rhino-server", version: "1.0.0" });

// 2. Register a Tool (rhino_execute)
server.registerTool("rhino_execute", {
  // Tool definition: name, description, input schema
}, async ({ pythonScript }) => {
  // Tool handler: what happens when tool is called
  // - Connect to Rhino
  // - Execute Python script
  // - Return result
});

// 3. Connect to stdio transport
const transport = new StdioServerTransport();
await server.connect(transport);
// Now server listens on stdin for MCP messages
```

## What's NOT in This Repo

- ❌ **No HTML/CSS/JavaScript front-end**
- ❌ **No web server (Express, etc.)**
- ❌ **No REST API endpoints**
- ❌ **No database**
- ❌ **No user interface**

## What IS in This Repo

- ✅ **MCP Server** (backend service)
- ✅ **Tool registration** (defines what tools are available)
- ✅ **Rhino COM integration** (connects to Rhino 3D)
- ✅ **Python script execution** (runs scripts in Rhino)

## Summary

**This repository is a backend service (MCP server) that:**
1. Provides tools to AI assistants (like Claude)
2. Acts as a bridge between Claude and Rhino 3D
3. Uses MCP protocol for communication (not HTTP/REST)
4. Uses stdio for transport (not web sockets or HTTP)
5. Controls Rhino 3D via COM interface

**The "front-end" is Claude Desktop** - it's a separate application that you configure to connect to this server.

Think of it like this:
- **Traditional Web App**: Browser ↔ HTTP ↔ Web Server ↔ Database
- **This MCP Setup**: Claude Desktop ↔ MCP Protocol ↔ This Server ↔ Rhino 3D

