# Complete Working Example

## Overview

Comprehensive example demonstrating:
- Cloning from existing buddy
- Custom access control (3 configurations with INSERT/DELETE)
- Custom prompts (Instructions and Information)
- Multiple context definitions
- Specific GenAI model selection
- Advanced context settings

---

## Full Workflow Code

```javascript
// ============================================================
// STEP 0: Authenticate
// ============================================================
await mcp__pega-dx-mcp__authenticate_pega({});


// ============================================================
// STEP 1: Create Case
// ============================================================
const caseResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-QnA-Work-KnowledgeQnA",
  content: { pyAddCaseContextPage: {} },
  processID: "pyStartCase"
});

const caseID = caseResponse.data.caseInfo.ID;
const businessID = caseResponse.data.caseInfo.businessID;
const createAssignment = caseResponse.data.caseInfo.assignments[0].ID;

console.log(`Created case: ${businessID}`);


// ============================================================
// STEP 2: Ask Clone or New (User selected "Clone")
// ============================================================

// Query existing buddies
const buddiesResponse = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_OriginalBuddyList",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" },
      { field: "pyDescription" }
    ],
    distinctResultsOnly: "true"
  },
  paging: { pageNumber: 1, pageSize: 50 }
});

// User selected: Clone from BUDDY-1


// ============================================================
// STEP 3: Gather Clone Input
// ============================================================
const buddyName = "Customer Support Buddy v2";
const buddyDescription = "Enhanced customer support assistant";
const originalBuddyId = "BUDDY-1";


// ============================================================
// STEP 4: Submit Create Action (with clone)
// ============================================================
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: createAssignment,
  actionID: "Create",
  content: {
    Name: buddyName,
    pyDescription: buddyDescription,
    IsCloned: true,
    OriginalBuddyId: originalBuddyId
  },
  pageInstructions: []
});


// ============================================================
// STEP 4b: Refresh Case (Required after Create)
// ============================================================
const refreshResponse = await fetch(
  `https://your-instance/prweb/app/KnowledgeBuddy/api/application/v2/cases/${caseID.replace(' ', '%20')}/views/pyDetailsTabContent/refresh`,
  {
    method: 'PATCH',
    body: JSON.stringify({
      content: {},
      pageInstructions: [],
      interestPage: ""
    })
  }
);

const refreshData = await refreshResponse.json();
const secureAssignment = refreshData.data.caseInfo.assignments[0].ID;


// ============================================================
// STEP 5: Configure Access Control (User selected "Custom")
// ============================================================

// Query available roles
const rolesResponse = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_BuddyAccessRoleList",
  query: {
    select: [
      { field: "pyAccessRole" },
      { field: "pyLabel" }
    ]
  },
  dataViewParameters: { CaseId: businessID },
  paging: { pageSize: 20 }
});

// User wants 3 configurations:
// 1. "use" -> "KnowledgeBuddy:Admin"
// 2. "view" -> "KnowledgeBuddy:BuddyManager"
// 3. "manage" -> "KnowledgeBuddy:Public"

const promptResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: secureAssignment,
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    // Delete existing 2 defaults
    { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },
    { target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "DELETE" },

    // Config 1: INSERT + UPDATE
    { target: ".BuddyAccessConfigurations", content: {}, listIndex: 1, instruction: "INSERT" },
    { content: { BuddyAccessType: "use" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },

    // Config 2: INSERT + UPDATE
    { target: ".BuddyAccessConfigurations", content: {}, listIndex: 2, instruction: "INSERT" },
    { content: { BuddyAccessType: "view" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },

    // Config 3: INSERT + UPDATE
    { target: ".BuddyAccessConfigurations", content: {}, listIndex: 3, instruction: "INSERT" },
    { content: { BuddyAccessType: "manage" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" },

    // Add roles
    { content: { AccessRoleName: "KnowledgeBuddy:Admin" }, target: ".BuddyAccessConfigurations", listIndex: 1, instruction: "UPDATE" },
    { content: { AccessRoleName: "KnowledgeBuddy:BuddyManager" }, target: ".BuddyAccessConfigurations", listIndex: 2, instruction: "UPDATE" },
    { content: { AccessRoleName: "KnowledgeBuddy:Public" }, target: ".BuddyAccessConfigurations", listIndex: 3, instruction: "UPDATE" }
  ]
});

const promptAssignment = promptResponse.data.nextAssignmentInfo.ID;


// ============================================================
// STEP 6: Configure Prompts (User selected custom)
// ============================================================
const systemMessage = `You are a customer service representative. Answer questions based *only* on CONTEXT provided between the text marked with ######.

**Constraints:**
1. Utilize *only* the provided CONTEXT for a comprehensive answer.
2. Do not state your role in the response.
3. If the answer is not within the CONTEXT, respond with "I don't know."
4. Answer in the same language as the question.`;

const userMessage = `CONTEXT:
######
{SEARCHRESULTS}
######

This is the end of CONTEXT.

QUESTION:
{QUESTION}`;


// ============================================================
// STEP 7: Configure Context (User selected "Add custom contexts")
// ============================================================

// Query collections
const collectionsResponse = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: {
    select: [
      { field: "pyID" },
      { field: "CollectionName" }
    ],
    distinctResultsOnly: "true"
  },
  paging: { pageNumber: 1, pageSize: 50 }
});

