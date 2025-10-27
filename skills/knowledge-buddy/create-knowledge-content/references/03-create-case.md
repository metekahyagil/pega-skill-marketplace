# Step 3: Create Content Case

## Purpose

Create a new Content case for knowledge base article.

## Case Type

```
PegaFW-KB-Work-Article
```

## MCP Tool

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
})
```

## Extract from Response

- **Case ID**: `caseID` (e.g., "KB-1023")
- **Assignment ID**: `assignments[0].ID`
- **Assignment Action**: "Create" (CREATEFORM_DEFAULT)

Case stage after creation: **Create** (PRIM0)

## Example Response

```javascript
{
  caseID: "KB-1023",
  assignments: [{
    ID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
    name: "Create"
  }]
}
```

Store `caseID` and `assignmentID` for next steps.

## Next Step

[04-configure-case.md](04-configure-case.md)

**Exceptions**: See [error-handling/03-create-case-exceptions.md](error-handling/03-create-case-exceptions.md) for troubleshooting.
