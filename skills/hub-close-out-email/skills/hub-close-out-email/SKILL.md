---
name: hub-close-out-email
description: Use this skill when the user asks to "write a close-out email", "Hub close-out", "thank-you email for a Hub session", "session follow-up email", "engagement close-out", or wants to generate a professional post-session thank-you email for an Innovation Hub customer engagement. Generates a markdown email and saves it to the outputs folder.
---

Generate a professional Innovation Hub session close-out thank-you email in markdown and save it to the `outputs/` folder.

## Overview

This skill generates a concise, professional thank-you email on behalf of the Irvine Innovation Hub team following a customer engagement session. The email is saved as a markdown file in the `outputs/` folder for review, editing, and eventual sending via the `send-email` skill.

## Workflow

### Step 1: Gather Inputs

Prompt the user for the following inputs. Use the `ask_user` tool for each — do not assume or fabricate any values.

1. **Customer name** — The customer or organization name (e.g., "Contoso", "Northwind Traders")
2. **Session date** — The date of the session (e.g., "March 31, 2026")
3. **Session length** — One of:
   - `single session` (half-day or less)
   - `one day` (full-day session)
   - `multi-day` (2+ day engagement)
4. **Topic of discussion** — A short, abbreviated description of the session topic (e.g., "AI and Azure BCDR", "Data & Analytics Modernization", "Security and Zero Trust", "Azure Landing Zone & AI Platform Assessment")
5. **Future sessions planned?** — Yes or No
   - If **Yes**, ask: "What are the upcoming sessions about?" (brief description)

### Step 2: Compose the Email

Generate the email following these rules:

#### Structure

- **Subject line:** `Thank You — [TOPIC] Session | Microsoft Innovation Hub`
- **Greeting:** `Dear [CUSTOMER NAME] Team,`
- **Body:** 2–3 short paragraphs (see composition rules below)
- **Closing:** Professional sign-off from the Innovation Hub team

#### Composition Rules

**Paragraph 1 — Thank You:**
- Thank the customer for their time and engagement
- Adjust wording based on session length:
  - Single session: "Thank you for joining us at the Microsoft Innovation Hub for our [TOPIC] session on [DATE]."
  - One day: "Thank you for spending the day with us at the Microsoft Innovation Hub for our [TOPIC] session on [DATE]."
  - Multi-day: "Thank you for your partnership and commitment throughout our multi-day [TOPIC] engagement at the Microsoft Innovation Hub on [DATE]."
- Reference the collaborative and productive nature of the discussion

**Paragraph 2 — Recap & Action Items:**
- Reference that a meeting recap and captured action items are included below the email
- Note that account team members will follow up on their respective action items
- Keep this brief — do NOT restate or fabricate recap details; assume the recap will be pasted immediately after the email

**Paragraph 3 — Forward-Looking:**
- If **future sessions are planned**: Express enthusiasm and explicitly state you are looking forward to the upcoming sessions. Reference the topic of the next sessions if provided.
- If **no future sessions planned**: Express that you look forward to future engagements and supporting the customer's continued journey.

#### Tone

- Professional, appreciative, and customer-facing
- Concise — no filler, no fluff
- Warm but not overly casual

### Step 3: Save the Email

Save the composed email as a markdown file to:

```
outputs/[CUSTOMER-NAME]-Hub-Close-Out-Email-[DATE].md
```

Where:
- `[CUSTOMER-NAME]` is the customer name with spaces replaced by hyphens
- `[DATE]` is the session date in `YYYY-MM-DD` format

The markdown file should contain:
1. A metadata header with the subject line, to (left as `[RECIPIENTS]` placeholder), and date
2. The full email body in markdown
3. A separator line (`---`)
4. A placeholder section for the meeting recap: `## Meeting Recap & Action Items` with instructions to paste the recap below

### Step 4: Confirm and Offer to Send

After saving, inform the user:
- The file path where the email was saved
- Offer to send the email via the `send-email` skill: "Would you like me to send this email? If so, provide the recipient email addresses."

## Email Template

```markdown
# Hub Close-Out Email

**Subject:** Thank You — [TOPIC] Session | Microsoft Innovation Hub
**To:** [RECIPIENTS]
**Date:** [DATE]

---

Dear [CUSTOMER NAME] Team,

[PARAGRAPH 1 — Thank you, adjusted for session length]

[PARAGRAPH 2 — Recap reference and action item follow-up note]

[PARAGRAPH 3 — Forward-looking statement, adjusted for future sessions]

We truly value the opportunity to partner with you and support your technology objectives.

Warm regards,
The Microsoft Innovation Hub Team — Irvine

---

## Meeting Recap & Action Items

> Paste the meeting recap and action items below this line.

```

## Example Output

```markdown
# Hub Close-Out Email

**Subject:** Thank You — AI and Azure BCDR Session | Microsoft Innovation Hub
**To:** [RECIPIENTS]
**Date:** March 31, 2026

---

Dear Contoso Team,

Thank you for joining us at the Microsoft Innovation Hub for our AI and Azure BCDR session on March 31, 2026. It was a pleasure collaborating with your team, and the discussions around your architecture and business continuity objectives were both insightful and productive.

Below you will find the meeting recap along with the action items we captured during our time together. Members of the Microsoft account team will be following up on their respective items in the coming days.

We are excited about the upcoming follow-on architecture design session and look forward to continuing our work together to advance your AI and BCDR strategy on Azure.

We truly value the opportunity to partner with you and support your technology objectives.

Warm regards,
The Microsoft Innovation Hub Team — Irvine

---

## Meeting Recap & Action Items

> Paste the meeting recap and action items below this line.

```

## Composes With

- **send-email** — After generating the close-out email, offer to send it via Outlook
- **meeting-summary** — Use meeting summaries as the recap content to paste after the email
- **engagement-closeout** — Complements the CE Hub closeout skill with the customer-facing email

## Rules

- ALWAYS prompt the user for all required inputs — never fabricate customer names, dates, or topics
- ALWAYS save the email to the `outputs/` folder as markdown
- ALWAYS keep the email concise (2–3 short paragraphs) — do not add extra sections, bulleted lists, or lengthy content
- ALWAYS include the placeholder section for the meeting recap — do not invent recap content
- NEVER include specific technical details, recommendations, or action items in the email body — the recap handles that
- Use the customer's actual name, not a generic greeting
- Do not include internal Microsoft references (CSU, TPID, etc.) — this is customer-facing
