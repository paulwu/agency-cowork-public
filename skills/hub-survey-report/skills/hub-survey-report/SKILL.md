---
name: hub-survey-report
description: Use this skill when the user asks for "Hub Report survey data", "Hub Context Survey SE", "engagement survey report", "pull survey data from Power BI", "NSAT report", "hub survey markdown", or wants to extract Innovation Hub engagement survey data from the Power BI report and save it as a markdown file.
---

# Hub Survey Report — Power BI Data Extraction

Extract survey data from the **Hub Report > Hub Context > Survey - SE** Power BI page and save as structured markdown.

## Overview

This skill queries the **MTC Survey - TA View** Power BI semantic model to replicate all 9 data visuals from the "MTC Engagement - Customer" report page. Data is formatted into a structured markdown document and saved to `outputs/`.

## Data Source

| Property | Value |
|----------|-------|
| Semantic Model ID | `[SEMANTIC_MODEL_ID]` |
| Semantic Model Name | MTC Survey - TA View |
| Report ID | `[REPORT_ID]` |
| Report Page | MTC Engagement - Customer |
| Workspace | Innovation Hub Report UAT |

## Workflow

### Step 1: Gather Parameters

Ask the user for:

1. **Date range** — Default: fiscal year to date (July 1 of the current FY through today). Accept custom start/end dates.
2. **Scope** — Default: "My engagements" (current user's data as scoped by RLS). No additional SE filter needed unless the user explicitly wants to filter to a specific architect.

Format the start date as `YYYY-MM-DDTHH:MM:SS` for use in DAX filters.

### Step 2: Execute DAX Queries

Use the `powerbi-remote-ExecuteQuery` tool with `artifactId: "[SEMANTIC_MODEL_ID]"`.

All queries MUST include these **base filters** (matching report-level and page-level filters):

```
'DimEvent'[statuscodename] = "Complete"
'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM"
'DimEvent'[msevtmgt_eventstartdate] >= DATE(start_year, start_month, start_day)
NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test")
```

Execute queries in 3 batched API calls using `daxQueries` arrays (max 4 per call).

---

#### API Call 1 — Engagement List + 3 Satisfaction Charts

**Query 1: Engagement Survey Responses List**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'FactSurvey'[EventStartDate],
        'FactSurvey'[EventID],
        'DimEvent'[msevtmgt_name],
        'DimEvent'[CustomerName],
        'DimEvent'[msevtmgt_eventtypename],
        'DimEvent'[cet_leadtecharchname],
        'DimEvent'[msevtmgt_buildingname]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey-old"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey",
    NOT(
        'FactSurvey'[QuestionName] = "Is there anything we could have done to improve the value of your experience?"
        || 'FactSurvey'[QuestionName] = "Name"
        || 'FactSurvey'[QuestionName] = "What did you find most beneficial about this engagement?"
    )
)
ORDER BY 'FactSurvey'[EventStartDate] DESC
```

Use `maxRows: 500`.

**Query 2: Overall Satisfaction Distribution**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'FactSurvey'[QuestionResponse],
        "ResponseCount", [Response Count]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey-old",
    'DimQuestion'[QuestionName] = "Overall, how satisfied are you with this engagement?"
        || 'DimQuestion'[QuestionName] = "Overall, how satisfied were you with this engagement?"
)
```

**Query 3: Increased Understanding of Microsoft Solutions**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'FactSurvey'[QuestionResponse],
        "ResponseCount", [Response Count]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement",
    'DimQuestion'[QuestionName] = "This engagement increased my understanding of Microsoft solutions relevant to my organization."
)
```

**Query 4: Helped Identifying Actionable Next Steps**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'FactSurvey'[QuestionResponse],
        "ResponseCount", [Response Count]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey-old",
    'DimQuestion'[QuestionName] = "This engagement identified actionable next steps to enable my organization's technical and business goals."
        || 'DimQuestion'[QuestionName] = "This engagement identified actionable next steps to enable my organization's technical and business goals"
)
```

---

#### API Call 2 — Met Objectives + Avg Scores + Free Text

**Query 5: Met Defined Objectives**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'FactSurvey'[QuestionResponse],
        "ResponseCount", [Response Count]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey-old",
    'DimQuestion'[QuestionName] = "This engagement met the defined objectives."
)
```

**Query 6: Average Response Score by Question**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'DimQuestion'[QuestionName],
        "AvgScore", [Avg Score]
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey-old"
        || 'DimQuestion'[SurveyName] = "Microsoft Technology Center Survey"
)
```

**Query 7: What Did You Find Most Beneficial?**

> **Important:** Free-text queries MUST filter on `'FactSurvey'[QuestionName]` (not `'DimQuestion'[QuestionName]`) and use `VALUES()` (not `SUMMARIZECOLUMNS`). The DimQuestion cross-table filter does not propagate correctly for free-text responses.

```dax
EVALUATE
CALCULATETABLE(
    VALUES('FactSurvey'[QuestionResponse]),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'FactSurvey'[QuestionName] = "What did you find most beneficial about this engagement?"
)
```

Use `maxRows: 500`.

**Query 8: Could We Have Improved Your Experience?**

