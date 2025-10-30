# Configure GenAI Model - Select Specific Model

## Overview

Select specific GenAI model (OpenAI, Azure, Claude, etc.) with custom output format and filtering options.

---

## Prerequisites

- D_bxGenAIModelsForBuddy query completed

See [01-on-demand-queries.md](01-on-demand-queries.md) for query details:

```javascript
mcp__pega-dx-mcp__get_list_data_view({
  dataViewID: "D_bxGenAIModelsForBuddy",
  dataViewParameters: { ModelId: "" }
  // ... see 01-on-demand-queries.md for full query
})
```

---

## Step 1: Select Model

**Atomic question**: "Select GenAI model"

```javascript
AskUserQuestion({
  questions: [{
    question: "Select GenAI model:",
    header: "Model",
    multiSelect: false,
    options: [
      { label: "GPT-4o", description: "ID: azure/openai/GPT-4o/2024-08-06" },
      { label: "GPT-3.5 Turbo", description: "ID: gpt-3.5-turbo" },
      { label: "Claude", description: "ID: anthropic/claude-3" }
      // ... populated from D_bxGenAIModelsForBuddy query
    ]
  }]
})
```

**Store**: Selected model's `pyID` (e.g., "azure/openai/GPT-4o/2024-08-06")

**Extract**: `ModelType` from query result (check if "GPT" to determine if output format is needed)

---

## Step 2: Output Format (if GPT model selected)

**Only ask if selected model's ModelType contains "GPT"**

**Atomic question**: "Select output format"

```javascript
AskUserQuestion({
  questions: [{
    question: "Select output format for GPT model:",
    header: "Output format",
    multiSelect: false,
    options: [
      {
        label: "JSON",
        description: "Structured JSON output format"
      },
      {
        label: "Custom",
        description: "Free-form text output"
      }
    ]
  }]
})
```

**Store**: `OutputFormat = "JSON"` or `"Custom"`

**If not GPT model**: Default to `OutputFormat = "Custom"`

---

## Step 3: Text Replacements

**Atomic question**: "Apply text replacements to user requests?"

```javascript
AskUserQuestion({
  questions: [{
    question: "Apply text replacements to user requests?",
    header: "Text replace",
    multiSelect: false,
    options: [
      {
        label: "Yes",
        description: "Apply text replacement rules to incoming questions"
      },
      {
        label: "No",
        description: "Use user questions as-is"
      }
    ]
  }]
})
```

**Store**: `IncludeTextReplacements = true` or `false`

---

## Step 4: Auto Filtering

**Atomic question**: "Apply auto filtering to user requests?"

```javascript
AskUserQuestion({
  questions: [{
    question: "Apply auto filtering to user requests?",
    header: "Auto filter",
    multiSelect: false,
    options: [
      {
        label: "Yes",
        description: "Apply automatic filtering rules to questions"
      },
      {
        label: "No",
        description: "No automatic filtering"
      }
    ]
  }]
})
```

**Store**: `IncludeAutoFiltering = true` or `false`

---

## Output Variables

After this step, store:

| Variable | Type | Example |
|----------|------|---------|
| genAIModelId | string | "azure/openai/GPT-4o/2024-08-06" |
| outputFormat | string | "JSON" or "Custom" |
| includeTextReplacements | boolean | true |
| includeAutoFiltering | boolean | true |

---

## Model Types Reference

| Model Type | Output Format Required | Example Models |
|------------|------------------------|----------------|
| **GPT** | Yes (JSON or Custom) | GPT-4o, GPT-3.5 Turbo |
| **Other** | No (defaults to Custom) | Claude, PaLM, Llama |

---

## Configuration Summary Example

**Custom GPT model setup**:
```javascript
{
  GenAIModelId: "azure/openai/GPT-4o/2024-08-06",
  OutputFormat: "Custom",
  IncludeTextReplacements: true,
  IncludeAutoFiltering: true
}
```

**API Equivalent** (query models):
```
POST /prweb/app/KnowledgeBuddy/api/application/v2/data_views/D_bxGenAIModelsForBuddy
{
  "dataViewParameters": {"ModelId": ""}
}
```

---

## Next Step

[08-submit-prompt.md](08-submit-prompt.md) - Combine all configuration and submit ConfigurePrompt assignment
