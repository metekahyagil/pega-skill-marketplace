# Pega Skill Marketplace

A collection of Claude Code skills for Pega platform development and automation.

## Available Skills

| Skill | Description |
|-------|-------------|
| [create-knowledge-content](./skills/create-knowledge-content/) | Automated workflow for creating knowledge base content in Pega Knowledge Buddy |

## Prerequisites

1. **Claude Code** - Download from [claude.ai/code](https://claude.ai/code)

2. **Pega DX MCP Server** - Install globally:
   ```bash
   npm install -g @marco-looy/pega-dx-mcp
   ```

3. **Pega Instance** - Active Pega Infinity instance with DX API enabled

## Installation

### Step 1: Configure MCP Server

Add Pega DX MCP Server to Claude Code configuration file:

**Location**:
- Linux/Mac: `~/.config/claude-code/mcp_settings.json`
- Windows: `%APPDATA%/claude-code/mcp_settings.json`

**Configuration**:
```json
{
  "mcpServers": {
    "pega-dx-mcp": {
      "command": "pega-dx-mcp",
      "env": {
        "PEGA_BASE_URL": "https://your-pega-instance.com",
        "PEGA_CLIENT_ID": "your-oauth-client-id",
        "PEGA_CLIENT_SECRET": "your-oauth-client-secret",
        "PEGA_API_VERSION": "v2"
      }
    }
  }
}
```

Restart Claude Code after adding the configuration.

### Step 2: Install Skills from Marketplace

**Option A: Install via Marketplace Command** (Recommended)

In Claude Code, run:
```
/plugin marketplace add https://github.com/metekahyagil/pega-skill-marketplace
```

Then restart Claude Code to load the skills.

**Option B: Install as User Skills** (Available in all projects)

```bash
# Clone the marketplace
git clone https://github.com/metekahyagil/pega-skill-marketplace.git

# Copy skills to user directory
cp -r pega-skill-marketplace/skills/* ~/.claude/skills/
```

**Option C: Install as Project Skills** (Available only in specific project)

```bash
# Navigate to your project directory
cd /path/to/your/project

# Clone the marketplace
git clone https://github.com/metekahyagil/pega-skill-marketplace.git

# Copy skills to project
cp -r pega-skill-marketplace/skills/* .claude/skills/
```

### Step 3: Verify Installation

In Claude Code:
```
# List available skills
/skills

# Test a skill
/create-knowledge-content
```

## Usage

Skills can be invoked in two ways:

**1. Direct Command:**
```
/create-knowledge-content
```

**2. Natural Language:**
```
"Create a new article about Model Context Protocol in the knowledge base"
"Upload this policy document to the knowledge base"
```

Claude will automatically select and use the appropriate skill.

## Contributing

To contribute a new skill:

1. Fork this repository
2. Create your skill in `skills/your-skill-name/`
   - Required: `SKILL.md` (skill definition)
   - Recommended: `README.md` (user documentation)
3. Test thoroughly with real Pega instances
4. Update this README to list your skill
5. Submit a pull request

### Skill Structure

```
skills/
└── your-skill-name/
    ├── SKILL.md          # Required: Skill definition
    ├── README.md         # Recommended: User guide
    └── references/       # Optional: Supporting docs
```

## Support

- **Issues**: [GitHub Issues](https://github.com/metekahyagil/pega-skill-marketplace/issues)
- **Questions**: [GitHub Discussions](https://github.com/metekahyagil/pega-skill-marketplace/discussions)

When reporting issues, include:
- Skill name
- Error messages
- Pega version
- Relevant case IDs

## License

MIT

---

**Disclaimer**: This is an experimental community project. Not an official Pegasystems product.
