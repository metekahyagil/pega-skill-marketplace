# Create Knowledge Buddy Case

## Overview

Create the Knowledge Buddy case and submit the Create assignment with buddy details.

## Part A: Create Case

Create empty case to get case ID and first assignment.

**MCP Tool**: `mcp__pega-dx-mcp__create_case`

```javascript
mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-QnA-Work-KnowledgeQnA",
  content: {
    pyAddCaseContextPage: {}
  },
  processID: "pyStartCase"
})
```

**API Equivalent** (from flow.md):
```
POST /prweb/app/KnowledgeBuddy/api/application/v2/cases?viewType=page
{
  "caseTypeID": "PegaFW-QnA-Work-KnowledgeQnA",
  "content": {"pyAddCaseContextPage": {}},
  "processID": "pyStartCase"
}
```

**Extract from Response**:
- Case ID: `data.caseInfo.ID` (e.g., "PEGAFW-QNA-WORK BUDDY-1002")
- Business ID: `data.caseInfo.businessID` (e.g., "BUDDY-1002")
- Assignment ID: `data.caseInfo.assignments[0].ID` (e.g., "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!CREATEFORM_DEFAULT")
- Action ID: `data.caseInfo.assignments[0].actions[0].ID` (should be "Create")

Case is now in **Create stage (PRIM0)**.

---

## Part B: Submit Create Assignment

Submit the Create assignment with buddy name, description, and clone information.

**MCP Tool**: `mcp__pega-dx-mcp__perform_assignment_action`

### Example 1: Cloning Existing Buddy

When `IsCloned=true`, include `OriginalBuddyId`:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1009!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    Name: "Customer Support Buddy",
    pyDescription: "Answers customer support questions",
    IsCloned: true,
    OriginalBuddyId: "BUDDY-1"
  },
  pageInstructions: []
})
```

**API Equivalent** (from flow.md):
```
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-QNA-WORK%20BUDDY-1009!CREATEFORM_DEFAULT/actions/Create?viewType=page
{
  "content": {
    "Name": "Customer Support Buddy",
    "pyDescription": "Answers customer support questions",
    "IsCloned": true,
    "OriginalBuddyId": "BUDDY-1"
  },
  "pageInstructions": []
}
```

---

### Example 2: Creating New Buddy

When `IsCloned=false`, omit `OriginalBuddyId`:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1010!CREATEFORM_DEFAULT",
  actionID: "Create",
  content: {
    Name: "Product Documentation Buddy",
    pyDescription: "Answers questions about product documentation",
    IsCloned: false
  },
  pageInstructions: []
})
```

**API Equivalent** (from flow.md):
```
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-QNA-WORK%20BUDDY-1010!CREATEFORM_DEFAULT/actions/Create?viewType=page
{
  "content": {
    "Name": "Product Documentation Buddy",
    "pyDescription": "Answers questions about product documentation",
    "IsCloned": false
  },
  "pageInstructions": []
}
```

---

## Field Reference

| Field | Type | Required | Description | Example |
|-------|------|----------|-------------|---------|
| Name | string | Yes | Buddy name | "Customer Support Buddy" |
| pyDescription | string | No | Buddy description | "Answers customer support questions" |
| IsCloned | boolean | Yes | Whether cloning from existing buddy | true or false |
| OriginalBuddyId | string | Conditional* | ID of buddy to clone from | "BUDDY-1" |

*Required only when `IsCloned=true`

---

## Stage Transition

After submitting Create assignment:
- Case transitions from **Create (PRIM0)** → **Secure (PRIM3)**
- System initializes `BuddyAccessConfigurations` with 2 default entries

**Extract from Response**:
- Security Assignment ID: `data.caseInfo.assignments[0].ID` (e.g., "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!SECURE_FLOW")
- Security Action ID: `data.caseInfo.assignments[0].actions[0].ID` (should be "ConfigureSecurity")

---

## Part C: Refresh Case (Required)

After Create submission, refresh the case to get updated assignment information.

**MCP Tool**: `mcp__pega-dx-mcp__refresh_case` (or equivalent API call)

```javascript
// Refresh case view
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/cases/{CASE-ID}/views/pyDetailsTabContent/refresh

Request body:
{
  "content": {},
  "pageInstructions": [],
  "interestPage": ""
}
```

**Example**:
```
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/cases/PEGAFW-QNA-WORK%20BUDDY-1015/views/pyDetailsTabContent/refresh
{
  "content": {},
  "pageInstructions": [],
  "interestPage": ""
}
```

**Purpose**: Refresh ensures the case view is synchronized with the latest assignment state before proceeding to Secure stage.

**Extract from Response**:
- Confirm Security Assignment ID is available
- Proceed to Step 5

---

## Response Handling

See [response-structures.md](response-structures.md) for complete response formats.

---

## Next Step

Configure access control in Secure stage. User will choose one of two paths:
- [04a-configure-access-default.md](04a-configure-access-default.md) - Use default access configuration
- [04b-configure-access-custom.md](04b-configure-access-custom.md) - Configure custom access
