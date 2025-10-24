# Response Structures Reference

This document contains JSON response examples for MCP tool calls used in the Knowledge Buddy creation workflow. Reference this when you need to understand response formats.

---

## create_case

**Used In**: Step 3 - Create Knowledge Buddy Case

**Response**:
```json
{
  "data": {
    "caseInfo": {
      "ID": "PEGAFW-QNA-WORK BUDDY-1002",
      "businessID": "BUDDY-1002",
      "caseTypeID": "PegaFW-QnA-Work-KnowledgeQnA",
      "status": "New",
      "stageID": "PRIM0",
      "stageLabel": "Create",
      "assignments": [{
        "ID": "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!CREATEFORM_DEFAULT",
        "name": "Create",
        "actions": [{
          "ID": "Create",
          "name": "Create",
          "type": "FlowAction"
        }]
      }],
      "content": {
        "Name": "",
        "pyDescription": "",
        "IsCloned": false,
        "OriginalBuddyId": ""
      }
    }
  }
}
```

**Key Fields**:
- `data.caseInfo.ID` - Full case ID
- `data.caseInfo.businessID` - Business ID (e.g., BUDDY-1002)
- `data.caseInfo.assignments[0].ID` - First assignment ID
- `data.caseInfo.assignments[0].actions[0].ID` - Action ID

---

## perform_assignment_action (Create Assignment)

**Used In**: Step 4 - Submit Create Assignment

**Response**:
```json
{
  "data": {
    "caseInfo": {
      "stageID": "PRIM3",
      "stageLabel": "Secure",
      "status": "New",
      "name": "TestBuddy",
      "assignments": [{
        "ID": "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!SECURE_FLOW",
        "name": "Configure security",
        "processID": "Secure_Flow",
        "actions": [{
          "ID": "ConfigureSecurity",
          "name": "Configure security"
        }]
      }],
      "content": {
        "Name": "TestBuddy",
        "BuddyAccessConfigurations": [
          {
            "BuddyAccessType": "manage",
            "AccessRoleName": "KnowledgeBuddy:BuddyManager",
            "pyID": "BUDDY-1002"
          },
          {
            "BuddyAccessType": "use",
            "AccessRoleName": "KnowledgeBuddy:Public",
            "pyID": "BUDDY-1002"
          }
        ]
      }
    }
  }
}
```

**Key Fields**:
- `data.caseInfo.stageID` - New stage (PRIM3 = Secure)
- `data.caseInfo.assignments[0].ID` - Security assignment ID
- `data.caseInfo.content.BuddyAccessConfigurations` - Initialized access configs

---

## perform_assignment_action (Configure Security)

**Used In**: Step 4 - Submit Security Assignment

**Response**:
```json
{
  "data": {
    "caseInfo": {
      "stageID": "PRIM1",
      "stageLabel": "Prompt",
      "assignments": [{
        "ID": "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!PROMPT_FLOW",
        "name": "Configure prompt",
        "actions": [{
          "ID": "ConfigurePrompt"
        }]
      }]
    }
  }
}
```

**Key Fields**:
- `data.caseInfo.stageID` - New stage (PRIM1 = Prompt)
- `data.caseInfo.assignments[0].ID` - Prompt assignment ID

---

## perform_assignment_action (Configure Prompt)

**Used In**: Step 5-7 - Submit Prompt Configuration

**Response**:
```json
{
  "data": {
    "caseInfo": {
      "ID": "PEGAFW-QNA-WORK BUDDY-1002",
      "businessID": "BUDDY-1002",
      "status": "Resolved-Completed",
      "stageID": "PRIM2",
      "stageLabel": "Resolve",
      "assignments": [],
      "availableActions": [
        { "name": "Create new version", "ID": "pyReopen" },
        { "name": "Create linked buddy", "ID": "CasewideLocalActions1" },
        { "name": "Edit configuration", "ID": "CasewideLocalActions2" }
      ]
    }
  },
  "confirmationNote": "Thank you! The next step in this case has been routed appropriately."
}
```

**Key Fields**:
- `data.caseInfo.status` - Case resolved
- `data.caseInfo.stageID` - Final stage (PRIM2 = Resolve)
- `data.caseInfo.assignments` - Empty (no more assignments)

