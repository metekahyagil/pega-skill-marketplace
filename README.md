# Pega Skill Marketplace

A collection of Claude Code skills for Pega platform development and automation.

## Skills

### create-knowledge-content

Automated workflow for creating knowledge base content articles in Pega Knowledge Buddy applications.

**Features:**
- ✅ Complete end-to-end content creation workflow
- ✅ Support for both text and file content (PDF, DOCX, TXT, MD)
- ✅ Collection and data source configuration
- ✅ Access control and role management
- ✅ Chunking configuration for text content
- ✅ Verified working implementation with successful test cases

**Use Cases:**
- Creating new knowledge base articles
- Ingesting documentation and policies
- Uploading documents for automatic text extraction
- Managing content access permissions

**Requirements:**
- Pega Knowledge Buddy application installed
- Pega DX MCP Server configured and running
- Valid OAuth2 credentials for Pega instance

## Installation

### Using Claude Code

1. **Install as Plugin** (Recommended):
   ```bash
   # In Claude Code, add this repository as a plugin source
   # The skill will be automatically available
   ```

2. **Or Copy to Project**:
   ```bash
   # Copy the skill directory to your project
   cp -r skills/create-knowledge-content /path/to/your/project/.claude/skills/
   ```

### Prerequisites

You must have the [Pega DX MCP Server](https://github.com/marco-looy/pega-dx-mcp) installed and configured:

```bash
npm install -g @marco-looy/pega-dx-mcp
```

Configure your `.env` file:
```bash
PEGA_BASE_URL=https://your-pega-instance.com
PEGA_CLIENT_ID=your-oauth-client-id
PEGA_CLIENT_SECRET=your-oauth-client-secret
PEGA_API_VERSION=v2
```

## Usage

Once installed, invoke the skill in Claude Code:

```
/create-knowledge-content
```

Or use the skill programmatically when users request:
- "Create a new article about X"
- "Add content to the knowledge base"
- "Upload this document to the knowledge base"

The skill will guide you through:
1. Verifying Pega connection
2. Selecting collection and data source
3. Providing content (text or file)
4. Configuring access permissions
5. Submitting for ingestion

## Verified Test Cases

This skill has been thoroughly tested with successful ingestion:

- **KB-1055**: DOCX file → Resolved-Published ✅
- **KB-1056**: MD file → Resolved-Published ✅

## Contributing

Contributions are welcome! Please:

1. Fork this repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request with clear description

## Support

For issues or questions:
- Open an issue on GitHub
- Provide relevant case IDs and error messages
- Include your Pega version and configuration

## License

MIT

## Credits

Created for use with:
- [Pega DX MCP Server](https://github.com/marco-looy/pega-dx-mcp) by Marco Looy
- [Claude Code](https://claude.ai/code) by Anthropic
- Pega Knowledge Buddy application

---

**Note**: This is an experimental project. Not an official Pegasystems product.
