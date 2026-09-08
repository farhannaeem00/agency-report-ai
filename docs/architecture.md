# Architecture

This document describes the technical architecture of **AgencyReport AI**, including how campaign data moves through the system, how each component interacts, and the key design decisions behind the workflow.

AgencyReport AI is designed as a modular automation pipeline that transforms campaign data into calculated marketing metrics, AI-generated performance insights, a client-ready HTML report, and automated email delivery.

---

# Overview

The system follows a simple data-processing pipeline:

```text
Campaign Data
      ↓
Validation
      ↓
Reporting Data Preparation
      ↓
Metric Calculation
      ↓
Period Comparison
      ↓
AI Insight Generation
      ↓
HTML Report Generation
      ↓
Storage + Email Delivery
```

The workflow uses **n8n as the orchestration layer** and integrates Google Sheets, Google Gemini, Supabase, and Gmail.

---

# System Architecture

```text
┌─────────────────────────┐
│     Google Sheets       │
│                         │
│ Simulated Campaign Data │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│          n8n            │
│                         │
│ Automation &            │
│ Orchestration Layer     │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│   Read Campaign Data    │
│                         │
│ Google Sheets Node      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│      Validate Data      │
│                         │
│ Remove Invalid Records  │
└────────────┬────────────┘
             │
             ▼
┌──────────────────────────────┐
│   Prepare Reporting Data     │
│                              │
│ Organize Client / Platform   │
│ and Reporting Period Data    │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│     Calculate Metrics        │
│                              │
│ CTR                          │
│ Conversion Rate              │
│ CPC                          │
│ CPA                          │
│ ROAS                         │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│   Compare Reporting Periods  │
│                              │
│ Current vs Previous          │
│ Performance Comparison       │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│      Prepare AI Input        │
│                              │
│ Structured Prompt            │
│ Real Calculated Metrics      │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│     Google Gemini API        │
│                              │
│ Generate Performance         │
│ Insights                     │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│      Parse AI Response       │
│                              │
│ Summary                      │
│ Key Wins                     │
│ Areas for Attention          │
│ Recommendations              │
└───────────────┬──────────────┘
                │
                ▼
┌──────────────────────────────┐
│    Generate HTML Report      │
│                              │
│ Metrics + Trends +           │
│ AI Performance Insights      │
└───────────────┬──────────────┘
                │
        ┌───────┴────────┐
        ▼                ▼
┌─────────────────┐  ┌─────────────────┐
│    Supabase     │  │      Gmail      │
│                 │  │                 │
│ Store Report    │  │ Send Report     │
│ History         │  │ to Client       │
└─────────────────┘  └─────────────────┘
```

---

# Component Responsibilities

## n8n — Automation and Orchestration Layer

n8n is the central automation engine of the system.

It is responsible for:

* Triggering the workflow
* Reading campaign data
* Validating records
* Preparing reporting data
* Calculating marketing metrics
* Comparing reporting periods
* Preparing structured AI prompts
* Calling the Google Gemini API
* Processing AI responses
* Generating HTML reports
* Storing reports
* Sending reports to clients
* Handling workflow errors

The workflow can support:

* Manual execution for testing
* Scheduled execution for recurring reporting

Using n8n keeps the automation logic visible and easy to inspect because each processing stage is represented by an individual workflow node.

---

## Google Sheets — Campaign Data Layer

Google Sheets is used as the campaign data source.

For Version 1, the project uses simulated campaign data representing advertising platforms such as:

* Meta Ads
* Google Ads

Each record can contain:

* Client name
* Platform
* Campaign name
* Reporting period
* Spend
* Impressions
* Clicks
* Conversions
* Revenue

Google Sheets was intentionally selected for the initial version because it provides a simple and controllable way to demonstrate the complete reporting pipeline without requiring live advertising account access.

The downstream workflow is designed around the campaign data structure, making the data source replaceable in a future version.

---

## Validation Layer

Campaign records are validated before any grouping or calculations occur.

The validation layer checks for:

* Missing required fields
* Invalid numeric values
* Incomplete campaign records

Invalid records are filtered out before entering the metric calculation stage.

```text
Raw Campaign Data
        ↓
Validate Records
        ↓
Valid Records
        ↓
Processing Pipeline
```

This prevents invalid values from propagating into later stages of the workflow.

---

## Reporting Data Preparation Layer

The workflow prepares campaign data for reporting and period comparison.

Data is organized according to relevant reporting information, including:

* Client
* Platform
* Reporting period

The workflow separates current and previous reporting periods so that performance trends can be calculated.

This stage focuses on organizing the data before metric calculations are performed.

---

## Metric Calculation Layer

Marketing metrics are calculated using workflow logic rather than relying on the AI model.

The system calculates:

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

The calculation logic includes safe division handling to prevent:

* `Infinity`
* `NaN`
* Runtime errors caused by zero denominators

The calculated metrics become the trusted data source for later reporting and AI analysis.

---

# Period Comparison Layer

