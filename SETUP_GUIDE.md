# Step-by-Step Setup Guide for Rhino MCP Server

## Prerequisites Checklist
- [x] Node.js installed
- [ ] Rhinoceros 3D installed on Windows
- [ ] Claude Desktop installed (or another MCP client)

## Step 1: Verify Installation

1. **Check Node.js is installed:**
   ```powershell
   node --version
   ```

2. **Verify dependencies are installed:**
   ```powershell
   npm install
   ```

3. **Build the project:**
   ```powershell
   npm run build
   ```

## Step 2: Verify Rhino is Installed

1. Make sure Rhinoceros 3D is installed on your system
2. You should be able to launch Rhino normally
3. The server will connect to Rhino via COM when needed

## Step 3: Configure Claude Desktop

### Find Claude Desktop Config File Location

The config file location depends on your OS:
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
  - Full path: `C:\Users\YOUR_USERNAME\AppData\Roaming\Claude\claude_desktop_config.json`

### Create/Edit the Config File

1. **Open the config file** (create it if it doesn't exist)
2. **Add this configuration:**

```json
{
  "mcpServers": {
    "rhino-mcp": {
      "command": "node",
      "args": [
        "C:\\Users\\aroncal\\Repos\\mcp_claude_rhino\\dist\\index.js"
      ]
    }
  }
}
```

**Important:** Replace the path with your actual path if different!

### Alternative: Use Relative Path (if Node is in PATH)

If you prefer, you can also use:
```json
{
  "mcpServers": {
    "rhino-mcp": {
      "command": "node",
      "args": [
        "C:\\Users\\aroncal\\Repos\\mcp_claude_rhino\\dist\\index.js"
      ],
      "env": {}
    }
  }
}
```

## Step 4: Restart Claude Desktop

1. **Close Claude Desktop completely**
2. **Reopen Claude Desktop**
3. The MCP server should start automatically when Claude Desktop launches

## Step 5: Verify It's Working

1. **Open Claude Desktop**
2. **Check the MCP connection:**
   - Look for any error messages in Claude Desktop
   - The server should connect automatically
3. **Test the tool:**
   - Ask Claude to use the `rhino_execute` tool
   - Example: "Can you create a point in Rhino at coordinates [0, 0, 0]?"

## Step 6: Testing the Server Manually (Optional)

If you want to test the server directly (without Claude Desktop):

1. **Open a terminal in the project directory**
2. **Run:**
   ```powershell
   npm run dev
   ```
3. **You should see:**
   ```
   Rhino MCP Server started and waiting for connections...
   Server name: rhino-server v1.0.0
   Registered tool: rhino_execute
   ```
4. **It will appear "stuck" - this is normal!** It's waiting for MCP protocol messages
5. **Press Ctrl+C to stop it**

## Troubleshooting

### Server won't start
- Check that Node.js is installed: `node --version`
- Verify the path in the config file is correct
- Make sure you ran `npm install` and `npm run build`

### Can't connect to Rhino
- Make sure Rhino 3D is installed
- Try launching Rhino manually first
- Check Windows COM registration

### Claude Desktop doesn't see the tool
- Restart Claude Desktop after editing the config
- Check the config file JSON syntax is valid
- Verify the path to `dist/index.js` is correct

### Path Issues
- Use double backslashes (`\\`) in JSON paths on Windows
- Or use forward slashes (`/`) - they work on Windows too
- Make sure the path is absolute (full path from C:\)

## Example Usage in Claude Desktop

Once configured, you can ask Claude:

- "Create a point in Rhino at [10, 20, 30]"
- "Draw a circle in Rhino with radius 5"
- "Execute this RhinoPython script: [your script here]"

Claude will automatically use the `rhino_execute` tool to interact with Rhino.

