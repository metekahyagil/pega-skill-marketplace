# Step 0: Verify Pega Connection

## Purpose

Authenticate with Pega DX MCP server before starting content creation workflow.

## MCP Tool

```javascript
mcp__pega-dx-mcp__authenticate_pega({})
```

Ensure authentication succeeds. If it fails, see [error-handling/00-connection-exceptions.md](error-handling/00-connection-exceptions.md) for troubleshooting.

## Next Step

[01-query-data.md](01-query-data.md)