The workflow compares current reporting performance with previous reporting periods.

This allows the system to identify changes in metrics such as:

* CTR
* Conversion Rate
* CPC
* CPA
* ROAS

Where applicable, percentage changes are calculated to show whether performance has:

* Improved
* Declined
* Remained stable

The comparison logic is completed before the AI step so that the AI receives already-calculated performance trends.

---

# AI Insight Layer

Google Gemini is used to interpret calculated campaign performance data and generate written insights.

The AI receives structured information generated by the workflow, including:

* Calculated metrics
* Current performance
* Previous performance
* Performance changes

The AI is responsible for generating:

* Performance summaries
* Key wins
* Areas for attention
* Recommendations

The AI is **not responsible for calculating campaign metrics**.

```text
Campaign Data
      ↓
Workflow Validation
      ↓
Metric Calculations
      ↓
Performance Comparison
      ↓
Structured AI Prompt
      ↓
Google Gemini
      ↓
Written Insights
```

This separation ensures that:

```text
Workflow Logic
      ↓
Calculates Numbers
      ↓
AI
      ↓
Interprets Numbers
```

The AI prompt is designed to use the supplied structured data as the basis for its analysis and to avoid generating unsupported performance figures.

---

# AI Response Processing

The response returned by Google Gemini is processed by a dedicated workflow step.

The system extracts the structured AI output, including:

```text
Performance Summary

Key Wins

Areas for Attention

Recommendations
```

The response is then parsed before being passed to the report-generation layer.

If the AI response cannot be parsed successfully, the workflow uses error handling to prevent an unexpected response format from crashing the entire workflow.

---

# HTML Report Generation Layer

The workflow combines:

* Campaign information
* Calculated metrics
* Period comparisons
* Performance trends
* AI-generated insights

into a client-ready HTML report.

The report includes:

* Branded header
* Campaign information
* Metric cards
* Trend indicators
* Performance summary
* Key wins
* Areas for attention
* Recommendations

The report uses responsive HTML and inline CSS so it can be displayed directly in email clients.

---

# Supabase — Report Storage Layer

Supabase provides persistent storage for generated reports.

The workflow stores report information in the `reports` table.

Stored information can include:

* Client name
* Platform
* Campaign name
* Reporting period
* Report content
* Creation timestamp

This provides historical record-keeping for generated reports.

```text
Generated Report
       ↓
Supabase
       ↓
Report History
```

Using persistent storage allows the system to retain report information after a workflow execution is complete.

---

# Gmail — Report Delivery Layer

Gmail is responsible for delivering the completed report to the client.

The generated HTML report is sent as the email body.

```text
HTML Report
     ↓
Gmail
     ↓
Client Inbox
```

Sending the report directly in the email body allows the client to view the report without downloading an attachment.

---

# Optional Local Report Preview

The architecture includes a report-to-file conversion path for local preview and testing.

```text
Generated HTML Report
        ↓
Convert Report to File
        ↓
Local Preview
```

This path is primarily useful for verifying the generated report output during development.

The client delivery workflow uses the HTML report directly as the email body.

---

# Error Handling Architecture

AgencyReport AI includes multiple layers of error handling.

```text
                ┌──────────────────────┐
                │     Workflow Run     │
                └──────────┬───────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Node Failure?  │
                  └────────┬────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
             No                        Yes
              │                         │
              ▼                         ▼
       Continue Workflow        Retry / Handle Error
                                        │
                                        ▼
                              Continue or Error Workflow
                                        │
                                        ▼
                                  Email Notification
```

The system uses different strategies depending on the failure type.

| Failure Point            | Handling Strategy                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------- |
| Missing required data    | Invalid records are filtered before processing                                           |
| Invalid numeric values   | Invalid records are removed before calculations                                          |
| Division by zero         | Safe division logic prevents `Infinity` and `NaN` values                                 |
| Invalid AI JSON response | Response parsing is protected with error handling and fallback behavior                  |
| Temporary AI API failure | AI request retry logic attempts the request again                                        |
| Individual item failure  | Failure isolation prevents one problematic item from immediately stopping all processing |
| Workflow-level failure   | Dedicated error handler workflow sends an email notification                             |

---

# Dedicated Error Handler Workflow

The project includes a separate:

```text
AgencyReport AI - Error Handler
```

workflow.

The error workflow uses n8n's Error Trigger to listen for workflow-level failures.

When a workflow error occurs:

```text
Main Workflow Failure
        ↓
Error Trigger
        ↓
Capture Error Information
        ↓
Send Email Notification
        ↓
Administrator
```

The error notification can include:

* Workflow name
* Error message
* Timestamp

This ensures important workflow failures are visible and actionable.

---

# Credential and Secret Management

Sensitive credentials are managed using n8n's built-in credential system.

This includes:

* Google OAuth credentials
* Gmail credentials
* Supabase credentials
* Google Gemini API credentials

Credentials are not hardcoded directly into workflow logic.

The repository should not contain:

* API keys
* OAuth tokens
* Passwords
* Database secrets

The exported workflow references credentials that must be configured by the user importing the workflow into their own n8n environment.

```text
GitHub Repository
        ↓
Workflow JSON
        ↓
Credential References Only
        ↓
User Connects Their Own Credentials
```

This allows the project to be safely shared as a portfolio repository without exposing sensitive information.

---

# Key Design Decisions

## Why n8n?

n8n was selected as the orchestration layer because it provides:

* Visual workflow automation
* Integration with multiple services
* Easy debugging
* Node-level inspection
* Manual testing
* Scheduled execution
* Self-hosting support

The visual workflow also makes the project easier to demonstrate and understand.

---

## Why Google Sheets Instead of Live Advertising APIs?

Version 1 uses Google Sheets as a simulated campaign data source.

This keeps the project:

* Easy to test
* Easy to demonstrate
* Fully controllable
* Free to build for portfolio purposes

The rest of the pipeline remains focused on processing campaign data rather than being dependent on a specific advertising API.

A future version could replace the data-source layer with live integrations such as:

* Meta Ads API
* Google Ads API

without changing the overall reporting pipeline concept.

---

## Why Google Gemini?

Google Gemini is used to generate AI-powered performance insights.

The AI model provides the language-generation layer of the system while workflow logic remains responsible for deterministic calculations.

This separation allows:

```text
Automation Logic
        +
Real Metrics
        +
AI Interpretation
        =
Automated Performance Reporting
```

---

## Why Separate Data Preparation and Metric Calculation?

The workflow separates data organization from metric calculation.

### Reporting Data Preparation

Responsible for:

* Organizing campaign records
* Identifying reporting periods
* Preparing data for comparison

### Metric Calculation

Responsible for:

* Calculating marketing metrics
* Handling division safety
* Calculating performance changes

This separation keeps workflow responsibilities clear and makes individual stages easier to test and debug.

---

## Why Validate Data Before Processing?

Validation occurs before grouping and calculations.

```text
Raw Data
    ↓
Validation
    ↓
Processing
```

This prevents invalid records from affecting:

* Grouping logic
* Metric calculations
* AI prompts
* Generated reports

Early validation improves the reliability of the downstream workflow.

---

## Why Supabase?

Supabase provides persistent PostgreSQL storage without requiring custom database infrastructure.

It allows the project to demonstrate:

* Persistent data storage
* Report history
* Structured records

rather than relying only on temporary workflow execution data.

---

## Why HTML Email Instead of PDF?

The report is delivered as an HTML email body.

This allows the client to:

* Open the report immediately
* View performance metrics directly
* Read AI-generated insights without downloading a file

```text
Email Opened
      ↓
Report Immediately Visible
```

PDF export remains a possible future enhancement.

---

# Architectural Principles

AgencyReport AI follows several core design principles.

## 1. Validate Before Processing

Data quality checks happen before calculations and AI processing.

---

## 2. Calculate Before AI

The workflow calculates metrics first.

The AI interprets the calculated data rather than generating the numbers.

---

## 3. Keep Components Focused

Each stage has a clear responsibility:

```text
Sheets
  ↓
Data Source

n8n
  ↓
Automation

Code Logic
  ↓
Data Processing

Gemini
  ↓
Insight Generation

Supabase
  ↓
Storage

Gmail
  ↓
Delivery
```

---

## 4. Handle Failures Explicitly

The workflow includes validation, safe calculations, retries, item-level failure handling, and workflow-level error notifications.

---

## 5. Keep Credentials Separate from Code

Secrets are managed through the workflow platform's credential system and are not intended to be committed to the repository.

---

# End-to-End Data Flow

```text
Google Sheets
      ↓
Read Campaign Data
      ↓
Validate Records
      ↓
Prepare Reporting Data
      ↓
Calculate Metrics
      ↓
Compare Reporting Periods
      ↓
Prepare Structured AI Prompt
      ↓
Google Gemini
      ↓
Parse AI Response
      ↓
Generate HTML Report
      ↓
      ├──────────────► Supabase
      │                 Store Report History
      │
      └──────────────► Gmail
                        Deliver Report to Client
```

---

# Summary

AgencyReport AI is designed as a modular AI-powered reporting automation system.

The architecture separates responsibilities across the pipeline:

| Component        | Responsibility                                 |
| ---------------- | ---------------------------------------------- |
| Google Sheets    | Campaign data source                           |
| n8n              | Workflow orchestration                         |
| Validation Logic | Data quality checks                            |
| Reporting Logic  | Data preparation and period comparison         |
| Metric Logic     | Marketing metric calculations                  |
| Google Gemini    | Performance interpretation and recommendations |
| HTML Generator   | Client-ready report creation                   |
| Supabase         | Persistent report storage                      |
| Gmail            | Automated report delivery                      |
| Error Handler    | Workflow failure notifications                 |

This architecture allows campaign data to move through a structured pipeline and become a data-driven, AI-assisted client report with minimal manual intervention.
