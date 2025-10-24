# Complete Examples

This document provides two complete working examples: Simple mode (minimal configuration) and Advanced mode (full customization).

## Example 1: Simple Mode - Minimal Configuration

Create a buddy with all default settings:

```javascript
// Step 0: Authenticate
await mcp__pega-dx-mcp__authenticate_pega({});

// Step 1: Query data (collections, models, roles, buddies)
const collections = await mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_IndexList",
  query: { select: [{ field: "CollectionName" }, { field: "pyID" }, { field: "pzInsKey" }] },
  paging: { pageSize: 100 }
});

// Step 2: User input (assume Simple mode selected)
const buddyName = "Support Buddy";
const buddyDescription = "Answers support questions";

// Step 3: Create case
const caseResponse = await mcp__pega-dx-mcp__create_case({
  caseTypeID: "PegaFW-QnA-Work-KnowledgeQnA",
  content: { pyAddCaseContextPage: {} },
  processID: "pyStartCase"
});

const createAssignment = caseResponse.data.caseInfo.assignments[0].ID;

// Step 4: Submit Create assignment
const secureResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: createAssignment,
  actionID: "Create",
  content: {
    Name: buddyName,
    pyDescription: buddyDescription,
    IsCloned: false
  },
  pageInstructions: []
});

const secureAssignment = secureResponse.data.nextAssignmentInfo.ID;

// Step 4b: Submit Security (keep defaults)
const promptResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: secureAssignment,
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    { instruction: "UPDATE", target: ".BuddyAccessConfigurations", listIndex: 1, content: {} },
    { instruction: "UPDATE", target: ".BuddyAccessConfigurations", listIndex: 2, content: {} }
  ]
});

const promptAssignment = promptResponse.data.nextAssignmentInfo.ID;

// Step 5: Submit Prompt (defaults, no collection)
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: promptAssignment,
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "<see default-prompts.md>",
    UserMessage: "<see default-prompts.md>",
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
        Collection: { pyID: "" },
        MustReturnResults: true,
        MaxChunk: 5,
        MaxChunkTotalSize: 5000,
        MinSimilarityScore: 80
      }
    }
  ]
});

// Case auto-resolves - verify success
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: caseResponse.data.caseInfo.ID
});

console.log(`Buddy created: ${finalCase.data.caseInfo.businessID}`);
console.log(`Status: ${finalCase.data.caseInfo.status}`);
```

---

## Example 2: Advanced Mode - Full Customization

Create a buddy with custom access control, collection reference, and GenAI settings:

```javascript
// Steps 0-3: Same as Simple Mode

// Step 4: Submit Create assignment (same as Simple)
const secureResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: createAssignment,
  actionID: "Create",
  content: {
    Name: "HR Policy Buddy",
    pyDescription: "Answers HR policy questions",
    IsCloned: false
  },
  pageInstructions: []
});

const secureAssignment = secureResponse.data.nextAssignmentInfo.ID;

// Step 4b: Custom access roles
const promptResponse = await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: secureAssignment,
  actionID: "ConfigureSecurity",
  content: {},
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".BuddyAccessConfigurations",
      listIndex: 1,
      content: {
        BuddyAccessType: "manage",
        AccessRoleName: "KnowledgeBuddy:Admin"
      }
    },
    {
      instruction: "UPDATE",
      target: ".BuddyAccessConfigurations",
      listIndex: 2,
      content: {
        BuddyAccessType: "use",
        AccessRoleName: "KnowledgeBuddy:Internal"
      }
    }
  ]
});

const promptAssignment = promptResponse.data.nextAssignmentInfo.ID;

// Step 5: Custom prompts with collection and GenAI model
await mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: promptAssignment,
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "You are an HR assistant. Answer questions using only the provided HR policy documents.",
    UserMessage: "HR Policies:\n{SEARCHRESULTS}\n\nQuestion: {QUESTION}",
    GenAIModelId: "gpt-4",
    OutputFormat: "Custom",
    IncludeTextReplacements: true,
    IncludeAutoFiltering: true
  },
  pageInstructions: [
    {
      instruction: "UPDATE",
      target: ".ContextDataConfigs",
      listIndex: 1,
      content: {
        Name: "SEARCHRESULTS",
        Collection: {
          pyID: "DC-101",
          pzInsKey: "PEGAFW-QNA-WORK DC-101"
        },
        MustReturnResults: true,
        MaxChunk: 10,
        MaxChunkTotalSize: 8000,
        MinSimilarityScore: 75
      }
    }
  ]
});

// Verify success
const finalCase = await mcp__pega-dx-mcp__get_case({
  caseID: caseResponse.data.caseInfo.ID
});

console.log(`Buddy created: ${finalCase.data.caseInfo.businessID}`);
```

---

## Key Differences

| Aspect | Simple Mode | Advanced Mode |
|--------|-------------|---------------|
| Access Roles | Default (keep existing) | Custom (Admin + Internal) |
| Prompts | Default templates | Custom prompts |
| Collection | Empty (configure later) | Selected collection (DC-101) |
| Context Parameters | Default (5 chunks, 5000 size, 80 score) | Custom (10 chunks, 8000 size, 75 score) |
| GenAI Model | Pega default | GPT-4 |
| Text Replacements | Disabled | Enabled |
| Auto Filtering | Disabled | Enabled |
