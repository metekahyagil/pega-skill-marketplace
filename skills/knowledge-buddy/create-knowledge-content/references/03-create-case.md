# Step 3: Create Content Case

After gathering user input, create a new Content case in the Pega Knowledge Buddy system.

## Case Type

The Content case uses the case type:
```
PegaFW-KB-Work-Article
```

## Creating the Case

Use the `create_case` MCP tool with empty content:

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
})
```

## What This Returns

The response includes:
- **caseID**: The unique case identifier (e.g., `KB-1023`)
- **assignmentID**: The first assignment for the Create action
  - Format: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE <caseID>!CREATEFORM_DEFAULT`
  - Example: `ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT`

## Case Stage

After creation, the case is in the **Create** stage (PRIM0).

## What to Capture

Store both values for the next step:
1. `caseID` - For final verification
2. `assignmentID` - For performing the Create action

## Example

```javascript
const response = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-KB-Work-Article",
  content: {}
});

// Response:
// {
//   caseID: "KB-1023",
//   assignments: [{
//     ID: "ASSIGN-WORKLIST PEGAFW-KB-WORK-ARTICLE KB-1023!CREATEFORM_DEFAULT",
//     name: "Create",
//     type: "Create",
//     ...
//   }]
// }

const caseID = response.caseID;
const assignmentID = response.assignments[0].ID;
```

## Next Step

After creating the case, proceed to **[04-configure-case.md](./04-configure-case.md)** to configure collection, data source, and access settings.

---

**Exceptions**: See **[error-handling/03-create-case-exceptions.md](error-handling/03-create-case-exceptions.md)** for case creation failures and troubleshooting.
