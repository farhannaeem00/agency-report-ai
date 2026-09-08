# AgencyReport AI

> **AI-powered marketing reporting automation for agencies**

AgencyReport AI automates the process of turning marketing campaign data into client-ready performance reports.

Built with **n8n, Google Sheets, Google Gemini, Supabase, and Gmail**, the system reads campaign data, validates it, calculates key marketing metrics, compares reporting periods, generates AI-powered performance insights, creates a polished HTML report, stores the report history, and delivers the final report directly by email.

The project demonstrates practical **AI + workflow automation engineering**, including data pipelines, business metric calculations, safe LLM integration, structured AI outputs, error handling, database storage, and automated report delivery — using a free tech stack.

---

# 🎯 The Problem

Marketing agencies often spend significant time preparing recurring client reports.

A typical reporting process can involve:

* Collecting campaign performance data
* Checking and validating the data
* Calculating marketing metrics
* Comparing current and previous reporting periods
* Writing performance summaries
* Identifying wins and areas for improvement
* Creating client-ready reports
* Sending reports manually

These repetitive tasks can consume valuable time that could otherwise be spent on campaign strategy and client growth.

---

# 💡 The Solution

AgencyReport AI automates the reporting workflow from campaign data to client delivery.

```text
Campaign Data
      ↓
Validate Data
      ↓
Calculate Metrics
      ↓
Compare Reporting Periods
      ↓
Generate AI Insights
      ↓
Create HTML Report
      ↓
Store Report History
      ↓
Email Client
```

The AI is provided with **real calculated metrics only** and is instructed to generate insights based on the supplied data rather than inventing performance numbers.

---

# ✨ Key Features

* 📊 **Automated data pipeline** — reads campaign data from Google Sheets, validates records, and prepares data for reporting
* 📈 **Marketing metric calculations** — calculates CTR, Conversion Rate, CPC, CPA, and ROAS with safe handling for division-by-zero edge cases
* 🔁 **Period-over-period comparison** — compares current and previous reporting periods to identify performance trends
* 🤖 **AI-generated insights** — uses Google Gemini to generate a performance summary, key wins, areas for attention, and recommendations based on real calculated metrics
* 🛡️ **Data-grounded AI** — AI prompts contain structured performance data and explicitly instruct the model not to invent numbers or results
* 🎨 **HTML report generation** — creates a responsive, branded report with metric cards, performance trends, and AI-generated commentary
* 🗄️ **Persistent report storage** — saves generated reports to Supabase for historical record-keeping
* 📧 **Automated report delivery** — sends the finished HTML report directly to the client through Gmail
* 🛡️ **Error handling** — includes retry logic for AI calls, `Continue On Fail` isolation, and a dedicated error-notification workflow
* ⏰ **Flexible execution** — supports manual execution for testing and scheduled execution for recurring reporting

---

# 🧱 Tech Stack

| Layer                      | Technology        | Purpose                                        |
| -------------------------- | ----------------- | ---------------------------------------------- |
| Automation & Orchestration | n8n (Self-hosted) | Manages the complete workflow and integrations |
| Campaign Data              | Google Sheets     | Provides simulated marketing campaign data     |
| AI Insights                | Google Gemini     | Generates data-grounded performance insights   |
| Database                   | Supabase          | Stores generated reports and reporting history |
| Report Format              | HTML + Inline CSS | Creates responsive client-ready reports        |
| Email Delivery             | Gmail             | Delivers reports directly to clients           |

## 💰 Cost

The project is designed around free-tier and self-hosted tools.

> **No paid infrastructure is required to build and demonstrate the project.**

Actual production usage may depend on the free-tier limits and policies of individual services.

---

# 🏗️ High-Level Architecture

```text
Google Sheets
Campaign Data
        │
        ▼
┌─────────────────────┐
│     n8n Trigger     │
│ Manual / Scheduled  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Read Campaign Data  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Validate Data     │
│ Remove Invalid Rows │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────┐
│ Prepare Reporting Data      │
│ Current vs Previous Period  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Calculate Metrics           │
│ CTR • CVR • CPC • CPA • ROAS│
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Compare Performance         │
│ Period-over-Period Changes  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Prepare AI Input            │
│ Real Calculated Data Only   │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Google Gemini              │
│ Generate Performance Insight│
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Generate HTML Report        │
└──────────────┬──────────────┘
               │
        ┌──────┴──────┐
        ▼             ▼
┌──────────────┐ ┌──────────────┐
│ Supabase     │ │ Gmail        │
│ Store Report │ │ Send Report  │
└──────────────┘ └──────────────┘
```

For a more detailed explanation, see:

* `docs/architecture.md`
* `docs/workflow.md`

---

# 🔄 How It Works

## Step 1 — Read Campaign Data

The workflow reads simulated marketing campaign data from Google Sheets.

The data represents campaign performance from platforms such as:

* Meta Ads
* Google Ads

