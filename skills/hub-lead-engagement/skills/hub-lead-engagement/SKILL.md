---
name: hub-lead-engagement
description: Use this skill when the user asks for "my engagements", "engagements I led", "hub lead engagements", "CEHub engagements", "what engagements did I lead", "list my sessions", "my Hub sessions", or wants to see Innovation Hub engagements where they were the Lead Technical Architect. Default date range is last 30 days.
---

# Hub Lead Engagement — My CEHub Engagements

List CEHub engagements where the current user was the **Lead Technical Architect**, with a configurable date range (default: last 30 days).

## Data Source

| Property | Value |
|----------|-------|
| Semantic Model ID | `[SEMANTIC_MODEL_ID]` |
| Semantic Model Name | MTC Survey - TA View |
| Source System | CEHub Dynamics 365 (`[CEHUB_URL]`) |
| User Identity | `user@example.com` via `EngagementArchitect[EmailAlias]` |

## Workflow

### Step 1: Determine Date Range

Ask the user if they want a custom date range. If not specified, default to **last 30 days** from today.

- Accept natural language like "last 60 days", "this quarter", "March 2026", "FY26 Q3"
- Convert to a start date in `YYYY, M, D` format for the DAX `DATE()` function
- The end date is always today (no future filter needed since we sort by date descending)

### Step 2: Execute DAX Query

Use `powerbi-remote-ExecuteQuery` with `artifactId: "[SEMANTIC_MODEL_ID]"`.

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'DimEvent'[msevtmgt_eventstartdate],
        'DimEvent'[msevtmgt_name],
        'DimEvent'[CustomerName],
        'DimEvent'[msevtmgt_eventtypename],
        'DimEvent'[msevtmgt_readableeventid],
        'DimEvent'[statuscodename],
        'DimEvent'[msevtmgt_buildingname]
    ),
    'DimEvent'[EDLPSource] = "CEHUB",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    'DimEvent'[msevtmgt_eventstartdate] <= DATE({end_year}, {end_month}, {end_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'EngagementArchitect'[EmailAlias] = "user@example.com",
    'EngagementArchitect'[IsLeadTA] = "Yes"
)
ORDER BY 'DimEvent'[msevtmgt_eventstartdate] DESC
```

Use `maxRows: 200`.

#### Important Filter Notes

- **Source:** Filter to `EDLPSource = "CEHUB"` only (not MTCCRM)
- **Status:** Do NOT filter on `statuscodename`. Include all statuses (Complete, Closeout, Confirmed, Qualified, etc.). Engagements in Closeout or Confirmed status are still valid and must be shown.
- **Lead TA:** Filter `IsLeadTA = "Yes"` AND `EmailAlias = "user@example.com"`
- **Exclude test:** `NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test")`

### Step 3: Deduplicate Results

The query may return duplicate rows per engagement (due to multiple architect entries). Deduplicate by `(msevtmgt_readableeventid, msevtmgt_eventstartdate)` — keep the first occurrence.

### Step 4: Present Results

Display as a markdown table sorted by date descending:

```
| Date | Event ID | Engagement | Customer | Type | Status | Hub |
|------|----------|------------|----------|------|--------|-----|
```

- Format dates as `MMM D` (e.g., "Apr 2") for readability
- Show customer as "—" if null
- Include total count: "*N engagements in the last X days*"

### Step 5 (Optional): Save to File

If the user asks to save, write to:

```
outputs/Hub-Lead-Engagements-{start_date}_to_{end_date}.md
```

## Examples

**User:** "What engagements did I lead?"
→ Run with default 30-day range, present table.

**User:** "My hub engagements for FY26 Q3"
→ Start: 2026-01-01, End: 2026-03-31, run query, present table.

**User:** "List my engagements last 90 days"
→ Calculate start date as today minus 90 days, run query, present table.

## Rules

- ALWAYS use semantic model ID `[SEMANTIC_MODEL_ID]`
- ALWAYS filter to `EDLPSource = "CEHUB"` — this is CEHub data only
- NEVER filter on `statuscodename = "Complete"` — include ALL statuses
- ALWAYS filter `IsLeadTA = "Yes"` and `EmailAlias = "user@example.com"`
- ALWAYS deduplicate by Event ID + Date before presenting
- ALWAYS sort by date descending (most recent first)
- NEVER fabricate engagement data — only use actual query results
- Default to 30 days if no date range specified
