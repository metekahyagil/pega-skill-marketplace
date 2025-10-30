# Configure GenAI Model - Use Default

## Overview

Use Pega's default GenAI model with optional text replacement and auto filtering settings. This is the fastest setup option.

---

## Configuration Values

When using default model:
- `GenAIModelId = ""`  (empty string)
- `OutputFormat = "Custom"`  (default for non-GPT models)

---

## Question 1: Text Replacements

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

## Question 2: Auto Filtering

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

| Variable | Type | Value | Example |
|----------|------|-------|---------|
| genAIModelId | string | "" | "" (empty) |
| outputFormat | string | "Custom" | "Custom" |
| includeTextReplacements | boolean | User choice | true |
| includeAutoFiltering | boolean | User choice | false |

---

## Configuration Summary Example

```javascript
{
  GenAIModelId: "",
  OutputFormat: "Custom",
  IncludeTextReplacements: false,
  IncludeAutoFiltering: false
}
```

---

## Next Step

[08-submit-prompt.md](08-submit-prompt.md) - Combine all configuration and submit ConfigurePrompt assignment
