# Configure Access Control - Custom Configuration

## Overview

Configure custom buddy access control with specific roles and access types. Supports dynamic number of configurations.

---

## Initial State

After Create assignment submission, case is in **Secure stage (PRIM3)** with:
- Security Assignment ID (e.g., "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!SECURE_FLOW")
- Action ID: "ConfigureSecurity"
- Default configs: 2 entries in `BuddyAccessConfigurations` page list

---

## Prerequisites

- D_BuddyAccessRoleList query completed (see [01-on-demand-queries.md](01-on-demand-queries.md))

---

## Step 1: Ask Number of Configurations

**Atomic question**: "How many access configurations?"

```javascript
AskUserQuestion({
  questions: [{
    question: "How many access configurations do you need?",
    header: "Config count",
    multiSelect: false,
    options: [
      { label: "1", description: "Single access configuration" },
      { label: "2", description: "Two access configurations" },
      { label: "3", description: "Three access configurations" },
      { label: "Enter custom", description: "Specify number (1-10)" }
    ]
  }]
})
```

---

## Step 2: For Each Configuration, Ask Details

**For each config** (repeat N times):

### Question 2a: Access Type

"Select access type for configuration {index}"

```javascript
AskUserQuestion({
  questions: [{
    question: "Select access type for configuration 1:",
    header: "Access type",
    multiSelect: false,
    options: [
      { label: "manage", description: "Full management access (edit configuration, delete buddy)" },
      { label: "use", description: "Use buddy for Q&A" },
      { label: "view", description: "View buddy configuration only" }
    ]
  }]
})
```

### Question 2b: Role Selection

"Select role for configuration {index}"

```javascript
AskUserQuestion({
  questions: [{
    question: "Select role for configuration 1:",
    header: "Role",
    multiSelect: false,
    options: [
      { label: "KnowledgeBuddy:Admin", description: "Administrator role" },
      { label: "KnowledgeBuddy:BuddyManager", description: "Buddy manager role" },
      { label: "KnowledgeBuddy:Public", description: "Public access role" }
      // ... populated from D_BuddyAccessRoleList query
    ]
  }]
})
```

---

## Step 3: Build pageInstructions

Build dynamic pageInstructions using INSERT and UPDATE operations.

**Pattern**:
1. DELETE existing defaults
2. For each config: INSERT empty item, then UPDATE with values

**Example for 3 configurations**:

```javascript
pageInstructions: [
  // Delete existing 2 defaults
  { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },
  { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },

  // Config 1: INSERT then UPDATE
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 1, instruction: "INSERT" },
  { content: { BuddyAccessType: "use" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },

  // Config 2: INSERT then UPDATE
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 2, instruction: "INSERT" },
  { content: { BuddyAccessType: "view" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },

  // Config 3: INSERT then UPDATE
  { target: ".BuddyAccessConfigurations", content: {}, listIndex: 3, instruction: "INSERT" },
  { content: { BuddyAccessType: "manage" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" },

  // Add roles
  { content: { AccessRoleName: "KnowledgeBuddy:Admin" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },
  { content: { AccessRoleName: "KnowledgeBuddy:BuddyManager" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },
  { content: { AccessRoleName: "KnowledgeBuddy:Public" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" }
]
```

---

## Step 4: Submit Security Assignment

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1011!SECURE_FLOW",
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    // ... dynamic instructions built in Step 3
  ]
})
```

**API Equivalent**:
```
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-QNA-WORK%20BUDDY-1011!SECURE_FLOW/actions/ConfigureSecurity?viewType=form
{
  "content": {},
  "pageInstructions": [...]
}
```

---

## Access Type Reference

| Access Type | Description | Capabilities |
|-------------|-------------|--------------|
| **manage** | Full management | Edit configuration, delete buddy, use buddy |
| **use** | Usage only | Ask questions, view responses |
| **view** | Read-only | View configuration, cannot edit or use |

---

## pageInstructions Rules

1. **Target format**: Must start with dot (`.BuddyAccessConfigurations`)
2. **listIndex**: 1-based indexing
3. **DELETE**: Removes item at specified index (subsequent items shift down)
4. **INSERT**: Adds empty item at specified index
5. **UPDATE**: Modifies item at specified index with content
6. **Pattern**: DELETE existing → INSERT empty → UPDATE with values

---

## Stage Transition

After successful submission:
- Case transitions from **Secure (PRIM3)** → **Prompt (PRIM1)**
- Prompt Assignment created

**Extract from Response**:
- Prompt Assignment ID: `data.caseInfo.assignments[0].ID` (e.g., "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1011!PROMPT_FLOW")
- Action ID: `data.caseInfo.assignments[0].actions[0].ID` (should be "ConfigurePrompt")

---

## Next Step

[05-configure-prompts.md](05-configure-prompts.md) - Configure Instructions and Information
