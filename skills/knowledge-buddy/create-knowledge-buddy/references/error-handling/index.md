# Error Handling Guide

This directory contains detailed error handling documentation for the Knowledge Buddy creation workflow. Each file corresponds to a specific step in the workflow and documents common issues, their causes, and solutions.

## Overview

The Knowledge Buddy creation workflow consists of 6 steps, each with potential error scenarios:

1. **Connection** - Verify Pega DX MCP server connection
2. **Case Creation** - Initialize Knowledge Buddy case
3. **Basic Information** - Submit buddy name and description
4. **Security Configuration** - Configure access control roles
5. **Prompt Configuration** - Set up GenAI prompts and context data
6. **Verification** - Confirm successful buddy creation

## Error Handling Files

Each step has a dedicated error handling document:

### [00-connection-exceptions.md](00-connection-exceptions.md)
Covers connection and authentication issues:
- Authentication failures
- MCP server configuration problems
- Wrong Pega instance
- Invalid credentials

### [01-create-case-exceptions.md](01-create-case-exceptions.md)
Covers case creation issues:
- Case type not found (404)
- Permission errors (403)
- Missing Knowledge Buddy application
- Response parsing errors

### [02-submit-basic-info-exceptions.md](02-submit-basic-info-exceptions.md)
Covers basic information submission issues:
- Required field validation (Name)
- Duplicate buddy names
- Assignment errors
- Stage transition problems

### [03-configure-security-exceptions.md](03-configure-security-exceptions.md)
Covers security configuration issues:
- Invalid role names
- pageInstructions errors (missing dot prefix)
- listIndex errors (0-based vs 1-based)
- Access configuration problems

### [04-configure-prompts-exceptions.md](04-configure-prompts-exceptions.md)
Covers prompt and context data configuration issues:
- Empty collection references
- Invalid placeholder usage
- pageInstructions errors
- ContextDataConfig structure validation
- MinSimilarityScore and MaxChunk settings

### [05-verify-success-exceptions.md](05-verify-success-exceptions.md)
Covers verification and testing issues:
- Status not Resolved-Completed
- Configuration verification problems
- Case retrieval issues
- Buddy testing problems (no results, incorrect responses)
- Access denied errors

## Common Error Patterns

### PageInstructions Errors

**Most common mistake**: Missing dot prefix on target paths

```javascript
// WRONG
target: "BuddyAccessConfigurations"
target: "ContextDataConfigs"

// CORRECT
target: ".BuddyAccessConfigurations"
target: ".ContextDataConfigs"
```

**Second most common**: Using 0-based index instead of 1-based

```javascript
// WRONG - listIndex is 1-based
listIndex: 0  // ❌

// CORRECT
listIndex: 1  // First item
listIndex: 2  // Second item
```

### Role and Field Validation

**Use exact names from Pega**:
- Role names are case-sensitive
- Field names must match exactly
- Query data views to get valid values

### API Response Handling

**Always check responses**:
- Verify status codes (200, 400, 403, 404)
- Check for error messages in response body
- Extract required values before proceeding
- Handle missing or null values gracefully

## Troubleshooting Workflow

When encountering an error:

1. **Identify the step**: Determine which workflow step failed
2. **Check the exception document**: Review the relevant error handling file
3. **Verify prerequisites**: Ensure previous steps completed successfully
4. **Review request format**: Check API call structure and parameters
5. **Examine response**: Look for error messages and status codes
6. **Apply solution**: Follow documented solutions for the error
7. **Retry or proceed**: Either retry the failed step or move forward

## General Troubleshooting Tips

### Check MCP Connection First
Always verify connection before starting workflow:
```javascript
mcp__pega-dx-mcp__authenticate_pega({})
```

### Use get_case for Debugging
Check current case state at any point:
```javascript
mcp__pega-dx-mcp__get_case({
  caseID: "PEGAFW-QNA-WORK BUDDY-####"
})
```

### Query Available Options
Always query data views before asking user:
- Collections: `D_IndexList`
- Roles: `D_BuddyAccessRoleList`
- Cases: `D_BuddyList`

### Validate Before Submitting
- Check required fields are present
- Verify field values are valid
- Ensure pageInstructions format is correct
- Confirm assignment IDs are accurate

### Log Everything
- Capture full API responses
- Log extracted values
- Document errors with context
- Save case IDs for reference

## Error Prevention Best Practices

1. **Query first, ask second**: Fetch available options before user input
2. **Validate inputs**: Check user inputs before API calls
3. **Use exact values**: Copy values from query responses exactly
4. **Handle empty cases**: Provide defaults when queries return nothing
5. **Confirm success**: Always verify operation succeeded before proceeding
6. **Document limitations**: Note when features aren't available

## Getting Additional Help

If error persists after following this guide:

1. **Review main workflow**: See parent reference files for context
2. **Check technical details**: Review [technical-details.md](../technical-details.md)
3. **Consult examples**: See [examples.md](../examples.md) for working code
4. **Enable Pega tracer**: Use Pega's built-in tracer to debug API calls
5. **Contact administrator**: Engage Pega administrator for configuration issues

## Related Documentation

- [Success Criteria](../success-criteria.md) - Verification checklist
- [Technical Details](../technical-details.md) - API specifications
- [Examples](../examples.md) - Working code examples
- [Default Prompts](../default-prompts.md) - Default prompt templates

## Quick Reference

### Essential Rules

✓ Target paths start with dot: `.FieldName`
✓ listIndex is 1-based (first item = 1)
✓ Role names are case-sensitive
✓ Always query available options first
✓ Verify response before proceeding
✓ Handle empty/null values gracefully

### Common Error Codes

- **400 Bad Request**: Validation error, check field values
- **403 Forbidden**: Permission error, check user roles
- **404 Not Found**: Resource doesn't exist, verify IDs
- **500 Internal Server Error**: Pega server error, check logs

### Key Data Views

- `D_IndexList`: Available collections
- `D_BuddyAccessRoleList`: Available access roles
- `D_BuddyList`: Existing buddies
- `D_DataSourceList`: Available data sources

### Case Type and Actions

- **Case Type**: `PegaFW-QnA-Work-KnowledgeQnA`
- **Create Action**: `Create`
- **Security Action**: `ConfigureSecurity`
- **Prompt Action**: `ConfigurePrompt`

This index provides an overview of error handling across the entire workflow. Refer to individual exception files for detailed error scenarios and solutions.
