# Initial User Input - Clone Existing Buddy

## Overview

Gather initial input when cloning an existing buddy configuration. This path copies configuration from an existing buddy (access, prompts, context, GenAI model).

## Prerequisites

- D_OriginalBuddyList query completed (see [01-on-demand-queries.md](01-on-demand-queries.md))

---

## User Input

**Single AskUserQuestion with 3 questions**:

```javascript
AskUserQuestion({
  questions: [
    {
      question: "What is the buddy name?",
      header: "Name",
      multiSelect: false,
      options: [
        { label: "Enter name", description: "Type your custom buddy name" },
        { label: "Leave empty", description: "No name (not recommended)" }
      ]
    },
    {
      question: "What is the buddy description? (Optional)",
      header: "Description",
      multiSelect: false,
      options: [
        { label: "Enter description", description: "Type your custom description" },
        { label: "Leave empty", description: "No description" }
      ]
    },
    {
      question: "Which buddy do you want to clone from?",
      header: "Clone source",
      multiSelect: false,
      options: [
        { label: "Customer Support Buddy", description: "ID: BUDDY-1" },
        { label: "HR Policy Buddy", description: "ID: BUDDY-2" }
        // ... populated from D_OriginalBuddyList query
      ]
    }
  ]
})
```

**Important**: For Questions 1 and 2, users should select "Type something" (automatically provided option) to enter their custom text. The provided options are just placeholders for the required schema.

**Capture**:
- Question 1: Buddy name (user enters custom text)
- Question 2: Buddy description (user enters custom text, optional)
- Question 3: Original buddy pyID from selected option (e.g., "BUDDY-1")

---

## Output Variables

After this step, store:

| Variable | Type | Required | Example |
|----------|------|----------|---------|
| buddyName | string | Yes | "Customer Support Buddy" |
| buddyDescription | string | No | "Answers customer support questions" |
| isCloned | boolean | Yes | true |
| originalBuddyId | string | Yes | "BUDDY-1" |

---

## API Call Format Example

These values will be used in Step 4 (Create assignment submission):

```json
{
  "Name": "Customer Support Buddy",
  "pyDescription": "Answers customer support questions",
  "IsCloned": true,
  "OriginalBuddyId": "BUDDY-1"
}
```

---

## Next Step

[03-create-case.md](03-create-case.md) - Create the Knowledge Buddy case
