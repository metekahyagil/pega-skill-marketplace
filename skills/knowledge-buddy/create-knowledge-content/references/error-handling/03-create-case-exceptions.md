# Step 3 Exceptions: Case Creation Failures

This document covers error handling and troubleshooting for Step 3 (Create Content Case).

**Happy Path**: [03-create-case.md](../03-create-case.md)

---

## Case Creation Failures

### Problem

Cannot create Content case using `mcp__pega-dx-mcp__create_case`.

### Symptoms

- `create_case` call fails with error
- Permission denied errors
- Case type not found errors
- Validation errors during case creation

### Expected Behavior

The Content case type (`PegaFW-KB-Work-Article`) accepts empty content during creation, so case creation should always succeed if authentication and permissions are valid.

---

## Solutions

### 1. Verify Authentication

**Issue**: Not authenticated or token expired.

**Check:**
```javascript
// Re-run Step 0 authentication
await mcp__pega-dx-mcp__authenticate_pega({});
```

**If this fails**: See [00-connection-exceptions.md](./00-connection-exceptions.md)

### 2. Check Permissions

**Issue**: User/client lacks permission to create cases.

**Required Permissions:**
- Access to Knowledge Buddy application
- Permission to create cases
- Access to PegaFW-KB-Work-Article case type
- Appropriate access group and role assignments

**How to Verify:**
1. Log into Pega with the same credentials
2. Try creating a case manually through the UI
3. Check Dev Studio → Security → Access Group
4. Verify case type access in access group settings

**Common Permission Issues:**
- User not assigned to appropriate access group
- Access group doesn't include Knowledge Buddy ruleset
- When rules restricting case creation
- Privilege level too low for case operations

### 3. Verify Case Type Availability

**Issue**: Case type `PegaFW-KB-Work-Article` not available in the instance.

**Check:**
1. Knowledge Buddy application must be installed
2. Case type must exist in the ruleset
3. Case type must be in an unlocked ruleset version

**How to Verify in Pega:**
- Dev Studio → Case Types → Search for "PegaFW-KB-Work-Article"
- Verify the case type exists and is not disabled
- Check the circumstancing and availability conditions

**If Case Type Missing:**
- Install/enable Knowledge Buddy application
- Verify correct application version is deployed
- Check that required rulesets are available
- Contact Pega administrator to enable the application

### 4. Instance Configuration Issues

**Issue**: Pega instance configuration problems.

**Common Issues:**
- Instance is in read-only mode (maintenance mode)
- Database connection issues
- System resources exhausted (memory, connections)
- Application rules not deployed correctly

**How to Check:**
- Check Pega system status in Dev Studio
- Review system alerts and warnings
- Check database connectivity
- Verify application deployment status

**Temporary Issues:**
- Wait and retry if instance is temporarily unavailable
- Check for scheduled maintenance windows
- Monitor system resource usage

---

## Validation Errors

### Problem

Case creation rejected due to validation errors.

### Symptoms

- Validation error messages
- Required field errors
- Business rule violations

### Solutions

#### Empty Content is Valid

The `PegaFW-KB-Work-Article` case type is designed to accept empty content during creation:

```javascript
// This should always work
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}  // Empty content is valid
})
```

#### If You're Passing Content

If you're trying to pass initial content (not recommended for this step):
- Ensure property names are correct
- Check data types match expectations
- Verify required fields if any
- Remove invalid or unknown properties

**Recommendation**: Always use empty content `{}` in Step 3. Configure the case in Step 4 instead.

---

## API or Network Errors

### Problem

API call fails due to network or server issues.

### Symptoms

- Timeout errors
- Connection refused
- 500 Internal Server Error
- Network unavailable

### Solutions

#### 1. Network Connectivity

- Verify network connection is active
- Check if Pega instance is reachable
- Test with ping or curl
- Verify no firewall blocking requests

#### 2. Server Issues

- Pega instance may be down or restarting
- Check instance health/status page
- Review Pega system logs
- Contact Pega administrator

#### 3. Retry Strategy

For transient issues:
```javascript
// Retry with exponential backoff
async function createCaseWithRetry(maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await mcp__pega-dx-mcp__create_case({
        caseTypeID: "PegaFW-KB-Work-Article",
        content: {}
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

---

## Case Type Configuration Issues

### Problem

Case type exists but has configuration problems.

### Symptoms

- Case creates but immediately errors
- Case creates with wrong status
- Assignment not created correctly

### Solutions

#### Verify Case Type Configuration

**Check in Pega:**
1. Case type stages are configured correctly
2. Initial stage is "Create" (PRIM0)
3. CreateForm_Default process exists
4. Assignment shape is configured in the process

**Common Configuration Issues:**
- Missing or disabled create stage
- Process flow errors
- Assignment configuration missing
- Stage transition rules incorrect

#### Ruleset and Version Issues

- Case type must be in available ruleset
- Ruleset version must be unlocked (for development)
- No circumstancing blocking case creation
- No availability conditions preventing access

---

## Troubleshooting Checklist

When case creation fails, verify:

- [ ] Authentication successful (Step 0 completed)
- [ ] User has permission to create cases
- [ ] Case type PegaFW-KB-Work-Article exists
- [ ] Knowledge Buddy application is installed
- [ ] Using empty content `{}` in create_case call
- [ ] Pega instance is accessible and healthy
- [ ] No network connectivity issues
- [ ] Access group includes required rulesets
- [ ] Case type configuration is correct
- [ ] No system maintenance or read-only mode

---

## What to Do If All Checks Pass

If case creation still fails after verifying all items:

1. **Test manually in Pega UI**:
   - Log into Pega with same credentials
   - Try creating the case through Dev Studio or portal
   - Note any error messages

2. **Check Pega logs**:
   - Review application logs for exceptions
   - Check tracer in Dev Studio
   - Look for validation or permission errors

3. **Verify DX API access**:
   - Test other DX API operations
   - Confirm DX API is fully functional
   - Check API version compatibility

4. **Contact administrator**:
   - Provide error messages
   - Share authentication details (not secrets!)
   - Request verification of permissions and configuration

---

## Important Notes

- Case creation with empty content should always succeed if permissions are correct
- Don't try to configure Collection/Datasource in Step 3 - wait for Step 4
- The case will be in Create stage (PRIM0) after creation
- You must capture both caseID and assignmentID for next steps
- If creation succeeds but returns unexpected data, continue to Step 4 and verify later

---

## Related Documentation

- **Happy Path**: [03-create-case.md](../03-create-case.md)
- **Authentication Issues**: [00-connection-exceptions.md](./00-connection-exceptions.md)
- **Next Step**: [04-configure-case.md](../04-configure-case.md)
- **Error Handling Index**: [index.md](./index.md)
