# Step 0 Exceptions: Connection Failures

## Authentication Errors

### Symptom
`authenticate_pega` fails with OAuth2 or connection errors.

### Common Causes

**Environment Variables Missing/Invalid**:
- `PEGA_BASE_URL`: Pega instance URL (e.g., `https://instance.pega.com`)
- `PEGA_CLIENT_ID`: OAuth2 client ID
- `PEGA_CLIENT_SECRET`: OAuth2 client secret

**MCP Configuration Issues**:
- Check `~/.claude/mcp_settings.json` has correct config:
```json
{
  "mcpServers": {
    "pega-dx-mcp": {
      "command": "npx",
      "args": ["-y", "@marco-looy/pega-dx-mcp"],
      "env": {
        "PEGA_BASE_URL": "https://your-instance.pega.com",
        "PEGA_CLIENT_ID": "your-client-id",
        "PEGA_CLIENT_SECRET": "your-client-secret"
      }
    }
  }
}
```
- Restart Claude Code after config changes

**Pega Instance Issues**:
- DX API not enabled (check Dev Studio → Integration-Resources → REST → PegaDXAPI)
- OAuth2 client not registered (Dev Studio → Records → Security → OAuth 2.0 Client Registration)
- Credentials expired or revoked
- Instance unreachable (firewall, VPN, DNS issues)

### Fix

1. Verify environment variables set
2. Test connectivity: `curl https://your-instance.pega.com/prweb/api/v1/`
3. Confirm MCP config valid JSON
4. Restart Claude Code
5. Verify DX API enabled in Pega
6. Check OAuth2 client registration

## Permission Errors

### Symptom
Authentication succeeds but API calls return 403 Forbidden.

### Fix
- Verify OAuth client has DX API access
- Check user access to Knowledge Buddy application
- Confirm access to `PegaFW-KB-Work-Article` case type

## Troubleshooting Checklist

- [ ] Environment variables set
- [ ] MCP config exists and valid
- [ ] Claude Code restarted
- [ ] Instance URL accessible
- [ ] DX API enabled in Pega
- [ ] OAuth2 client registered
- [ ] Credentials valid

Only proceed if authentication succeeds.

**Happy Path**: [00-connection.md](../00-connection.md)
