# Step 0: Verify Pega Connection

Before starting the content creation workflow, you must authenticate and verify connection to your Pega instance.

## Quick Checklist

- [ ] Call `authenticate_pega` MCP tool
- [ ] Verify authentication succeeds (no errors)
- [ ] Proceed to Step 1 (Query Data)

---

## Authentication

Use the `authenticate_pega` MCP tool to validate connectivity:

```javascript
mcp__pega-dx-mcp__authenticate_pega({})
```

## What This Validates

- OAuth2 authentication is working
- Access token is obtained successfully
- Pega server is accessible
- MCP configuration is correct

## Important

**Only proceed with content creation if authentication succeeds.**

If authentication fails, resolve the configuration issues before continuing with the workflow.

## Next Step

After successful authentication, proceed to **[01-query-data.md](./01-query-data.md)** to discover available collections and data sources.

---

**Exceptions**: See **[error-handling/00-connection-exceptions.md](error-handling/00-connection-exceptions.md)** for authentication failures and troubleshooting.
