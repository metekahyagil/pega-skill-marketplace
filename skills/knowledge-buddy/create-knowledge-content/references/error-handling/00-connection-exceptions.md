# Step 0 Exceptions: Connection and Authentication Failures

This document covers error handling and troubleshooting for Step 0 (Verify Pega Connection).

**Happy Path**: [00-connection.md](../00-connection.md)

---

## Authentication Failures

### Problem

Authentication with Pega fails when calling `mcp__pega-dx-mcp__authenticate_pega({})`.

### Symptoms

- Error message about authentication failure
- Unable to obtain access token
- Connection refused or timeout errors
- OAuth2 authentication errors

### Solutions

#### 1. Check Environment Variables

Verify all required environment variables are set correctly:

**Required Variables:**
- `PEGA_BASE_URL` - Your Pega instance URL (e.g., `https://your-instance.pega.com`)
- `PEGA_CLIENT_ID` - OAuth2 client ID
- `PEGA_CLIENT_SECRET` - OAuth2 client secret

**How to Check:**
```bash
echo $PEGA_BASE_URL
echo $PEGA_CLIENT_ID
# Don't echo the secret for security
```

**Common Issues:**
- Variables not set or empty
- Incorrect URL format (missing https://, trailing slash, etc.)
- Expired or invalid credentials
- Copy-paste errors with extra spaces or special characters

#### 2. Verify Network Connectivity

Ensure you can reach the Pega instance:

**Tests:**
```bash
# Test basic connectivity
ping your-instance.pega.com

# Test HTTPS connectivity
curl -I https://your-instance.pega.com

# Check if DX API endpoint is accessible
curl https://your-instance.pega.com/prweb/api/v1/
```

**Common Issues:**
- Firewall blocking outbound HTTPS (port 443)
- VPN required but not connected
- Proxy configuration needed
- DNS resolution issues
- Instance URL incorrect or instance down

#### 3. Verify Pega Configuration

Ensure the Pega instance is configured correctly:

**DX API Must Be Enabled:**
- Check in Pega: Dev Studio → Integration-Resources → REST → Service Package "PegaDXAPI"
- Verify the service package is active and deployed
- Confirm the authentication service is running

**OAuth2 Configuration:**
- Client ID and secret must be registered in Pega
- Check in Pega: Dev Studio → Records → Security → OAuth 2.0 Client Registration
- Verify redirect URIs are configured if required
- Confirm token lifetime settings
- Check that the client has appropriate access rights

**Credentials Validity:**
- Credentials not expired
- Client not disabled or revoked
- Password/secret not changed recently
- Account not locked

#### 4. Check MCP Configuration

Verify the pega-dx-mcp server is properly installed and configured:

**Installation Check:**
```bash
# Check if installed globally
npm list -g @marco-looy/pega-dx-mcp

# Or check local installation
npm list @marco-looy/pega-dx-mcp
```

**MCP Settings File:**
Location: `~/.claude/mcp_settings.json` or project-specific

**Required Configuration:**
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

**Common Issues:**
- MCP server not installed
- Configuration file doesn't exist or wrong location
- JSON syntax errors in configuration
- Environment variables not set in MCP config
- Wrong command or args
- Claude Code not restarted after config changes

---

## Certificate and SSL Errors

### Problem

SSL/TLS certificate validation fails.

### Symptoms

- Certificate verification errors
- Self-signed certificate errors
- SSL handshake failures

### Solutions

#### For Self-Signed Certificates (Development Only)

⚠️ **Warning**: Only use for development/testing environments, never in production!

You may need to configure Node.js to accept self-signed certificates:

```bash
# Set environment variable (temporary, for testing only)
export NODE_TLS_REJECT_UNAUTHORIZED=0
```

**Better Solution**: Add the certificate to your system's trusted certificate store.

#### For Valid Certificate Issues

- Ensure system time is correct (certificate validation uses timestamps)
- Update your system's CA certificate bundle
- Check for intermediate certificates
- Verify the certificate matches the domain

---

## Timeout Errors

### Problem

Connection times out before completing authentication.

### Symptoms

- Timeout error messages
- No response from server
- Connection hanging

### Solutions

1. **Check Instance Availability**:
   - Is the Pega instance running?
   - Is it accessible from your network?
   - Are there scheduled maintenance windows?

2. **Network Performance**:
   - Test network latency to the instance
   - Check for network congestion
   - Verify bandwidth availability

3. **Increase Timeout** (if supported by MCP configuration):
   - Some network environments may need longer timeouts
   - Configure in MCP settings if available

---

## Permission Errors

### Problem

Authentication succeeds but access is denied.

### Symptoms

- 403 Forbidden errors
- "Access denied" messages
- Authentication works but API calls fail

### Solutions

1. **Verify User Permissions**:
   - Ensure the OAuth client has access to DX API
   - Check role and access group assignments
   - Verify case type access (PegaFW-KB-Work-Article)

2. **Check Application Access**:
   - Confirm access to Knowledge Buddy application
   - Verify ruleset access
   - Check data access restrictions

3. **Review Access Control Rules**:
   - When rules may block API access
   - Privilege requirements for operations
   - Access roles for content creation

---

## Troubleshooting Checklist

When authentication fails, verify in this order:

- [ ] Environment variables are set correctly
- [ ] Pega instance URL is accessible
- [ ] Network connectivity (no firewall/proxy issues)
- [ ] MCP server is installed
- [ ] MCP configuration file exists and is valid
- [ ] Claude Code has been restarted after config changes
- [ ] DX API is enabled in Pega instance
- [ ] OAuth2 client is registered in Pega
- [ ] Credentials are valid and not expired
- [ ] User/client has required permissions
- [ ] No certificate or SSL issues

---

## Important

**Only proceed with content creation if authentication succeeds.**

If you cannot resolve authentication issues:
1. Verify with Pega administrator that DX API is enabled
2. Confirm OAuth2 client credentials are correct
3. Test connectivity outside of Claude Code (e.g., with curl or Postman)
4. Check Pega system logs for authentication failures
5. Review MCP server logs for detailed error messages

---

## Related Documentation

- **Happy Path**: [00-connection.md](../00-connection.md)
- **Error Handling Index**: [index.md](./index.md)
- **Next Step**: [01-query-data.md](../01-query-data.md)
