# Step 5: Configure Prompts (Simple Mode)

## Purpose

Submit Configure Prompt assignment with default settings. This is the final step - case auto-resolves after submission.

## MCP Tool

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!PROMPT_FLOW",
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
        Collection: { pyID: "" },  // Empty = no collection yet
        MustReturnResults: true,
        MaxChunk: 5,
        MaxChunkTotalSize: 5000,
        DataSourcesCSV: "",
        ResponseAttributesCSV: "",
        MinSimilarityScore: 80
      }
    }
  ]
})
```

## Fields

**SystemMessage**: GenAI instructions (see [default-prompts.md](default-prompts.md))
**UserMessage**: Prompt template with `{SEARCHRESULTS}` and `{QUESTION}` placeholders
**GenAI Settings**: Empty model ID (uses Pega default), Custom output format, no text replacements or filtering

**ContextDataConfigs**: Default SEARCHRESULTS with empty collection reference (configure later)

## Result

Case auto-transitions: Prompt (PRIM1) → Resolve (PRIM2)
Status: Resolved-Completed

## Response Format

See [response-structures.md](response-structures.md)

## Next Step

[08-verify-success.md](08-verify-success.md)