Each record can include:

* Client name
* Platform
* Campaign name
* Reporting period
* Spend
* Impressions
* Clicks
* Conversions
* Revenue

---

## Step 2 — Validate Data

Before calculations begin, the workflow validates campaign records.

Rows containing missing required fields or invalid numeric values are filtered out to prevent incorrect calculations and downstream workflow failures.

---

## Step 3 — Prepare Reporting Data

The workflow prepares campaign data for reporting and comparison.

Data is organized according to the reporting context, including the current and previous reporting periods, allowing performance trends to be calculated.

---

## Step 4 — Calculate Marketing Metrics

The workflow calculates key performance metrics.

### CTR — Click-Through Rate

```text
CTR = (Clicks / Impressions) × 100
```

### Conversion Rate

```text
Conversion Rate = (Conversions / Clicks) × 100
```

### CPC — Cost Per Click

```text
CPC = Spend / Clicks
```

### CPA — Cost Per Acquisition

```text
CPA = Spend / Conversions
```

### ROAS — Return on Ad Spend

```text
ROAS = Revenue / Spend
```

The workflow includes safe handling for division-by-zero scenarios.

---

## Step 5 — Compare Reporting Periods

The calculated metrics are compared with the previous reporting period.

This helps identify whether performance has:

* Improved
* Declined
* Remained stable

Percentage changes are calculated where applicable.

---

## Step 6 — Generate AI Insights

The calculated performance data is converted into a structured prompt and sent to Google Gemini.

The AI receives **real calculated metrics only**.

The prompt instructs the model to generate insights without inventing campaign numbers or unsupported performance claims.

The AI produces structured insights such as:

* Performance summary
* Key wins
* Areas for attention
* Recommendations

---

# 🤖 Safe AI Integration

AgencyReport AI is designed to reduce unreliable AI output.

The workflow follows this approach:

```text
Real Campaign Data
        ↓
Validated Data
        ↓
Calculated Metrics
        ↓
Structured AI Prompt
        ↓
Google Gemini
        ↓
Structured Insights
```

The AI is used for **interpretation and recommendations**, while metric calculations are performed directly by the workflow.

This means:

```text
n8n / Workflow Logic
        ↓
Calculates Numbers
        ↓
Google Gemini
        ↓
Explains the Numbers
```

The AI is not responsible for calculating or inventing performance metrics.

---

## Step 7 — Parse AI Response

The workflow processes the AI response and extracts the structured output.

The generated insight data includes:

* Summary
* Key wins
* Areas for attention
* Recommendations

This structured data is then combined with the calculated metrics.

---

## Step 8 — Generate HTML Report

The workflow creates a client-ready HTML report.

The report includes:

* Branded report header
* Campaign information
* Key performance metrics
* Performance comparisons
* Trend indicators
* AI-generated performance summary
* Key wins
* Areas for attention
* Recommendations

The report uses responsive HTML and inline CSS so it can be displayed directly in email clients.

---

## Step 9 — Store Report History

Each generated report is stored in Supabase.

The database provides historical record-keeping for generated reports.

---

## Step 10 — Deliver the Report

The completed report is sent directly to the client using Gmail.

```text
HTML Report Generated
        ↓
Report Stored in Supabase
        ↓
Email Prepared
        ↓
Client Receives Report
```

---

# 📊 Example Report Output

A generated report can include information such as:

```text
Marketing Performance Report

CTR
3.8%
↑ Improved compared to the previous period

ROAS
4.2
↑ Higher return on advertising spend

CPA
$14.50
↓ Lower acquisition cost

AI Performance Summary

Campaign performance improved during the current
reporting period. Increased engagement and improved
return on ad spend indicate stronger campaign efficiency.

Key Wins

• ROAS improved compared to the previous period
• Conversion performance increased

Areas for Attention

• Monitor campaigns with declining CTR

Recommendations

• Review high-performing campaigns for scaling
• Optimize lower-performing campaigns
```

> The values above are example values used only to demonstrate the report format.

---

# 🛡️ Error Handling

AgencyReport AI includes multiple reliability measures.

## AI Retry Logic

If the AI request fails temporarily, the workflow retries the request before continuing.

```text
AI Request
    ↓
Failure
    ↓
Retry
    ↓
Retry
    ↓
Retry
```

---

## Continue On Fail

If one item encounters an issue, the workflow can continue processing instead of immediately crashing the entire run.

This helps isolate problematic records.

---

## Dedicated Error Workflow

A separate workflow handles workflow-level failures.

```text
Main Workflow Error
        ↓
Error Handler Workflow
        ↓
Send Error Notification
        ↓
Administrator
```

This ensures failures are surfaced instead of remaining silent.

---

# 📂 Repository Structure