// User wants 2 contexts:
// Context 1: SEARCHRESULTS with collection DC-1
// Context 2: ADDITIONAL_DOCS with collection DC-2

// Query data sources for DC-1
const ds1Response = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_DataSourceListByCollection",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" }
    ],
    distinctResultsOnly: "true"
  },
  dataViewParameters: {
    pyID: "DC-1",  // Parameter name is "pyID", value is collection's pyID
    Available: "Yes"
  },
  paging: { maxResultsToFetch: "100" }
});

// Query data sources for DC-2
const ds2Response = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_DataSourceListByCollection",
  query: {
    select: [
      { field: "Name" },
      { field: "pyID" }
    ],
    distinctResultsOnly: "true"
  },
  dataViewParameters: {
    pyID: "DC-2",  // Parameter name is "pyID", value is collection's pyID
    Available: "Yes"
  },
  paging: { maxResultsToFetch: "100" }
});

// Query attributes
const attrsResponse = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_AttributeList",
  dataViewParameters: {
    indexName: "knowledge"
  }
});


// ============================================================
// STEP 8: Configure GenAI (User selected GPT-4o)
// ============================================================

// Query GenAI models
const modelsResponse = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_bxGenAIModelsForBuddy",
  query: {
    select: [
      { field: "pyID" },
      { field: "pyLabel" },
      { field: "ModelType" }
    ]
  },
  dataViewParameters: { ModelId: "" }
});

// User selected:
// - Model: "azure/openai/GPT-4o/2024-08-06"
// - Output format: "Custom"
// - Text replacements: true
// - Auto filtering: true


// ============================================================
// STEP 9: Submit ConfigurePrompt Assignment
// ============================================================
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: promptAssignment,
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: systemMessage,
    UserMessage: userMessage,
    GenAIModelId: "azure/openai/GPT-4o/2024-08-06",
    OutputFormat: "Custom",
    IncludeTextReplacements: true,
    IncludeAutoFiltering: true
  },
  pageInstructions: [
    // ============================================
    // Context 1: SEARCHRESULTS (DC-1)
    // ============================================
    {
      instruction: "UPDATE",
      target: ".ContextDataConfigs",
      listIndex: 1,
      content: {
        Name: "SEARCHRESULTS",
        Collection: { pyID: "DC-1" },
        MustReturnResults: true,
        MaxChunk: 5,
        MaxChunkTotalSize: 5000,
        DataSourcesCSV: "",
        ResponseAttributesCSV: "",
        MinSimilarityScore: 80
      }
    },

    // Delete and insert data sources for Context 1
    { instruction: "DELETE ALL", target: ".ContextDataConfigs(1).DataSources" },
    {
      target: ".ContextDataConfigs(1).DataSources",
      content: { pyID: "SRC-2001" },
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      target: ".ContextDataConfigs(1).DataSources",
      content: { pyID: "SRC-2" },
      listIndex: 2,
      instruction: "INSERT"
    },

    // Insert response attributes for Context 1
    {
      target: ".ContextDataConfigs(1).ResponseAttributes",
      content: { Name: "contentID" },
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      target: ".ContextDataConfigs(1).ResponseAttributes",
      content: { Name: "contentKey" },
      listIndex: 2,
      instruction: "INSERT"
    },

    // ============================================
    // Context 2: ADDITIONAL_DOCS (DC-2)
    // ============================================
    {
      target: ".ContextDataConfigs",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },

    {
      instruction: "UPDATE",
      target: ".ContextDataConfigs",
      listIndex: 2,
      content: {
        Name: "ADDITIONAL_DOCS",
        Collection: { pyID: "DC-2" },
        MustReturnResults: false,
        MaxChunk: 3,
        MaxChunkTotalSize: 3000,
        DataSourcesCSV: "",
        ResponseAttributesCSV: "",
        MinSimilarityScore: 70
      }
    },

    // Delete and insert data sources for Context 2
    { instruction: "DELETE ALL", target: ".ContextDataConfigs(2).DataSources" },
    {
      target: ".ContextDataConfigs(2).DataSources",
      content: { pyID: "SRC-3" },
      listIndex: 1,
      instruction: "INSERT"
    },

    // Insert response attributes for Context 2
    {
      target: ".ContextDataConfigs(2).ResponseAttributes",
      content: { Name: "contentURL" },
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      target: ".ContextDataConfigs(2).ResponseAttributes",
      content: { Name: "contentSourceSystem" },
      listIndex: 2,
      instruction: "INSERT"
    }
  ]
});

