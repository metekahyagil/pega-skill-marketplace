---
name: create-knowledge-buddy
description: Create and configure Knowledge Buddy cases in Pega Knowledge Buddy applications. Use this skill when the user wants to create a new knowledge buddy with configurable access control, GenAI prompts, context data definitions, and model settings. Supports both simple (default settings) and advanced (full customization) configuration modes.
---

# Create Knowledge Buddy

## Overview

This skill provides a streamlined workflow for creating Knowledge Buddy cases in Pega Knowledge Buddy applications. It automates the multi-step process of creating a Knowledge Buddy case, configuring access roles, setting up GenAI prompts and context data definitions, selecting GenAI models, and completing the buddy configuration.

The workflow supports two configuration modes:
- **Simple**: Uses default settings (recommended for quick setup)
- **Advanced**: Full customization of access roles, prompts, context definitions, and GenAI settings

The workflow also supports cloning from existing buddies to quickly replicate configurations.

## When to Use This Skill

- Creating new knowledge buddies for Q&A applications
- Setting up GenAI-powered assistants with custom prompts
- Configuring context data sources for semantic search
- Cloning existing buddy configurations
- User requests like "Create a new knowledge buddy", "Set up a Q&A assistant", or "Clone an existing buddy"

## Workflow

### Step 0: Verify Pega Connection

Authenticate with Pega and verify connectivity. See [00-connection.md](references/00-connection.md)

---

### Step 1: Query Available Data

Query collections, GenAI models, access roles, and existing buddies to present options to the user. See [01-query-data.md](references/01-query-data.md)

---

### Step 2: Gather User Input

Collect buddy name, description, settings mode (simple/advanced), and clone preferences. See [02-user-input.md](references/02-user-input.md)

---

### Step 3: Create Knowledge Buddy Case

Create case using `PegaFW-QnA-Work-KnowledgeQnA` case type. Capture caseID and assignmentID. See [03-create-case.md](references/03-create-case.md)

---

### Step 4: Configure Access Control (Secure Stage)

Submit Create assignment, then Configure Security assignment. Simple mode uses default roles, Advanced mode allows customization. Case transitions to Prompt stage. See [04-configure-secure.md](references/04-configure-secure.md)

---

### Step 5: Configure Prompts (Prompt Stage)

Submit Configure Prompt assignment with prompts, context data, and GenAI settings. Simple mode uses defaults, Advanced mode allows full customization. Case auto-resolves after submission.

See [05-configure-prompt-simple.md](references/05-configure-prompt-simple.md), [06-configure-prompt-advanced.md](references/06-configure-prompt-advanced.md), [07-configure-genai.md](references/07-configure-genai.md)

---

### Step 6: Verify Success

Retrieve final case status, verify Resolved-Completed, and report case ID and buddy URL to user. See [08-verify-success.md](references/08-verify-success.md)

---

## Technical References

### Core Patterns and Structures
- **[technical-details.md](references/technical-details.md)** - Case types, assignment actions, pageInstructions patterns, data structures, stage transitions, and MCP tools reference

### Default Configuration
- **[default-prompts.md](references/default-prompts.md)** - Default SystemMessage (Instructions) and UserMessage (Information) templates with placeholders

### Error Handling
- **[error-handling.md](references/error-handling.md)** - Common issues and solutions including validation errors, empty references, pageInstructions issues, and stage transition problems

### Working Examples
- **[examples.md](references/examples.md)** - Complete working examples for Simple mode (minimal config) and Advanced mode (full customization)

### Technical Dictionary
- **[dictionary.md](dictionary.md)** - Comprehensive reference of MCP tools, data pages, case types, and class IDs used in the workflow

---

## Configuration Modes

| Feature | Simple | Advanced |
|---------|--------|----------|
| **Access Roles** | Default | Customizable |
| **Prompts** | Default | Editable |
| **Context Data** | Empty | Select collection + customize |
| **GenAI Model** | Default | Select model + configure |

---

## Next Steps After Buddy Creation

After successful buddy creation:
- Buddy is available for use in the Knowledge Buddy portal
- Can be used to answer questions based on configured context
- Can be edited via "Edit configuration" action
- Can be cloned to create similar buddies
- Can have new versions created via "Create new version" action

For troubleshooting, error handling, or detailed technical information, refer to the comprehensive reference documents listed above.
