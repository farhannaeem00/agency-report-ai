# AgencyReport AI

**AI-powered marketing client reporting automation** — built with n8n, Google Sheets, Google Gemini, Supabase, and Gmail.

AgencyReport AI automatically collects marketing campaign data, calculates key performance metrics, compares current vs. previous reporting periods, generates AI-written performance insights, and delivers a polished, branded HTML report directly to a client's inbox — with zero manual work.

This project was built as a portfolio piece to demonstrate practical AI + workflow automation engineering: designing a real data pipeline, integrating an LLM safely (grounded strictly in real data, no hallucinated numbers), handling errors gracefully, and shipping a production-style deliverable — all on a 100% free tech stack.

---

## ✨ Features

- 📊 **Automated data pipeline** — reads campaign data from Google Sheets, validates it, and groups it by client + platform
- 📈 **Metric calculations** — CTR, Conversion Rate, CPC, CPA, and ROAS, with safe handling of division-by-zero edge cases
- 🔁 **Period-over-period comparison** — automatically compares the current reporting period against the previous one
- 🤖 **AI-generated insights** — Google Gemini analyzes the *real* calculated metrics (never invents numbers) to produce a performance summary, key wins, areas for attention, and recommendations
- 🎨 **Polished HTML report** — a responsive, branded report with color-coded metric cards and AI commentary
- 🗄️ **Persistent storage** — every generated report is saved to a Supabase database for historical record-keeping
- 📧 **Automated delivery** — the finished report is emailed directly to the client via Gmail, rendered inline
- 🛡️ **Error handling** — retry logic on AI API calls, `Continue On Fail` isolation so one bad record doesn't crash a run, and a dedicated error-notification workflow
- ⏰ **Dual triggers** — supports both manual execution (for testing) and a weekly scheduled run (for production use)

---

## 🧱 Tech Stack