console.log("ConfigurePrompt submitted - case auto-resolving...");


// ============================================================
// STEP 10: Verify Success
// ============================================================
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: caseID
});

const status = finalCase.data.caseInfo.status;
const buddyID = finalCase.data.caseInfo.businessID;

console.log(`Buddy created successfully!`);
console.log(`  - Case ID: ${caseID}`);
console.log(`  - Buddy ID: ${buddyID}`);
console.log(`  - Status: ${status}`);
console.log(`  - URL: https://your-instance/prweb/app/KnowledgeBuddy?BuddyID=${buddyID}`);
```

---

## Configuration Summary

| Component | Value |
|-----------|-------|
| **Clone Source** | BUDDY-1 |
| **Name** | Customer Support Buddy v2 |
| **Description** | Enhanced customer support assistant |
| **Access Configs** | 3 configurations (use/view/manage) |
| **Roles** | Admin, BuddyManager, Public |
| **Instructions** | Custom SystemMessage (simplified) |
| **Information** | Custom UserMessage with SEARCHRESULTS placeholder |
| **Context 1** | SEARCHRESULTS → DC-1 (2 data sources, 2 attributes, 5 chunks, score 80) |
| **Context 2** | ADDITIONAL_DOCS → DC-2 (1 data source, 2 attributes, 3 chunks, score 70) |
| **GenAI Model** | azure/openai/GPT-4o/2024-08-06 |
| **Output Format** | Custom |
| **Text Replacements** | Enabled |
| **Auto Filtering** | Enabled |

---

## Key Patterns Demonstrated

1. **Cloning**: IsCloned=true with OriginalBuddyId
2. **Dynamic Access**: DELETE existing, INSERT empty, UPDATE with values
3. **Custom Prompts**: Custom SystemMessage and UserMessage strings
4. **Multiple Contexts**: Two context definitions with different collections
5. **Data Sources**: DELETE ALL → INSERT pattern for each context
6. **Response Attributes**: INSERT for each attribute per context
7. **GenAI Selection**: Specific model ID with custom settings
8. **Advanced Context Settings**: Different MaxChunk, MinSimilarityScore per context

---

## Minimal Example (Defaults)

For comparison, here's the minimal default setup:

```javascript
// Steps 0-4: Same (authenticate, create case, submit Create action, refresh case)

// Step 5: Default access
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: secureAssignment,
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    { instruction: "UPDATE", target: ".BuddyAccessConfigurations", listIndex: 1, content: {} },
    { instruction: "UPDATE", target: ".BuddyAccessConfigurations", listIndex: 2, content: {} }
  ]
});

// Step 6-9: Default prompts, default SEARCHRESULTS context, default GenAI
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: promptAssignment,
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "<default from default-prompts.md>",
    UserMessage: "<default from default-prompts.md>",
    GenAIModelId: "",
    OutputFormat: "Custom",
    IncludeTextReplacements: false,
    IncludeAutoFiltering: false
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".ContextDataConfigs",
      listIndex: 1,
      content: {
        Name: "SEARCHRESULTS",
        Collection: { pyID: "" },  // No collection (null)
        MustReturnResults: true,
        MaxChunk: 5,
        MaxChunkTotalSize: 5000,
        DataSourcesCSV: "",
        ResponseAttributesCSV: "",
        MinSimilarityScore: 80
      }
    }
  ]
});
```

This minimal example takes the fastest path with all defaults. Note that even with defaults, SEARCHRESULTS context is explicitly configured with its default values.
