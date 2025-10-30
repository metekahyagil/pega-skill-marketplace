# Submit ConfigurePrompt Assignment

## Overview

Final step: Combine all configuration (prompts, context data, GenAI settings) and submit ConfigurePrompt assignment. Case auto-resolves after submission.

**Important**: When using pageInstructions to configure context data (data sources, response attributes, advanced settings), the `perform_assignment_action` call applies ALL changes in a single submission. No separate refresh is needed when using the MCP tool - pageInstructions are processed atomically with the content.

---

## Combining Configuration

Combine values from previous steps:

**From Step 5** ([05-configure-prompts.md](05-configure-prompts.md)):
- `systemMessage` (Instructions)
- `userMessage` (Information)

**From Step 6** (context configuration - one of 06a/06b/06c):
- `pageInstructions` array for context configurations

**From Step 7** (GenAI configuration - one of 07a/07b):
- `genAIModelId`
- `outputFormat`
- `includeTextReplacements`
- `includeAutoFiltering`

---

## Submission Structure

**MCP Tool**: `mcp__pega-dx-mcp__perform_assignment_action`

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1013!PROMPT_FLOW",
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "<from step 5>",
    UserMessage: "<from step 5>",
    GenAIModelId: "<from step 7>",
    OutputFormat: "<from step 7>",
    IncludeTextReplacements: <from step 7>,
    IncludeAutoFiltering: <from step 7>
  },
  pageInstructions: [
    /* Context configuration from step 6 */
  ]
})
```

---

## Example 1: Default Configuration

Minimal setup with defaults:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1013!PROMPT_FLOW",
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "You are a customer service representative...",  // Default from default-prompts.md
    UserMessage: "CONTEXT:\n######\n{SEARCHRESULTS}\n######\n...",  // Default from default-prompts.md
    GenAIModelId: "",  // Empty = default model
    OutputFormat: "Custom",
    IncludeTextReplacements: false,
    IncludeAutoFiltering: false
  },
  pageInstructions: []  // Empty = use default context
})
```

---

## Example 2: Custom Prompts + Single Context

Custom prompts with SEARCHRESULTS context:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1012!PROMPT_FLOW",
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "You are a customer service representative...",
    UserMessage: "CONTEXT:\n######\n{SEARCHRESULTS}\n######\n...",
    GenAIModelId: "azure/openai/GPT-4o/2024-08-06",
    OutputFormat: "Custom",
    IncludeTextReplacements: true,
    IncludeAutoFiltering: true
  },
  pageInstructions: [
    // Update SEARCHRESULTS context
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

    // Delete and insert data sources
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

    // Insert response attributes
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
    {
      target: ".ContextDataConfigs(1).ResponseAttributes",
      content: { Name: "contentSourceSystem" },
      listIndex: 3,
      instruction: "INSERT"
    },

    // Update with advanced settings
    {
      instruction: "UPDATE",
      target: ".ContextDataConfigs",
      listIndex: 1,
      content: {
        Collection: { pyID: "DC-1" },
        AdvancedSettings: true,
        MustReturnResults: true,
        MaxChunk: 9,
        MaxChunkTotalSize: 99,
        MinSimilarityScore: 999
      }
    }
  ]
})
```

---

## Example 3: Multiple Contexts

Two contexts with different collections (from flow.md):

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1013!PROMPT_FLOW",
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "You are a customer service representative...",
    UserMessage: "CONTEXT:\n######\n{SEARCHRESULTS}\n######\n...",
    GenAIModelId: "azure/openai/GPT-4o/2024-08-06",
    OutputFormat: "Custom",
    IncludeTextReplacements: true,
    IncludeAutoFiltering: true
  },
  pageInstructions: [
    // Update Context 1: SEARCHRESULTS
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
        DataSourcesCSV: "",
        ResponseAttributesCSV: "",
        MinSimilarityScore: 80
      }
    },

    // Insert Context 2
    {
      target: ".ContextDataConfigs",
      content: {},
      listIndex: 2,
      instruction: "INSERT"
    },

    // Configure Context 2 data sources
    { instruction: "DELETE ALL", target: ".ContextDataConfigs(2).DataSources" },
    {
      target: ".ContextDataConfigs(2).DataSources",
      content: { pyID: "SRC-2001" },
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      target: ".ContextDataConfigs(2).DataSources",
      content: { pyID: "SRC-2" },
      listIndex: 2,
      instruction: "INSERT"
    },

    // Update Context 2 data sources
    {
      content: { pyID: "SRC-2001" },
      target: ".ContextDataConfigs(2).DataSources",
      listIndex: 1,
      instruction: "UPDATE"
    },
    {
      content: { pyID: "SRC-2" },
      target: ".ContextDataConfigs(2).DataSources",
      listIndex: 2,
      instruction: "UPDATE"
    },

    // Insert Context 2 response attributes
    {
      target: ".ContextDataConfigs(2).ResponseAttributes",
      content: { Name: "contentSourceSystem" },
      listIndex: 1,
      instruction: "INSERT"
    },
    {
      target: ".ContextDataConfigs(2).ResponseAttributes",
      content: { Name: "contentURL" },
      listIndex: 2,
      instruction: "INSERT"
    }
  ]
})
```

