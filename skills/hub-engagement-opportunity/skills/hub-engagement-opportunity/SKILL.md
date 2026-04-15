---
name: hub-engagement-opportunity
description: Use this skill when the user asks for "engagements and opportunities", "hub engagement opportunities", "MSX opportunities for my engagements", "what opportunities are linked to my sessions", "engagement pipeline", or wants to see Innovation Hub engagements with their associated MSX opportunities. Builds on hub-lead-engagement by cross-referencing each engagement's customer account with MSX opportunities.
---

# Hub Engagement Opportunity — Engagements × MSX Opportunities

List CEHub engagements where the current user was Lead TA, then cross-reference each engagement's customer account with open MSX opportunities. Default date range: last 30 days.

## Data Sources

| Source | Purpose |
|--------|---------|
| Power BI: `[SEMANTIC_MODEL_ID]` (MTC Survey - TA View) | CEHub engagements, customer names, TPIDs |
| MSX Dataverse (via `msx-mcp` tools) | Opportunities by account |

## Constants

```
MSX_APP_ID = "[MSX_APP_ID]"
MSX_BASE_URL = "https://[MSX_URL]/main.aspx"
```

MSX Opportunity URL pattern:
```
{MSX_BASE_URL}?appid={MSX_APP_ID}&pagetype=entityrecord&etn=opportunity&id={opportunity_guid}
```

## Workflow

### Step 1: Determine Date Range

Same as `hub-lead-engagement`: ask user or default to **last 30 days**.

### Step 2: Get Lead Engagements from Power BI

Use `powerbi-remote-ExecuteQuery` with `artifactId: "[SEMANTIC_MODEL_ID]"`.

```dax
EVALUATE
CALCULATETABLE(
    SUMMARIZECOLUMNS(
        'DimEvent'[msevtmgt_eventstartdate],
        'DimEvent'[msevtmgt_name],
        'DimEvent'[CustomerName],
        'DimEvent'[TPID],
        'DimEvent'[msevtmgt_eventtypename],
        'DimEvent'[msevtmgt_readableeventid],
        'DimEvent'[statuscodename]
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

#### Filter Rules (same as hub-lead-engagement)

- **Source:** `EDLPSource = "CEHUB"` only
- **Status:** Do NOT filter on status — include ALL (Complete, Closeout, Confirmed, Qualified)
- **Lead TA:** `IsLeadTA = "Yes"` AND `EmailAlias = "user@example.com"`
- **Exclude test:** `NOT CONTAINSSTRING('DimEvent'[msevtmgt_name], "test")`

Deduplicate by `(msevtmgt_readableeventid, msevtmgt_eventstartdate)`.

### Step 3: Collect Unique Customer Accounts

From the engagement results, extract unique customer accounts. Skip engagements with null/empty CustomerName (internal events like FieldDays, Innovation Days, Holiday Blocks, OOO, etc.).

Build a list of `(CustomerName, TPID)` pairs to look up in MSX.

### Step 4: Query MSX Opportunities per Account

For each unique customer account, use the `msx-mcp-get_account_opportunities` tool:

```
msx-mcp-get_account_opportunities(
    account_name: "{CustomerName}",   # Use if no TPID
    tpid: "{TPID}",                   # Use if TPID available (preferred — exact match)
    format: "compact",
    limit: 25,
    status: "open"
)
```

**Prefer TPID** when available — it gives exact account match. Fall back to `account_name` if TPID is null.

**Parallelize:** Make multiple `get_account_opportunities` calls in parallel for different accounts to minimize latency.

### Step 5: Build Combined Output

For each engagement, match its customer to the MSX results. Present TWO tables:

#### Table 1: Engagement Summary

```markdown
| Date | Engagement | Customer | Type | Status |
|------|-----------|----------|------|--------|
```

#### Table 2: MSX Opportunities by Engagement Customer

For each engagement that has a customer, list the associated MSX opportunities:

```markdown
### {Engagement Name} — {Customer Name}

