# Configure Access Control - Default Configuration

## Overview

Keep 2 default access configurations without modification.

---

## Initial State

After Create assignment submission, case is in **Secure stage (PRIM3)** with:
- Security Assignment ID (e.g., "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!SECURE_FLOW")
- Action ID: "ConfigureSecurity"
- Default configs: 2 entries in `BuddyAccessConfigurations` page list

---

## Default Configurations

**Default configs**: Typically includes:
- "manage" access for Admin role
- "use" access for Public role

---

## pageInstructions

Update existing entries with empty content to keep defaults:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1011!SECURE_FLOW",
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

**Result**: Default configs preserved.

---

## Access Type Reference

| Access Type | Description | Capabilities |
|-------------|-------------|--------------|
| **manage** | Full management | Edit configuration, delete buddy, use buddy |
| **use** | Usage only | Ask questions, view responses |
| **view** | Read-only | View configuration, cannot edit or use |

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
