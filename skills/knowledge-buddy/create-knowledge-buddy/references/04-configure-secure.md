# Step 4: Configure Access Control (Secure Stage)

## Purpose

Submit Create assignment to transition to Secure stage, then configure access roles.

## Part A: Submit Create Assignment

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    Name: "TestBuddy",
    pyDescription: "TestBuddy description",
    IsCloned: false
    // OriginalBuddyId: "BUDDY-1001" // Only if IsCloned is true
  },
  pageInstructions: []
})
```

**Fields**:
- `Name`: Buddy name (required)
- `pyDescription`: Description (optional)
- `IsCloned`: Boolean
- `OriginalBuddyId`: Required only if IsCloned is true

Case transitions: Create (PRIM0) → Secure (PRIM3). System initializes `BuddyAccessConfigurations` with 2 default entries.

**Extract**: Security assignment ID and action ID from response.

## Part B: Submit Security Assignment

### Simple Mode

Keep default access configurations using empty content:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!SECURE_FLOW",
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".BuddyAccessConfigurations",
      listIndex: 1,
      content: {}  // Empty = keep existing
    },
    {
      instruction: "UPDATE",
      target: ".BuddyAccessConfigurations",
      listIndex: 2,
      content: {}  // Empty = keep existing
    }
  ]
})
```

### Advanced Mode

Allow user to customize roles, then submit with custom values:

```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".BuddyAccessConfigurations",
    listIndex: 1,
    content: {
      BuddyAccessType: "manage",
      AccessRoleName: "KnowledgeBuddy:Admin"  // Custom role
    }
  },
  {
    instruction: "UPDATE",
    target: ".BuddyAccessConfigurations",
    listIndex: 2,
    content: {
      BuddyAccessType: "use",
      AccessRoleName: "KnowledgeBuddy:Internal"  // Custom role
    }
  }
]
```

**Access Types**: `manage`, `use`, `view`

**Available Roles**: See D_BuddyAccessRoleList from Step 1

## PageInstructions Rules

- Target MUST start with dot: `.BuddyAccessConfigurations`
- `listIndex` is 1-based
- Empty `content: {}` keeps existing values
- Use `UPDATE` for modifying existing items

Case transitions: Secure (PRIM3) → Prompt (PRIM1)

**Extract**: Prompt assignment ID from response.

## Response Formats

See [response-structures.md](response-structures.md)

## Next Step

[05-configure-prompt-simple.md](05-configure-prompt-simple.md)
