# Teams Chat Export

Export Teams meeting chat conversations to clean markdown files.

## Quick Start

1. Copy the `teams-chat-export/` folder into your project's `skills/` directory
2. Ensure you have **WorkIQ MCP** configured in your `.mcp.json` or `.vscode/mcp.json`
3. Ask Copilot: *"Download the Teams chat from the March 24 meeting titled 'Project Kickoff'"*

## Prerequisites

- **GitHub Copilot CLI** — any project folder
- **WorkIQ MCP** (`workiq`) — required for M365 search
- **Teams MCP** (`microsoft-teams`) — optional, enhances retrieval with exact timestamps
- **Microsoft 365 account** with Teams access

No Python, no scripts, no framework dependencies. Pure instruction-driven skill.

## What It Does

1. Finds the meeting via WorkIQ search
2. Retrieves all chat messages with sender attribution
3. Gets the attendee list
4. Formats everything as a clean markdown file
5. Saves to `outputs/` with an auto-generated filename (`YYYY-MM-DD <topic> Chat.md`)

## Example Output

```
outputs/2026-03-24 Contoso QBR Chat.md
```

## License

MIT
