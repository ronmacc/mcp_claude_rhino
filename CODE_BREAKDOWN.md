# Code Breakdown: Understanding Functions and Their Sources

## Import Statements (Lines 1-6)

### Line 1: `McpServer` from MCP SDK
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
```
- **Library**: `@modelcontextprotocol/sdk` (external npm package)
- **What it is**: A class for creating MCP servers
- **Used on**: Line 8 - `new McpServer({...})`

### Line 2: `StdioServerTransport` from MCP SDK
```typescript
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
```
- **Library**: `@modelcontextprotocol/sdk` (external npm package)
- **What it is**: A class for stdio (standard input/output) communication
- **Used on**: Line 76 - `new StdioServerTransport()`

### Line 3: `z` from Zod
```typescript
import { z } from "zod";
```
- **Library**: `zod` (external npm package - NOTE: missing from package.json!)
- **What it is**: Schema validation library
- **Used on**: Line 38 - `z.string()` (validates that input is a string)

### Line 4: `winax` (default import)
```typescript
import winax from 'winax';
```
- **Library**: `winax` (external npm package)
- **What it is**: Windows COM object library
- **Used on**: Line 26 - `new winax.Object('Rhino.Application')`

### Line 5: `fs` from Node.js
```typescript
import fs from 'fs';
```
- **Library**: Node.js built-in module (no npm install needed)
- **What it is**: File system operations
- **Used on**: 
  - Line 45 - `fs.existsSync(tempDir)` (check if directory exists)
  - Line 46 - `fs.mkdirSync(tempDir, { recursive: true })` (create directory)
  - Line 50 - `fs.writeFileSync(scriptFile, pythonScript)` (write file)
  - Line 55 - `fs.existsSync(scriptFile)` (check if file exists)
  - Line 56 - `fs.unlinkSync(scriptFile)` (delete file)

### Line 6: `path` from Node.js
```typescript
import path from 'path';
```
- **Library**: Node.js built-in module (no npm install needed)
- **What it is**: Path manipulation utilities
- **Used on**: 
  - Line 43 - `path.join(process.cwd(), "temp-scripts")` (join path segments)
  - Line 49 - `path.join(tempDir, ...)` (join path segments)

## Code Breakdown by Line

### Lines 8-11: Server Creation
```typescript
const server = new McpServer({
  name: "rhino-server",
  version: "1.0.0"
});
```
- **`new McpServer`**: From MCP SDK (line 1)
- **`{ name, version }`**: Plain JavaScript object (not from a library)

### Lines 13-34: Custom Function
```typescript
async function getRhinoInstance() {
  // Custom function - not from any library
  if (rhinoInstance) {
    try {
      rhinoInstance.Visible;  // Property access on COM object
      return rhinoInstance;
    } catch {
      rhinoInstance = null;
    }
  }
  
  try {
    rhinoInstance = new winax.Object('Rhino.Application');  // winax library
    rhinoInstance.Visible = true;  // COM object property
    await new Promise(resolve => setTimeout(resolve, 3000));  // Built-in JavaScript
    return rhinoInstance;
  } catch (error) {
    console.error('Failed to create Rhino instance:', error);  // Built-in Node.js
    throw error;  // Built-in JavaScript
  }
}
```
- **`getRhinoInstance`**: Custom function (defined in this file)
- **`new winax.Object`**: From winax library (line 4)
- **`new Promise`**: Built-in JavaScript
- **`setTimeout`**: Built-in JavaScript/Node.js
- **`console.error`**: Built-in Node.js
- **`rhinoInstance.Visible`**: Property of COM object (not a library function)

### Lines 35-74: Tool Registration
```typescript
server.registerTool("rhino_execute", {
  title: "Execute RhinoPython Script",
  description: "Execute Python script that interacts with Rhino",
  inputSchema: { pythonScript: z.string() }
}, async ({ pythonScript }) => {
  // Tool handler function
});
```
- **`server.registerTool`**: Method from MCP SDK (McpServer class)
- **`z.string()`**: From zod library (line 3)

### Inside Tool Handler (Lines 40-73):
```typescript
const rhino = await getRhinoInstance();  // Custom function (line 15)

const tempDir = path.join(process.cwd(), "temp-scripts");  // path library + process.cwd() (Node.js built-in)

if (!fs.existsSync(tempDir)) {  // fs library (Node.js built-in)
    fs.mkdirSync(tempDir, { recursive: true });  // fs library
}

const scriptFile = path.join(tempDir, `script_${Date.now()}.py`);  // path library + Date.now() (built-in JS)
fs.writeFileSync(scriptFile, pythonScript);  // fs library

const result = rhino.RunScript(`-_RunPythonScript "${scriptFile}"`, 0);  // COM object method (Rhino API)

setTimeout(() => {  // Built-in JavaScript
    if (fs.existsSync(scriptFile)) {  // fs library
        fs.unlinkSync(scriptFile);  // fs library
    }
}, 1000);

return { content: [{ type: "text", text: "..." }] };  // Plain JavaScript object
```

### Lines 76-77: Server Connection
```typescript
const transport = new StdioServerTransport();  // From MCP SDK (line 2)
await server.connect(transport);  // Method from MCP SDK (McpServer class)
```

## Summary: Function Sources

### External npm Packages:
- **MCP SDK**: `McpServer`, `StdioServerTransport`, `server.registerTool()`, `server.connect()`
- **winax**: `winax.Object()`
- **zod**: `z.string()` (but missing from package.json!)

### Node.js Built-in Modules:
- **fs**: `existsSync()`, `mkdirSync()`, `writeFileSync()`, `unlinkSync()`
- **path**: `join()`
- **process**: `process.cwd()` (global, not imported)

### Built-in JavaScript:
- `new Promise()`, `setTimeout()`, `Date.now()`, `console.error()`, `throw`, `await`, `async`

### Custom Code (This File):
- `getRhinoInstance()` function
- `rhinoInstance` variable
- `server` variable

### COM Object Methods (Rhino):
- `rhinoInstance.Visible` (property)
- `rhinoInstance.RunScript()` (method)

## How to Identify Function Sources

1. **Check the imports** (lines 1-6) - these tell you what external libraries are used
2. **Look for the library name prefix**:
   - `fs.` = Node.js file system
   - `path.` = Node.js path utilities
   - `winax.` = winax library
   - `z.` = zod library
   - `server.` = MCP SDK (McpServer instance)
3. **No prefix usually means**:
   - Built-in JavaScript (`setTimeout`, `Promise`, `Date`)
   - Node.js globals (`process`, `console`)
   - Custom code in this file
4. **Check package.json** to see what's installed
5. **Use IDE hover/IntelliSense** - hover over a function to see its source

