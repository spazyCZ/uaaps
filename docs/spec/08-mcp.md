## 9. MCP Server Configuration

Model Context Protocol servers extend agent capabilities with external tool access.

### Universal `servers.json` Format

```json
{
  "servers": {
    "my-service": {
      "command": "node",
      "args": ["${PACKAGE_ROOT}/mcp-server/index.js"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

### `servers.json` Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `servers` | `map<string, ServerDef>` | **Yes** | Map of server name to server definition. Server names MUST be `[a-z0-9-]`. |
| `servers.<name>.command` | `string` | **Yes** | Executable used to launch the MCP server process. |
| `servers.<name>.args` | `string[]` | No | Arguments passed to the command. |
| `servers.<name>.env` | `map<string, string>` | No | Environment variables injected into the server process. Use `${VAR}` syntax to reference host-environment variables (e.g. `${API_KEY}`). |
| `servers.<name>.type` | `string` | No | Transport type. Permitted values: `stdio` (default), `http`. |

### Platform Mapping

| Platform | Plugin Location | Project Location | Global Location | Root Variable |
|----------|----------------|-----------------|----------------|---------------|
| Claude Code | `.mcp.json` at plugin root | `.mcp.json` at project root | `~/.claude/.mcp.json` | `${CLAUDE_PLUGIN_ROOT}` |
| Cursor | `mcp.json` at plugin root | `.cursor/mcp.json` | `~/.cursor/mcp.json` | Working dir relative |
| Universal | `mcp/servers.json` | `mcp/servers.json` | N/A | `${PACKAGE_ROOT}` |

> **Critical**: Cursor plugin MCP uses `mcp.json` (no dot prefix, no `.cursor/` wrapper). Project/global MCP uses `.cursor/mcp.json`. Claude Code always uses `.mcp.json` (leading dot).
