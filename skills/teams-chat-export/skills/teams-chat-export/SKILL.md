# Teams Chat Export

Export Teams meeting chat conversations to clean markdown files.

## Prerequisites

| Requirement | Required | Notes |
|-------------|----------|-------|
| **GitHub Copilot CLI** | Yes | Any project folder — no framework dependencies |
| **WorkIQ MCP** (`workiq`) | Yes | Used to find meetings and retrieve chat messages from M365 |
| **Teams MCP** (`microsoft-teams`) | Optional | Enables direct `ListChatMessages` for richer retrieval; WorkIQ is the fallback |
| **Microsoft 365 account** | Yes | With Teams access and meeting chat history |

This skill is **framework-independent** — it works with plain Copilot CLI invoked from any project folder. It requires no custom scripts, no Python, no Playwright, and no local caching. All retrieval is done through MCP tools.

## Triggers

Invoke this skill when the user asks to:
- "download a Teams meeting chat"
- "export a meeting conversation"
- "save the chat from a Teams meeting"
- "get the meeting chat messages"
- "teams chat export"

## Workflow

### Step 1: Gather Inputs

Ask the user for the following (skip any the user already provided):

| Input | Required | Example |
|-------|----------|---------|
| **Meeting title** | Yes | "Contoso Quarterly Business Review" |
| **Meeting date** | Yes | March 24, 2026 |
| **Output filename** | No — auto-generate if omitted | `2026-03-24 Contoso QBR Chat.md` |

**Filename convention** (when auto-generating):
```
YYYY-MM-DD <short descriptive topic> Chat.md
```

Rules for the short topic:
- Extract the most distinctive part of the meeting title (skip generic prefixes like "Microsoft Innovation Hub")
- Keep it under 5 words
- Use title case

Examples:
| Meeting Title | Auto-generated Filename |
|---------------|------------------------|
| Microsoft Innovation Hub Irvine Welcomes Contoso | `2026-03-24 Contoso Chat.md` |
| Weekly Engineering Standup | `2026-04-01 Engineering Standup Chat.md` |
| Contoso ADS - Azure Migration Planning | `2026-04-10 Contoso Azure Migration Chat.md` |

### Step 2: Find the Meeting Chat

Use WorkIQ to locate the meeting and its chat messages:

```
workiq-ask_work_iq:
  "Find the Teams meeting chat from [date] titled '[meeting title]'.
   I need the chat ID or link to the meeting conversation."
```

WorkIQ will return:
- Meeting metadata (title, date, time, organizer)
- Chat ID (embedded in Teams URLs in the response)
- Confirmation that the meeting chat exists

**Extract the chat ID** from any Teams URL in the response. The chat ID format for meeting chats is:
```
19:meeting_<base64>@thread.v2
```

### Step 3: Retrieve Chat Messages

**Primary method — WorkIQ** (always available):

```
workiq-ask_work_iq:
  "Show me all the chat messages from the Teams meeting chat '[meeting title]'
   on [date]. List each message with the sender name, timestamp, and full
   message text. Include every message."
```

If WorkIQ indicates there are more messages than shown, make a follow-up call:

```
workiq-ask_work_iq:
  "Continue showing the remaining Teams chat messages from '[meeting title]'
   on [date]. Show any messages not yet listed."
```

**Enhanced method — Teams MCP** (when available):

If `microsoft-teams-ListChatMessages` is available AND you have the chat ID from Step 2:

1. Call `microsoft-teams-ListChats` with `topic: "<meeting title>"` and `top: 20` to satisfy the MCP tool chain requirement
2. Call `microsoft-teams-ListChatMessages` with the chat ID, `top: 50`, `orderby: "createdDateTime asc"`
3. If more than 50 messages, paginate with subsequent calls

The Teams MCP method returns richer data (exact timestamps, message IDs, HTML content). Fall back to WorkIQ if the Teams MCP is unavailable or times out.

### Step 4: Get Attendees

```
workiq-ask_work_iq:
  "For the Teams meeting '[meeting title]' on [date],
   who were all the attendees/participants? List their names and organizations."
```

### Step 5: Format and Save

Compile all data into a markdown file using this template:

```markdown
# <Meeting Title>

**Meeting Chat Conversation**

| Field | Detail |
|-------|--------|
| **Date** | <full date> |
| **Time** | <start time> – <end time> <timezone> |
| **Organizer** | <name> — <organization> |
| **Participants** | <name — org>; <name — org>; ... |
| **Chat ID** | `<chat ID>` |

---

## Chat Messages

### <Sender Name>

<message content — use code blocks for commands/code, plain text otherwise>

---

### <Sender Name>

<next message>

---

*<N> messages total — extracted from Teams meeting chat via <method used>*
```

**Formatting rules:**
- Use `### Sender Name` for each message (H3 heading)
- Wrap CLI commands, code, or structured data in triple-backtick code blocks
- Keep plain conversational messages as regular text
- Separate each message with a horizontal rule (`---`)
- Include a footer line with message count and retrieval method

**Save location:**
- Default: `outputs/` directory (create if it doesn't exist)
- If `outputs/` doesn't exist and the user hasn't specified a path, save in the current working directory
- Always confirm the save path with the user if it differs from their request

### Step 6: Report Results

Confirm to the user with:
- File path where the chat was saved
- Message count
- Participant list
- Date/time of the meeting

## Error Handling

| Scenario | Action |
|----------|--------|
| WorkIQ can't find the meeting | Ask user to verify the exact meeting title and date. Try partial title match. |
| WorkIQ returns partial messages | Make follow-up WorkIQ calls to retrieve remaining messages. |
| Teams MCP times out | Fall back to WorkIQ (primary method). Note the fallback in the output footer. |
| Teams MCP lacks `Chat.Read` scope | Fall back to WorkIQ. This is expected for delegated Graph API tokens. |
| No messages found | Inform the user — the meeting may not have had chat activity. |
| Chat ID not extractable | Proceed with WorkIQ message retrieval without the chat ID. |

## Limitations

- **WorkIQ may not return exact timestamps** for each message — only the meeting date. If exact timestamps are needed, the Teams MCP method is required.
- **WorkIQ search is semantic** — it returns the best matches, which may occasionally miss very short messages (e.g., emoji-only reactions).
- **Meeting chats from very old meetings** (>90 days) may have reduced availability depending on retention policies.
- **External participant messages** may show display names without organization affiliation.
