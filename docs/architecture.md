# Architecture

This document describes the technical architecture of **AgencyReport AI** — how data flows through the system, why each tool was chosen, and how the pieces fit together.

---

## System Diagram

```
┌─────────────────┐
│  Google Sheets   │   Simulated campaign data source
│ (campaign_data)  │   (Meta Ads / Google Ads style rows)
└────────┬─────────┘
         │
         ▼
┌─────────────────────────────┐
│            n8n               │   Orchestration engine
│  (self-hosted, free)         │
└────────┬─────────────────────┘
         │
         ▼
┌─────────────────┐
│  Read Campaign   │   Google Sheets node (OAuth2)
│      Data        │
└────────┬─────────┘
         ▼
┌─────────────────┐
│  Validate Data   │   Filter node — drops rows with missing/invalid fields
└────────┬─────────┘
         ▼
┌──────────────────────┐
│ Group by Client &     │   Code node — groups by client_name + platform,
│ Period                │   splits into current vs. previous reporting period
└────────┬──────────────┘
         ▼
┌──────────────────────────┐
│ Calculate Metrics &       │   Code node — CTR, Conv. Rate, CPC, CPA, ROAS
│ Comparison                │   + % change vs. previous period (safe division)
└────────┬──────────────────┘
         ▼
┌──────────────────────┐
│ Prepare AI Input      │   Code node — builds a structured, data-grounded prompt
└────────┬──────────────┘
         ▼
┌──────────────────────────┐
│ Generate AI Insights       │   HTTP Request → Google Gemini API
│ (gemini-3.6-flash)         │   Auth: Query Auth credential (API key never in URL)
│                             │   Retry on fail: 3x, 2s backoff
└────────┬────────────────────┘
         ▼
┌──────────────────────┐
│ Parse AI Response     │   Code node — extracts and JSON-parses the model's
│                       │   structured output (summary, wins, issues, recs)
└────────┬──────────────┘
         ▼
┌──────────────────────┐
│ Generate HTML Report  │   Code node — builds a responsive, branded HTML report
└────────┬──────────────┘
         │
   ┌─────┴─────┐
   ▼           ▼
┌───────────┐ ┌───────────────────┐
│ Convert   │ │ Store Report in    │   Supabase insert → `reports` table
│ Report to │ │ Supabase           │
│ File      │ └────────┬───────────┘
│ (local    │          ▼
│ preview)  │ ┌───────────────────┐
└───────────┘ │ Send Report Email  │   Gmail node — HTML email body
              └───────────────────┘
```

---

## Design Decisions

### Why n8n?
n8n is free and self-hostable with no execution limits, unlike most cloud automation platforms that cap free-tier runs. Its visual workflow model also makes the data pipeline easy to inspect and debug at every step — each node's output can be checked independently.

### Why Google Sheets as the data source (instead of real ad platform APIs)?
Real Meta Ads and Google Ads API access requires business verification, app review, and in some cases paid ad spend to test against. Using Google Sheets as a simulated data source keeps the project 100% free, fully controllable, and fast to demo — while preserving the exact same data shape a real integration would produce. Swapping Sheets for a real API later only requires replacing the `Read Campaign Data` node; every downstream node is unaffected.

### Why Google Gemini for AI insights?
Gemini offers a genuinely free tier with no credit card requirement, unlike OpenAI or Anthropic, whose APIs require paid credits. This made it the only viable option for a fully-free portfolio build.

### Why two separate Code nodes for grouping and metrics (Steps 4 and 5)?
Splitting "group and split periods" from "calculate metrics" keeps each function focused and independently testable. This also made debugging significantly easier during development — each node's output could be verified in isolation before adding the next layer of logic.

### Why validate data *before* grouping?
Filtering invalid rows first prevents bad data (missing fields, non-numeric values) from propagating into the grouping and metric-calculation logic, where it could cause silent errors or crashes (e.g., division by zero, `undefined` fields).

### Why Supabase for storage?
Supabase provides a free-tier hosted Postgres database with a simple REST-like interface, requiring no server management — a good fit for a portfolio project that still needs to demonstrate real persistent storage rather than in-memory-only processing.

### Why send the report as an HTML email body rather than a PDF or attachment?
An inline HTML email requires zero extra steps for the client to view the report — no downloading or opening attachments. It also better demonstrates responsive HTML/CSS skills. PDF export is noted as a Version 2 enhancement.

---

## Error Handling Strategy

| Failure point | Mitigation |
|---|---|
| Missing/invalid row data | `Validate Data` (Filter node) removes bad rows before they reach calculation logic |
| Division by zero (e.g., 0 clicks) | `safeDiv()` helper function returns `0` instead of `Infinity`/`NaN` |
| AI response is not valid JSON | `Parse AI Response` wraps `JSON.parse()` in a try/catch and returns a fallback object with a `parse_error` field instead of crashing |
| Gemini API temporary overload / rate limit | `Retry On Fail` (3 attempts, 2s delay) on the `Generate AI Insights` node |
| One item fails even after retries | `Continue On Fail` enabled, so the rest of the batch still completes |
| Any workflow-level failure | A dedicated `AgencyReport AI - Error Handler` workflow (triggered via n8n's Error Trigger) sends an email alert with the workflow name, error message, and timestamp |

---

## Credential Security

All API keys and OAuth tokens are stored in n8n's built-in credential system, never hardcoded into node parameters or URLs. This means the exported workflow JSON (committed to this repository) contains **no secrets** — only references to credential names, which must be reconnected by whoever imports the workflow into their own n8n instance.