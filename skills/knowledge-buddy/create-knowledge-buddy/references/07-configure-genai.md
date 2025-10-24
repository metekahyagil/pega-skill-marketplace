# Step 7: Configure GenAI Settings (Advanced Mode)

## Purpose

Select GenAI model and configure output settings in Advanced mode.

## GenAI Configuration

Present models from D_bxGenAIModelsForBuddy (Step 1) and gather user preferences:

### Model Selection

- **Pega-Default** (pyID: ""): Pega's default GenAI model
- **Custom models**: OpenAI, Azure, etc. (pyID: "gpt-4", etc.)

### Settings Based on Model

**For all models**:
- **OutputFormat**: "Custom" (free text) or "JSON" (structured)
- **IncludeTextReplacements**: Apply text replacements (boolean)
- **IncludeAutoFiltering**: Apply auto-filtering (boolean)

## Content Fields

```javascript
content: {
  SystemMessage: "<user-reviewed or default>",
  UserMessage: "<user-reviewed or default>",
  GenAIModelId: "gpt-4",  // or "" for default
  OutputFormat: "Custom",  // or "JSON"
  IncludeTextReplacements: true,
  IncludeAutoFiltering: false
}
```

## Final Submission

Combine prompt content with context data pageInstructions from Step 6:

```javascript
mcp__pega-dx-mcp__perform_assignment_action({
  assignmentID: "ASSIGN-WORKLIST PEGAFW-QNA-WORK BUDDY-1002!PROMPT_FLOW",
  actionID: "ConfigurePrompt",
  content: {
    SystemMessage: "...",
    UserMessage: "...",
    GenAIModelId: "gpt-4",
    OutputFormat: "Custom",
    IncludeTextReplacements: true,
    IncludeAutoFiltering: false
  },
  pageInstructions: [
    /* ContextDataConfigs from Step 6 */
  ]
})
```

Case auto-transitions: Prompt (PRIM1) → Resolve (PRIM2)
Status: Resolved-Completed

## Next Step

[08-verify-success.md](08-verify-success.md)