```text
agency-report-ai/
│
├── README.md
│
├── n8n-workflows/
│   ├── agency-report-ai.json
│   └── agency-report-ai-error-handler.json
│
├── sample-data/
│   └── campaign-data.csv
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

# 🔒 Sample Data

This project uses **simulated campaign data**.

It does not connect directly to live Meta Ads or Google Ads accounts.

This was an intentional scope decision for Version 1 to keep the project:

* Free to build and demonstrate
* Easy to test
* Simple to understand
* Focused on the automation and AI reporting pipeline

The project still demonstrates the complete workflow from campaign data processing to AI insights and automated report delivery.

## Sample Data Fields

| Field              | Description                          |
| ------------------ | ------------------------------------ |
| `client_name`      | Client or company name               |
| `platform`         | Advertising platform                 |
| `campaign_name`    | Campaign name                        |
| `reporting_period` | Reporting period in `YYYY-MM` format |
| `spend`            | Advertising spend                    |
| `impressions`      | Number of impressions                |
| `clicks`           | Number of clicks                     |
| `conversions`      | Number of conversions                |
| `revenue`          | Revenue generated                    |

---

# ⚙️ Setup and Running Locally

## 1. Set Up n8n

Install n8n locally:

```bash
npx n8n
```

You can also run n8n using Docker.

---

## 2. Configure Google Sheets

Create a Google Cloud project and configure OAuth2 credentials for Google Sheets.

Enable:

* Google Sheets API
* Google Drive API

Connect the credentials to the Google Sheets node in n8n.

---

## 3. Configure Google Gemini

Create an API key through Google AI Studio and connect it to the AI integration used by the workflow.

---

## 4. Configure Supabase

Create a Supabase project and create the `reports` table.

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

Connect your Supabase credentials to the relevant n8n nodes.

---

## 5. Configure Gmail

Configure Gmail OAuth2 credentials and connect them to the Gmail node.

The Gmail integration is responsible for delivering the generated HTML report.

---

## 6. Import the Workflows

Import the following workflows into n8n:

```text
n8n-workflows/
├── agency-report-ai.json
└── agency-report-ai-error-handler.json
```

After importing:

1. Configure all required credentials
2. Connect the Google Sheet data source
3. Configure Supabase
4. Configure Gmail
5. Configure the AI integration
6. Test the workflow manually

---

# 🧪 Testing

The workflow can be tested manually using the sample campaign data.

Recommended testing process:

```text
Add Sample Campaign Data
        ↓
Run Workflow Manually
        ↓
Verify Validation
        ↓
Check Calculated Metrics
        ↓
Review AI Output
        ↓
Verify HTML Report
        ↓
Check Supabase Storage
        ↓
Confirm Email Delivery
```

Before production use, test:

* Missing required fields
* Invalid numeric values
* Zero impressions
* Zero clicks
* Zero conversions
* AI API failures
* Database failures
* Email delivery issues

---

# 🧭 Current Scope

Version 1 focuses on demonstrating the complete reporting automation pipeline.

The following features are intentionally outside the current scope:

* Live Meta Ads API integration
* Live Google Ads API integration
* Custom frontend dashboard
* PDF report export
* Slack notifications
* Complex authentication
* Multi-agent systems

These features can be added as future enhancements.

---

# 🚀 Possible Version 2 Enhancements

Potential improvements include:

* 🔗 Live Meta Ads API integration
* 🔗 Live Google Ads API integration
* 📄 PDF report export
* 📊 Dashboard for browsing report history
* 🔔 Slack notifications
* 📈 Expanded analytics and visualizations
* 🏢 More advanced client and reporting management
* 📦 Expanded batch-processing capabilities

---

# 🖼️ Screenshots

Screenshots are available in:

```text
docs/screenshots/
```

The project includes screenshots of:

* Google Sheet campaign data
* Complete n8n workflow
* Metric calculations
* AI-generated insights
* Generated HTML report
* Supabase report storage
* Gmail report delivery

---

# 🎯 Business Value

AgencyReport AI demonstrates how marketing agencies can reduce repetitive reporting work through automation.

Instead of manually:

```text
Collect Data
↓
Calculate Metrics
↓
Compare Results
↓
Write Insights
↓
Create Report
↓
Send Email
```

The workflow automates the reporting pipeline:

```text
Campaign Data
↓
Automated Processing
↓
Metric Calculation
↓
AI Analysis
↓
HTML Report
↓
Database Storage
↓
Client Delivery
```

This approach can help agencies spend less time on repetitive reporting tasks and more time on strategy, optimization, and client growth.

---

# 👤 Author

**Farhan Naeem**

AI & Automation Developer focused on building:

* AI-powered workflows
* Business automation systems
* LLM-powered applications
* Data pipelines
* Practical AI solutions

This project demonstrates end-to-end automation engineering, data processing, safe LLM integration, report generation, database storage, and production-style error handling.

---

## 📄 License

This project is built as a portfolio and learning project.

Feel free to explore the repository and use it as a reference for learning workflow automation and AI-powered reporting systems.