---

## get_list_data_view (D_IndexList)

**Used In**: Step 1 - Query Collections

**Response**:
```json
{
  "resultCount": 3,
  "data": [
    {
      "CollectionName": "Product Documentation",
      "pyID": "DC-1",
      "pzInsKey": "PEGAFW-QNA-WORK DC-1"
    },
    {
      "CollectionName": "HR Policies",
      "pyID": "DC-2",
      "pzInsKey": "PEGAFW-QNA-WORK DC-2"
    }
  ]
}
```

**Key Fields**:
- `data[].CollectionName` - Display name
- `data[].pyID` - Collection ID
- `data[].pzInsKey` - Full key for references

---

## get_list_data_view (D_bxGenAIModelsForBuddy)

**Used In**: Step 1 - Query GenAI Models

**Response**:
```json
{
  "resultCount": 5,
  "data": [
    {
      "pyID": "",
      "pyLabel": "Pega-Default",
      "ModelType": "Pega-Default"
    },
    {
      "pyID": "gpt-4",
      "pyLabel": "GPT-4",
      "ModelType": "OpenAI"
    }
  ]
}
```

**Key Fields**:
- `data[].pyID` - Model ID (empty for Pega-Default)
- `data[].pyLabel` - Display name
- `data[].ModelType` - Model type

---

## get_list_data_view (D_BuddyAccessRoleList)

**Used In**: Step 1 - Query Access Roles

**Response**:
```json
{
  "resultCount": 6,
  "data": [
    {
      "pyAccessRole": "KnowledgeBuddy:Admin",
      "pyLabel": "Buddy administrator"
    },
    {
      "pyAccessRole": "KnowledgeBuddy:BuddyManager",
      "pyLabel": "Knowledge buddy manager"
    },
    {
      "pyAccessRole": "KnowledgeBuddy:Public",
      "pyLabel": "Knowledge buddy public"
    }
  ]
}
```

**Key Fields**:
- `data[].pyAccessRole` - Role identifier
- `data[].pyLabel` - Display name

---

## get_list_data_view (D_OriginalBuddyList)

**Used In**: Step 1 - Query Existing Buddies

**Response**:
```json
{
  "resultCount": 2,
  "data": [
    {
      "Name": "Customer Support Buddy",
      "pyID": "BUDDY-1001",
      "pyDescription": "Handles customer support questions"
    },
    {
      "Name": "HR Assistant",
      "pyID": "BUDDY-1002",
      "pyDescription": "Answers HR policy questions"
    }
  ]
}
```

**Key Fields**:
- `data[].Name` - Buddy name
- `data[].pyID` - Buddy ID
- `data[].pyDescription` - Description

---

## get_case

**Used In**: Step 6 - Verify Success

**Response**:
```json
{
  "data": {
    "caseInfo": {
      "ID": "PEGAFW-QNA-WORK BUDDY-1002",
      "businessID": "BUDDY-1002",
      "caseTypeID": "PegaFW-QnA-Work-KnowledgeQnA",
      "status": "Resolved-Completed",
      "stageID": "PRIM2",
      "stageLabel": "Resolve",
      "name": "TestBuddy",
      "assignments": [],
      "stages": [
        {
          "ID": "PRIM0",
          "name": "Create",
          "visited_status": "completed"
        },
        {
          "ID": "PRIM3",
          "name": "Secure",
          "visited_status": "completed"
        },
        {
          "ID": "PRIM1",
          "name": "Prompt",
          "visited_status": "completed"
        },
        {
          "ID": "PRIM2",
          "name": "Resolve",
          "visited_status": "completed"
        }
      ],
      "content": {
        "Name": "TestBuddy",
        "pyDescription": "TestBuddy description",
        "SystemMessage": "...",
        "UserMessage": "...",
        "BuddyAccessConfigurations": [...],
        "ContextDataConfigs": [...]
      }
    }
  }
}
```

**Key Fields**:
- `data.caseInfo.status` - Should be "Resolved-Completed"
- `data.caseInfo.name` - Buddy name
- `data.caseInfo.stages` - All stages with visited_status
- `data.caseInfo.content` - Complete configuration
