# QA Plugin Setup

This plugin requires MCP (Model Context Protocol) servers to be configured in your Claude Code installation.

## Prerequisites

- Docker (for mcp-atlassian)
- Node.js (for playwright and mcp-gsheets)
- Google Cloud credentials (for mcp-gsheets)
- Jira/Confluence credentials (for mcp-atlassian)

## Setup Instructions

### 1. Add MCP Server Configuration

Add the following to your `.claude/config.json` in your project root:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["@playwright/mcp@latest"],
      "env": {}
    },
    "mcp-atlassian": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "CONFLUENCE_URL",
        "-e",
        "CONFLUENCE_USERNAME",
        "-e",
        "CONFLUENCE_API_TOKEN",
        "-e",
        "JIRA_URL",
        "-e",
        "JIRA_USERNAME",
        "-e",
        "JIRA_API_TOKEN",
        "ghcr.io/sooperset/mcp-atlassian:latest"
      ],
      "env": {
        "CONFLUENCE_URL": "${JIRA_URL}",
        "CONFLUENCE_USERNAME": "${JIRA_USERNAME}",
        "CONFLUENCE_API_TOKEN": "${JIRA_API_TOKEN}",
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_USERNAME": "${JIRA_USERNAME}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    },
    "mcp-gsheets": {
      "command": "npx",
      "args": ["-y", "mcp-gsheets@latest"],
      "env": {
        "GOOGLE_PROJECT_ID": "${GOOGLE_PROJECT_ID}",
        "GOOGLE_APPLICATION_CREDENTIALS": "${GOOGLE_APPLICATION_CREDENTIALS}"
      }
    }
  }
}
```

### 2. Set Environment Variables

Create a `.env` file or add to your shell profile:

```bash
# Jira/Confluence
export JIRA_URL="https://your-instance.atlassian.net"
export JIRA_USERNAME="your-email@domain.com"
export JIRA_API_TOKEN="your-api-token"

# Google Sheets
export GOOGLE_PROJECT_ID="your-project-id"
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/credentials.json"
```

### 3. Restart Claude Code

After adding the configuration, restart Claude Code for the changes to take effect.

## Verifying Setup

You can verify the MCP servers are working by using any of these agents:
- `jira-ticket-analyzer` - requires mcp-atlassian
- `playwright-code-generator` - requires playwright
- `google-sheets-manager` - requires mcp-gsheets

## Troubleshooting

If MCP tools aren't available:
1. Check that `.claude/config.json` exists and has the correct configuration
2. Verify environment variables are set
3. Ensure Docker is running (for mcp-atlassian)
4. Restart Claude Code
