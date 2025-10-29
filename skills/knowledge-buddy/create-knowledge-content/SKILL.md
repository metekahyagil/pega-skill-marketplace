---
name: create-knowledge-content
description: Simplified workflow for creating knowledge base content articles in Pega Knowledge Buddy applications. Use this skill when the user wants to create, author, and ingest content into a knowledge base with collection, data source, access configuration, and chunking settings. Handles the complete end-to-end workflow from case creation through content ingestion with support for multiple content chunks and access roles.
---

# Create Knowledge Content

## Overview

Streamlined workflow for creating knowledge base content articles in Pega Knowledge Buddy. Create a Content case, progressively configure collection and data source associations, set access roles, configure chunking parameters, author content (text or file), and submit for ingestion into the knowledge base vector store.

## When to Use This Skill

- Creating new knowledge base articles or content
- Adding content to existing collections and data sources
- Ingesting documentation, policies, or reference materials
- Uploading documents for automatic text extraction and indexing
- User requests like "Create article about X", "Add content to knowledge base", or "Upload document to knowledge base"

## Workflow

### Step 0: Verify Pega Connection

Authenticate and verify connection to Pega before starting the workflow.

**Reference**: [00-connection.md](references/00-connection.md)

---

### Step 1: Create Content Case

Create a new Content case using the `PegaFW-KB-Work-Article` case type to initiate the workflow.

**Reference**: [01-create-case.md](references/01-create-case.md)

---

### Step 2: Query and Select Collection

Fetch available collections from the system and use `AskUserQuestion` tool to allow user to select the target collection.

**Reference**: [02-query-collections.md](references/02-query-collections.md)

---

### Step 3: Refresh Assignment

Refresh the assignment after case creation to prepare for configuration.

**Reference**: [03-refresh-assignment.md](references/03-refresh-assignment.md)

---

### Step 4: Query and Select Data Source

Fetch data sources filtered by the selected collection ID and allow user to select the target data source.

**Reference**: [04-query-datasources.md](references/04-query-datasources.md)

---

### Step 5: Query and Select Access Roles

Fetch available access roles and use `AskUserQuestion` tool with multi-select to allow user to choose one or more access roles.

**Reference**: [05-query-access-roles.md](references/05-query-access-roles.md)

---

### Step 6: Configure Chunking Settings

Use `AskUserQuestion` tool to determine if user wants advanced settings. If No, use defaults. If Yes, collect chunking method, size, and overlap.

**Reference**: [06-configure-chunking.md](references/06-configure-chunking.md)

---

### Step 7: Submit Configuration

Submit the Create action using pageInstructions to associate collection, data source, access roles, and optionally chunking parameters.

**Reference**: [07-submit-configuration.md](references/07-submit-configuration.md)

---

### Step 8: Refresh After Configuration

Refresh the case view after configuration is submitted.

**Reference**: [08-refresh-configuration.md](references/08-refresh-configuration.md)

---

### Step 9: Author Content

Use `AskUserQuestion` tool to select content format (Text or File). Use `AskUserQuestion` tool to collect title, abstract, and content/file path. Submit authoring action.

**Reference**: [09-author-content.md](references/09-author-content.md)

---

### Step 10: Verify and Display

Retrieve final case summary and details. Summarize the information and display to user including case ID and status.

**Reference**: [10-verify-display.md](references/10-verify-display.md)

---

## Technical References

- **[technical-details.md](references/technical-details.md)** - Case types, assignment actions, MCP tools reference
- **[pageInstructions-patterns.md](references/pageInstructions-patterns.md)** - PageInstructions rules and patterns
- **[examples.md](references/examples.md)** - Complete working examples with actual values
- **[default-settings.md](references/default-settings.md)** - Default chunking settings and configuration modes
