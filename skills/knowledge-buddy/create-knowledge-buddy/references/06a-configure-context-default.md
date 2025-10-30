# Configure Context Data - Use Defaults

## Overview

Keep SEARCHRESULTS context with default values. No collection or data sources configured. This is the simplest and fastest setup option.

---

## Default Values

| Setting | Default Value | Description |
|---------|---------------|-------------|
| Name | "SEARCHRESULTS" | Context variable name |
| Collection | null (empty) | No collection assigned |
| MustReturnResults | true | Fail if no results found |
| MaxChunk | 5 | Maximum number of chunks |
| MaxChunkTotalSize | 5000 | Maximum total size of chunks |
| DataSources | null (empty) | No data sources assigned |
| ResponseAttributes | null (empty) | No attributes returned |
| MinSimilarityScore | 80 | Minimum similarity threshold (0-100) |

---

## pageInstructions

UPDATE with defaults:

```javascript
pageInstructions: [
  {
    instruction: "UPDATE",
    target: ".ContextDataConfigs",
    listIndex: 1,
    content: {
      Name: "SEARCHRESULTS",
      Collection: { pyID: "" },  // Empty collection
      MustReturnResults: true,
      MaxChunk: 5,
      MaxChunkTotalSize: 5000,
      DataSourcesCSV: "",
      ResponseAttributesCSV: "",
      MinSimilarityScore: 80
    }
  }
]
```

---

## Output Variables

After this step, store:

| Variable | Type | Example |
|----------|------|---------|
| contextConfigs | array | `[{ Name: "SEARCHRESULTS", Collection: { pyID: "" }, ... }]` |
| pageInstructions | array | Array with single UPDATE instruction |

These will be combined with prompts and GenAI settings in final submission (see [08-submit-prompt.md](08-submit-prompt.md)).

---

## Next Step

Configure GenAI model settings. User will choose one of two paths:
- [07a-configure-genai-default.md](07a-configure-genai-default.md) - Use default GenAI model
- [07b-configure-genai-custom.md](07b-configure-genai-custom.md) - Select specific model
