# Initial User Input - Create New Buddy

## Overview

Gather initial input when creating a new buddy from scratch with default configuration.

---

## User Input

**Single AskUserQuestion with 2 questions**:

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
    }
  ]
})
```

**Important**: Users should select "Type something" (automatically provided option) to enter their custom text. The provided options are just placeholders for the required schema.

**Capture**:
- Question 1: Buddy name (user enters custom text)
- Question 2: Buddy description (user enters custom text, optional)

---

## Output Variables

After this step, store:

| Variable | Type | Required | Example |
|----------|------|----------|---------|
| buddyName | string | Yes | "Customer Support Buddy" |
| buddyDescription | string | No | "Answers customer support questions" |
| isCloned | boolean | Yes | false |

---

## API Call Format Example

These values will be used in Step 4 (Create assignment submission):

```json
{
  "Name": "Customer Support Buddy",
  "pyDescription": "Answers customer support questions",
  "IsCloned": false
}
```

---

## Next Step

[03-create-case.md](03-create-case.md) - Create the Knowledge Buddy case