> **Important:** Same pattern as Query 7 — filter on `'FactSurvey'[QuestionName]` directly.

```dax
EVALUATE
CALCULATETABLE(
    VALUES('FactSurvey'[QuestionResponse]),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'FactSurvey'[QuestionName] = "Is there anything we could have done to improve the value of your experience?"
)
```

Use `maxRows: 500`.

---

#### API Call 3 — Survey Response Details

**Query 9: Survey Response Details by Engagement**

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'DimEvent'[msevtmgt_eventstartdate],
        'DimEvent'[msevtmgt_readableeventid],
        'EngagementArchitect'[FullName],
        'FactSurvey'[activityId],
        'DimQuestion'[QuestionName],
        "Response", MIN('FactSurvey'[QuestionResponse])
    ),
    'DimEvent'[statuscodename] = "Complete",
    'DimEvent'[EDLPSource] = "CEHUB" || 'DimEvent'[EDLPSource] = "MTCCRM",
    'DimEvent'[msevtmgt_eventstartdate] >= DATE({start_year}, {start_month}, {start_day}),
    NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test"),
    'DimQuestion'[SurveyName] = "Attendee MTC Engagement"
)
ORDER BY 'DimEvent'[msevtmgt_eventstartdate] DESC
```

Use `maxRows: 1000`.

---

### Step 3: Format as Markdown

Structure the markdown output with these sections, matching the report page visuals:

```markdown
# Hub Report — Hub Context > Survey - SE

**Data Source:** MTC Survey - TA View (Power BI)
**Date Range:** {start_date} to {end_date}
**Extracted:** {today's date and time}
**Report:** Hub Report > Hub Context > Survey - SE (Report ID: aa1f01e7, App ID: 171bd9de)

---

## Summary Scores

| Question | Avg Score |
|----------|-----------|
| {from Query 6 results} |

---

## Overall Satisfaction

| Response | Count |
|----------|-------|
| {from Query 2 results} |

**Total responses:** {sum of counts}

---

## Increased Understanding of Microsoft Solutions

| Response | Count |
|----------|-------|
| {from Query 3 results} |

---

## Actionable Next Steps Identified

| Response | Count |
|----------|-------|
| {from Query 4 results} |

---

## Met Defined Objectives

| Response | Count |
|----------|-------|
| {from Query 5 results} |

---

## Engagement Survey Responses

| Date | Event ID | Engagement | Customer | Type | Lead TA | Location |
|------|----------|------------|----------|------|---------|----------|
| {from Query 1 results} |

---

## What Did You Find Most Beneficial?

{from Query 7 — render each response as a blockquote}

> "{response 1}"

> "{response 2}"

{...all responses, no truncation}

---

## How Could We Have Improved Your Experience?

{from Query 8 — render each response as a blockquote}

> "{response 1}"

> "{response 2}"

{...all responses, no truncation}

---

## Survey Response Details by Engagement

{from Query 9 — format as a table with engagement info in rows and questions as columns}

| Date | Event ID | Architect | Respondent | Q: Overall Satisfaction | Q: Understanding | Q: Next Steps | Q: Met Objectives |
|------|----------|-----------|------------|------------------------|------------------|---------------|-------------------|
| {pivot query 9 results by activityId → questions} |
```

#### Formatting Rules

- Sort engagement lists by date descending (most recent first)
- Round average scores to 2 decimal places
- Include all free-text responses — do NOT truncate
- For the details pivot (Query 9), pivot from flat rows into a table where each respondent (activityId) is one row and questions are columns
- Use the response text (not numeric score) in the pivot table
- If any query returns no data, include the section header with "No data available for the selected date range."

### Step 4: Save the File

Save to:

```
outputs/Hub_Report-Hub_Context-Survey-SE.md
```

If the user specified a custom date range or scope, append context:

```
outputs/Hub_Report-Hub_Context-Survey-SE-{YYYY-MM-DD}_to_{YYYY-MM-DD}.md
```

### Step 5: Confirm

After saving, tell the user:
- The file path
- Total engagement count and response count from the data
- Overall satisfaction score (from Avg Score query)
- Offer to analyze trends or specific engagements if needed

## Validation

Before formatting, verify:
1. The semantic model name in the response matches "MTC Survey - TA View"
2. All 9 queries returned (even if some return empty results)
3. No DAX execution errors occurred

If a query fails, retry once. If it fails again, include the section with an error note and continue with remaining queries.

## Composes With

- **meeting-summary** — Cross-reference engagement survey scores with meeting outcomes
- **hub-close-out-email** — Include survey highlights in close-out communications
- **visual-explainer** — Generate visual charts from the extracted survey data

## Rules

- ALWAYS use the exact semantic model ID `[SEMANTIC_MODEL_ID]`
- ALWAYS apply all base filters (Complete status, CEHUB/MTCCRM source, date range, not test)
- ALWAYS apply visual-specific filters per query as documented above
- ALWAYS include all free-text responses without truncation
- NEVER fabricate or estimate data — only use actual query results
- NEVER modify the DAX query filters unless the user explicitly requests a different scope
- ALWAYS save output to the `outputs/` folder
