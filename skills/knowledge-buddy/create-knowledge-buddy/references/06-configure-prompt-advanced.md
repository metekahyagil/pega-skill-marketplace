# Step 6: Configure Context Data (Advanced Mode)

## Purpose

Configure context data sources and parameters for semantic search in Advanced mode.

## Context Data Configuration

System initializes one SEARCHRESULTS context. In Advanced mode:

1. Present default configuration to user
2. Allow user to select Collection from D_IndexList (Step 1)
3. Allow customization of parameters:
   - **MaxChunk**: Max chunks to return (default: 5)
   - **MaxChunkTotalSize**: Max total size (default: 5000)
   - **MinSimilarityScore**: Similarity threshold 0-100 (default: 80)
   - **MustReturnResults**: Fail if no results (default: true)
4. Option to add additional context definitions (APPEND instruction)

## PageInstructions Pattern

Use UPDATE to modify first context:

```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: {
        pyID: "DC-123",  // User-selected collection
        pzInsKey: "PEGAFW-QNA-WORK DC-123"
      },
      MustReturnResults: true,
      MaxChunk: 10,  // User-customized
      MaxChunkTotalSize: 8000,
      MinSimilarityScore: 75,
      DataSourcesCSV: "",
      ResponseAttributesCSV: ""
    }
  }
]
```

## Adding Additional Context

Use APPEND for new context definitions:

```javascript
{
  instruction: "APPEND",
  target: ".ContextDataConfigs",
  content: {
    Name: "ADDITIONALCONTEXT",
    Collection: { pyID: "DC-456" },
    // ... other settings
  }
}
```

## Next Steps

- **For Simple Mode**: Submit with these pageInstructions (see [05-configure-prompt-simple.md](05-configure-prompt-simple.md))
- **For Advanced Mode**: Continue to [07-configure-genai.md](07-configure-genai.md) for GenAI model selection