| MSX Topic | MSX Opp # | Est. Value | MSX URL |
|-----------|-----------|-----------|---------|
| {opportunity name} | {MSX ID, e.g. 7-3CFBDME3FQ} | ${value} | [Open in MSX]({url}) |
```

Where the MSX URL is constructed as:
```
https://[MSX_URL]/main.aspx?appid=[MSX_APP_ID]&pagetype=entityrecord&etn=opportunity&id={opportunity_guid}
```

- `{opportunity_guid}` = the Opportunity ID (Dataverse GUID) from the MSX search results
- `{MSX ID}` = the MSX ID field (e.g., `7-3CFBDME3FQ`)

#### Formatting Rules

- Sort engagements by date descending
- Sort opportunities by estimated value descending within each engagement
- For engagements with no customer (internal events), list them in Table 1 but note "Internal — no MSX opportunities"
- If an account has no open opportunities, note "No open MSX opportunities found"
- Include total counts: "N engagements, M unique customers, P total opportunities"

### Step 6 (Optional): Save to File

If the user asks to save, write to:

```
outputs/Hub-Engagement-Opportunities-{start_date}_to_{end_date}.md
```

## Example Output

```markdown
## Engagement Summary

*5 engagements, 3 customers, 12 opportunities (last 30 days)*

| Date | Engagement | Customer | Type | Status |
|------|-----------|----------|------|--------|
| Apr 2 | Port of LA - Fabric Envisioning | Adventure Works | Business Envisioning | Complete |
| Mar 31 | Contoso - Azure Design Session | Contoso | Architecture Design | Closeout |
| Mar 26 | FieldDays @ the Innovation Hub | — | Activation | Complete |
| Mar 19 | Probation Infrastructure DR - ADS | Northwind Traders | Architecture Design | Complete |
| Mar 17 | Infrastructure & Data Strategy | Fabrikam | Solution Envisioning | Complete |

---

### Port of LA - Fabric Envisioning — Adventure Works

| MSX Topic | MSX Opp # | Est. Value | MSX URL |
|-----------|-----------|-----------|---------|
| FY26 City of LA Fabric | 7-XXXXXXX | $500,000 | [Open in MSX](https://[MSX_URL]/main.aspx?appid=[MSX_APP_ID]&pagetype=entityrecord&etn=opportunity&id=guid-here) |

---

### Contoso - Azure Design Session — Contoso

| MSX Topic | MSX Opp # | Est. Value | MSX URL |
|-----------|-----------|-----------|---------|
| FY26 Contoso M365 Copilot | 7-3FGJRFAQCG | $202,500 | [Open in MSX](https://[MSX_URL]/main.aspx?appid=[MSX_APP_ID]&pagetype=entityrecord&etn=opportunity&id=be1236f5-0890-f011-b4cc-7c1e5215a78e) |

---

### FieldDays @ the Innovation Hub — Internal

*Internal event — no MSX opportunities*
```

## Composes With

- **hub-lead-engagement** — Step 2 reuses the same engagement query
- **hub-survey-report** — Cross-reference survey scores with opportunity pipeline
- **hub-close-out-email** — Include opportunity context in close-out communications

## Rules

- ALWAYS use semantic model ID `[SEMANTIC_MODEL_ID]` for engagements
- ALWAYS use `msx-mcp-get_account_opportunities` (or `search_opportunities`) for MSX data
- NEVER filter on `statuscodename = "Complete"` — include ALL engagement statuses
- ALWAYS construct MSX URLs using the pattern: `https://[MSX_URL]/main.aspx?appid=[MSX_APP_ID]&pagetype=entityrecord&etn=opportunity&id={guid}`
- ALWAYS use TPID for account lookup when available (exact match); fall back to account_name
- ALWAYS skip internal events (null CustomerName) for MSX lookup — don't waste API calls
- ALWAYS parallelize MSX account queries for different customers
- NEVER fabricate opportunity data — only use actual MSX query results
- Default to 30 days if no date range specified