**API Equivalent** (from flow.md):
```
PATCH /prweb/app/KnowledgeBuddy/api/application/v2/assignments/ASSIGN-WORKLIST%20PEGAFW-QNA-WORK%20BUDDY-1013!PROMPT_FLOW/actions/ConfigurePrompt?viewType=form
{
  "content": {...},
  "pageInstructions": [...]
}
```

---

## Stage Transition

After successful submission:
- Case transitions from **Prompt (PRIM1)** → **Resolve (PRIM2)**
- Status changes to **Resolved-Completed**
- Buddy is now active and ready for use

**No further assignments**: Case auto-resolves, no manual resolution needed.

---

## Response Handling

**Extract from Response**:
- Final case status: `data.caseInfo.status` (should be "Resolved-Completed")
- Case ID: `data.caseInfo.ID`
- Business ID: `data.caseInfo.businessID`

---

## Content Fields Reference

| Field | Type | Required | Default | Example |
|-------|------|----------|---------|---------|
| SystemMessage | string | Yes | See default-prompts.md | "You are a customer service..." |
| UserMessage | string | Yes | See default-prompts.md | "CONTEXT:\n######\n{SEARCHRESULTS}..." |
| GenAIModelId | string | Yes | "" (default) | "azure/openai/GPT-4o/2024-08-06" |
| OutputFormat | string | No | "Custom" | "JSON" or "Custom" |
| IncludeTextReplacements | boolean | No | false | true |
| IncludeAutoFiltering | boolean | No | false | true |

---

## Common Patterns

### Pattern 1: Default Everything
```javascript
content: {
  SystemMessage: "<default>",
  UserMessage: "<default>",
  GenAIModelId: "",
  OutputFormat: "Custom",
  IncludeTextReplacements: false,
  IncludeAutoFiltering: false
},
pageInstructions: []
```

### Pattern 2: Custom Prompts Only
```javascript
content: {
  SystemMessage: "<custom>",
  UserMessage: "<custom>",
  GenAIModelId: "",
  OutputFormat: "Custom",
  IncludeTextReplacements: false,
  IncludeAutoFiltering: false
},
pageInstructions: []
```

### Pattern 3: Custom Everything
```javascript
content: {
  SystemMessage: "<custom>",
  UserMessage: "<custom>",
  GenAIModelId: "azure/openai/GPT-4o/2024-08-06",
  OutputFormat: "Custom",
  IncludeTextReplacements: true,
  IncludeAutoFiltering: true
},
pageInstructions: [/* complex context config */]
```

---

## Next Step

[09-verify-success.md](09-verify-success.md) - Verify case status and report buddy URL to user
