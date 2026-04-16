# Hub Close-Out Email

Generate professional thank-you emails for Innovation Hub engagement close-outs.

## Quick Start

1. Copy the `hub-close-out-email/` folder into your project's `skills/` directory
2. Ask Copilot: *"Write a close-out email for the Contoso engagement on April 10"*

## Prerequisites

- **GitHub Copilot CLI** — any project folder
- **send-email skill** — optional, for sending the email directly from the CLI

No external APIs, no scripts, no framework dependencies. Pure instruction-driven skill.

## What It Does

1. Gathers session details (customer, date, topic, session length, future plans)
2. Composes a professional 2–3 paragraph thank-you email with:
   - Warm acknowledgment of the session
   - Reference to recap/action items (placeholder for attachment)
   - Forward-looking statement based on next steps
3. Saves as markdown with metadata header (subject, recipients, date)
4. Optionally sends via the `send-email` skill

## Inputs

| Input | Required | Example |
|-------|----------|---------|
| Customer name | Yes | "Contoso" |
| Session date | Yes | "March 31, 2026" |
| Session length | Yes | single session / one day / multi-day |
| Topic | Yes | "Azure AI Architecture Design Session" |
| Future sessions planned | Yes | Yes/No + description |

## Output

```
outputs/CONTOSO-Hub-Close-Out-Email-2026-04-10.md
```

Markdown file with email metadata header, professional body text, and a recap placeholder section.

## License

MIT