| Layer | Tool | Why |
|---|---|---|
| Automation / Orchestration | [n8n](https://n8n.io) (self-hosted) | Free, unlimited, visual workflow builder |
| Data Source | Google Sheets | Free, simple to simulate real campaign data |
| AI / Insights | Google Gemini (`gemini-3.6-flash`) via Google AI Studio | Generous free tier, no credit card required |
| Database | [Supabase](https://supabase.com) | Free-tier Postgres database with a simple REST/JS interface |
| Report Format | HTML (responsive, inline CSS) | Renders cleanly in browsers and email clients alike |
| Delivery | Gmail (OAuth2) | Free, reliable, familiar to any client |

**100% free to build and run** — no paid APIs, no paid hosting, no subscriptions.

---

## 🏗️ Architecture

```
Google Sheets (campaign data)
        ↓
   n8n Trigger (Manual or Weekly Schedule)
        ↓
   Read Campaign Data
        ↓
   Validate Data              → discards rows with missing/invalid fields
        ↓
   Group by Client & Period   → groups rows by client + platform, splits current vs. previous period
        ↓
   Calculate Metrics & Comparison → CTR, Conv. Rate, CPC, CPA, ROAS + % change vs. previous period
        ↓
   Prepare AI Input           → formats a structured, data-grounded prompt
        ↓
   Generate AI Insights       → Google Gemini API (retry-on-fail enabled)
        ↓
   Parse AI Response          → extracts structured JSON (summary, wins, issues, recommendations)
        ↓
   Generate HTML Report       → builds a responsive, branded report
        ↓
   ┌───────────────┴───────────────┐
   ↓                               ↓
Convert Report to File      Store Report in Supabase
(for local preview)                ↓
                             Send Report Email (Gmail, HTML body)
```

**Error handling:** Any node failure in the AI step retries automatically (3x). If an item still fails, `Continue On Fail` ensures the rest of the batch still completes. A separate `AgencyReport AI - Error Handler` workflow listens for any workflow-level failure and sends an email alert.

---

## 📂 Repository Structure

```
agency-report-ai/
│
├── README.md
│
├── n8n-workflows/
│   ├── agency-report-ai.json          # main automation workflow
│   └── agency-report-ai-error-handler.json
│
├── sample-data/
│   └── campaign-data.csv              # sample simulated campaign data
│
└── docs/
    ├── architecture.md
    ├── workflow.md
    └── screenshots/
        ├── 01-google-sheet-data.png
        ├── 02-n8n-workflow-overview.png
        ├── 03-metric-calculations.png
        ├── 04-ai-output.png
        ├── 05-generated-report.png
        ├── 06-supabase-reports-table.png
        └── 07-email-delivery.png
```

---

## 🚀 How It Works — Step by Step

1. **Data source:** A Google Sheet simulates campaign data pulled from ad platforms (Meta Ads, Google Ads), with fields for client, platform, campaign, reporting period, spend, impressions, clicks, conversions, and revenue.
2. **Validation:** Rows missing required fields or containing invalid numeric data are filtered out before processing, preventing crashes downstream.
3. **Grouping & comparison:** Rows are grouped by client + platform, then split into "current" and "previous" reporting periods for trend comparison.
4. **Metric calculation:** CTR, Conversion Rate, CPC, CPA, and ROAS are calculated for both periods, along with percentage change — all with safe division-by-zero handling.
5. **AI insight generation:** A structured prompt (containing *only* real calculated data) is sent to Google Gemini, which is explicitly instructed not to invent numbers or results. The AI returns a JSON object with a performance summary, key wins, areas for attention, and recommendations.
6. **Report generation:** All of the above is assembled into a polished, responsive HTML report with a branded header, metric cards with color-coded trend arrows, and the AI-generated commentary.
7. **Storage:** Every generated report is inserted into a Supabase `reports` table for historical tracking.
8. **Delivery:** The report is emailed directly to the client as an HTML email body via Gmail.

---

## 🔒 Sample Data

This project uses **simulated data** in a Google Sheet — it does not connect to any real Meta Ads or Google Ads accounts. This was an intentional scope decision (see "What Was Not Built" below) to keep the project free, simple, and demo-friendly while still proving the full automation and AI pipeline end-to-end.

Sample fields:

| Field | Description |
|---|---|
| `client_name` | Client/company name |
| `platform` | Meta Ads / Google Ads |
| `campaign_name` | Campaign name |
| `reporting_period` | Reporting period (`YYYY-MM`) |
| `spend` | Advertising spend |
| `impressions` | Number of impressions |
| `clicks` | Number of clicks |
| `conversions` | Number of conversions |
| `revenue` | Revenue generated |

---

## ⚙️ Setup / Running Locally

1. **n8n:** Install locally with `npx n8n`, or via Docker.
2. **Google Sheets:** Create a Google Cloud project, enable the Sheets + Drive APIs, and create OAuth2 credentials for the n8n Google Sheets node.
3. **Google Gemini:** Get a free API key from [Google AI Studio](https://aistudio.google.com).
4. **Supabase:** Create a free project and run the SQL below to create the `reports` table.
5. **Gmail:** Enable the Gmail API in the same Google Cloud project and reuse/create OAuth2 credentials for the n8n Gmail node.
6. Import `n8n-workflows/agency-report-ai.json` into your n8n instance and connect your credentials to each node.

```sql
create table reports (
  id uuid primary key default gen_random_uuid(),
  client_name text,
  platform text,
  campaign_name text,
  reporting_period text,
  report_content text,
  created_at timestamp with time zone default now()
);
```

---

## 🧭 What Was Not Built (Out of Scope for v1)

Per the original project scope, the following were intentionally excluded from this version to keep the project focused and free to run:

- Real Meta Ads / Google Ads API integrations
- A full analytics platform or custom frontend dashboard
- PDF report export
- Multi-client batch processing
- Slack notifications
- Complex authentication / multi-agent systems

These are documented as potential **Version 2** enhancements.

---

## 📈 Possible Version 2 Enhancements

- Real Meta Ads / Google Ads API integration
- PDF export in addition to HTML
- A simple dashboard to browse report history from Supabase
- Multi-client batch report generation in a single run
- Slack notification when a new report is generated

---

## 🖼️ Screenshots

*(See `docs/screenshots/` for the full set)*

- Google Sheet with sample campaign data
- Full n8n workflow canvas
- Calculated metrics output
- AI-generated insights (raw JSON)
- Final polished HTML report
- Supabase `reports` table with stored history
- Delivered email in Gmail inbox

---

## 👤 Author

Built by **Farhan Shah** as a full-stack AI engineering portfolio project, demonstrating end-to-end workflow automation, safe LLM integration, and production-style error handling — all on a free tech stack.