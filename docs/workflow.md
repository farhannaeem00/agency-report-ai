# Workflow Walkthrough

This document explains the **AgencyReport AI** automation workflow node by node.

It describes:

* What each node does
* Why the node exists
* How data moves through the workflow
* How marketing metrics are calculated
* How AI insights are generated safely
* How reports are stored and delivered
* How errors are handled

The main workflow transforms campaign data into a client-ready AI-powered marketing performance report.

---

# Main Workflow Overview

The workflow follows this sequence:

```text
Trigger
   ↓
Read Campaign Data
   ↓
Validate Data
   ↓
Group by Client & Platform
   ↓
Calculate Metrics
   ↓
Compare Reporting Periods
   ↓
Prepare AI Input
   ↓
Generate AI Insights
   ↓
Parse AI Response
   ↓
Generate HTML Report
   ↓
   ├───────────────┐
   ▼               ▼
Preview File    Store Report
                    ↓
                Send Email
```

The main workflow supports both manual and scheduled execution.

---

# Triggers

## Manual Trigger

### Node

`When clicking 'Execute workflow'`

The manual trigger allows the workflow to be executed directly from n8n.

It is primarily used for:

* Testing the workflow
* Debugging individual nodes
* Demonstrating the automation
* Running reports on demand

---

## Weekly Schedule

### Node

`Weekly Schedule`

The Schedule Trigger automatically starts the workflow according to the configured reporting schedule.

In the current implementation, the workflow is configured to run:

```text
Every Monday
```

This allows the reporting pipeline to run automatically without manual execution.

---

## Shared Workflow Entry Point

Both triggers connect to the same processing pipeline.

```text
Manual Trigger
       │
       │
       ├───────────┐
       │           │
       ▼           ▼
                  Read Campaign Data
                         ▲
                         │
                         │
                  Weekly Schedule
```

This avoids duplicated workflow logic.

Regardless of how the workflow starts, all data follows the same reporting pipeline.

---

# Step 1 — Read Campaign Data

### Node Type

**Google Sheets**

### Authentication

OAuth2

### Operation

`Get Row(s)`

---

The workflow reads campaign data from the configured Google Sheet.

Each row represents campaign performance data for a reporting period.

The data source contains information such as:

* Client name
* Platform
* Campaign name
* Reporting period
* Advertising spend
* Impressions
* Clicks
* Conversions
* Revenue

Example data structure:

```text
Client Name
Platform
Campaign Name
Reporting Period
Spend
Impressions
Clicks
Conversions
Revenue
```

### Purpose

Google Sheets acts as the Version 1 campaign data source.

```text
Google Sheets
      ↓
Campaign Records
      ↓
n8n Workflow
```

---

# Step 2 — Validate Data

### Node Type

**Filter**

The validation step removes incomplete or invalid campaign records before they enter the processing pipeline.

The workflow checks that required fields such as:

* `client_name`
* `platform`
* `reporting_period`

are present.

The workflow also validates numeric fields such as:

* `spend`
* `clicks`
* `impressions`

Numeric values must be valid and greater than or equal to zero.

---

## Validation Flow

```text
Campaign Data
      ↓
Validate Record
      ↓
 ┌───────────────┐
 │               │
Valid          Invalid
 │               │
 ▼               ▼
Continue       Discard
```

Rows that fail validation are routed to the discarded output and do not continue through the reporting pipeline.

### Why Validation Happens Early

Validating data before processing prevents:

* Missing values
* Invalid numeric calculations
* Unexpected workflow errors
* Incorrect AI input
* Incorrect reports

---

# Step 3 — Group by Client and Platform

### Node Type

**Code**

### Language

JavaScript

### Execution Mode

Run Once for All Items

---

After validation, campaign records are grouped using:

```text
client_name + platform
```

Each group represents a reporting context for a specific client and platform.

The records inside each group are sorted by:

```text
reporting_period
```

The workflow identifies:

* The most recent period as the **current period**
* The previous period as the **previous period**

---

## Example

Input:

```text
Client: ABC Agency

Meta Ads
2026-07
2026-08
```

Output:

```text
Current Period
2026-08

Previous Period
2026-07
```

The node outputs one workflow item for each client/platform group.

---

## Purpose

This step prepares campaign data for period-over-period comparison.

```text
Validated Rows
      ↓
Group by Client + Platform
      ↓
Sort by Reporting Period
      ↓
Current + Previous Period
```

---

# Step 4 — Calculate Metrics and Performance Comparison

### Node Type

**Code**

### Language

JavaScript

### Execution Mode

Run Once for All Items

---

This node calculates marketing metrics for the reporting data.

The workflow calculates:

---

## CTR — Click-Through Rate

```text
CTR = (Clicks / Impressions) × 100
```

---

## Conversion Rate

```text
Conversion Rate = (Conversions / Clicks) × 100
```

---

## CPC — Cost Per Click

```text
CPC = Spend / Clicks
```

---

## CPA — Cost Per Acquisition

```text
CPA = Spend / Conversions
```

---

## ROAS — Return on Ad Spend

```text
ROAS = Revenue / Spend
```

---

# Safe Division Handling

All calculations use a `safeDiv()` helper function.

If a denominator is:

```text
0
```

the function returns:

```text
0
```

instead of producing:

* `Infinity`
* `NaN`
* Runtime calculation errors

Example:

```text
Spend = 500
Clicks = 0

CPC = 0
```

This keeps the workflow stable when campaign data contains zero values.

---

# Performance Comparison

The workflow calculates metric changes between:

```text
Current Period
       vs
Previous Period
```

Where applicable, percentage changes are calculated.

Example:

```text
Previous CTR: 2.5%

Current CTR: 3.0%

Change: +20%
```

The calculated metrics and performance changes are used by:

* The AI insight generation step
* The HTML report generation step

---

# Step 5 — Prepare AI Input

### Node Type

**Code**

### Language

JavaScript

### Execution Mode

Run Once for All Items

---

This node prepares the structured input sent to Google Gemini.

The prompt contains real workflow-generated data, including:

### Campaign Data

* Spend
* Impressions
* Clicks
* Conversions
* Revenue

### Calculated Metrics

* CTR
* Conversion Rate
* CPC
* CPA
* ROAS

### Reporting Comparison

* Current period metrics
* Previous period metrics
* Percentage changes

---

## AI Grounding

The prompt explicitly instructs the AI:

> Use only the provided data. Do not invent numbers, metrics, or unsupported results.

The expected response format is also defined.

Example structure:

```json
{
  "summary": "",
  "key_wins": [],
  "areas_for_attention": [],
  "recommendations": []
}
```

---

## Purpose

The workflow prepares the data before sending it to the AI.

```text
Real Campaign Data
        ↓
Workflow Calculations
        ↓
Structured AI Prompt
        ↓
Google Gemini
```

The AI does not calculate campaign metrics.

Instead:

```text
Workflow
   ↓
Calculates Numbers
   ↓
AI
   ↓
Explains Performance
```

---

# Step 6 — Generate AI Insights

### Node Type

**HTTP Request**

### Service

Google Gemini API

### Model

`gemini-3.6-flash`

### Authentication

Query Authentication credential

The API key is stored in the n8n credential system and is not hardcoded in the workflow.

---

## API Request

The prepared AI prompt is sent to Google Gemini.

Gemini receives:

* Real campaign data
* Calculated metrics
* Period comparison data
* Instructions for generating structured insights

The AI generates:

* Performance summary
* Key wins
* Areas for attention
* Recommendations

---

# Retry Handling

The node is configured with retry logic.

```text
AI Request
     ↓
Failure
     ↓
Retry 1
     ↓
Retry 2
     ↓
Retry 3
```

The workflow is configured for:

* Up to 3 retry attempts
* 2-second delay between attempts

This helps handle temporary failures such as:

* Temporary API errors
* Service overload
* Short-lived rate-limit issues

---

# Continue On Fail

`Continue On Fail` is enabled for this step.

If one AI request fails even after retries:

```text
Group A
   ↓
AI Success
   ↓
Continue

Group B
   ↓
AI Failure
   ↓
Continue

Group C
   ↓
AI Success
   ↓
Continue
```

This prevents one failed AI request from automatically stopping all other workflow items.

---

# Step 7 — Parse AI Response

### Node Type

**Code**

### Language

JavaScript

### Execution Mode

Run Once for All Items

---

Google Gemini returns its response inside the standard API response structure.

The workflow extracts the generated text from:

```text
candidates[0]
   ↓
content
   ↓
parts[0]
   ↓
text
```

---

# Response Processing

The workflow:

1. Extracts the AI-generated text
2. Removes Markdown code fences if present
3. Parses the response as JSON
4. Restores the original campaign and metric data
5. Combines the AI insights with the workflow data

The original data is retrieved from:

```text
Prepare AI Input
```

because the HTTP Request node returns the Gemini API response as its primary output.

---

# Invalid AI Response Handling

If the AI response cannot be parsed as valid JSON:

```text
AI Response
      ↓
Try JSON Parse
      ↓
 ┌─────────────┐
 │             │
Success      Failure
 │             │
 ▼             ▼
Use Data    Return Fallback
```

A fallback object is returned containing a:

```text
parse_error
```

field.

This prevents an unexpected AI response format from immediately crashing the workflow.

---

# Step 8 — Generate HTML Report

### Node Type

**Code**

### Language

JavaScript

### Execution Mode

Run Once for All Items

---

This node creates the final client-ready report.

The workflow combines:

* Client information
* Campaign information
* Reporting periods
* Calculated metrics
* Performance comparisons
* AI-generated insights

into a complete HTML document.

---

# Report Contents

The generated report includes:

## Branded Header

A branded header introduces the report.

---

## Campaign Overview

The report displays campaign context such as:

* Client
* Platform
* Campaign
* Reporting period

---

## Key Metrics

The report displays:

* CTR
* Conversion Rate
* CPC
* CPA
* ROAS

---

## Performance Trends

Metric changes are displayed with trend indicators.

The logic correctly handles the difference between:

### Performance Metrics

Higher values are generally considered positive for metrics such as:

* CTR
* Conversion Rate
* ROAS

### Cost Metrics

Lower values are generally considered positive for:

* CPC
* CPA

For example:

```text
ROAS increases
      ↓
Positive Trend

CPA decreases
      ↓
Positive Trend
```

---

## AI Insights

The report includes:

* Performance summary
* Key wins
* Areas for attention
* Recommendations

---

# Step 9A — Convert Report to File

### Node Type

**Convert to File**

### Format

HTML

---

The generated `report_html` content can be converted into a binary `.html` file.

This branch is primarily used for:

* Local preview
* Testing
* Downloading the generated report
* Verifying HTML output in a browser

```text
HTML Report
     ↓
Convert to File
     ↓
HTML File
     ↓
Browser Preview
```

This branch is not required for client email delivery.

---

# Step 9B — Store Report in Supabase

### Node Type

**Supabase**

### Operation

Insert

### Table

`reports`

---

The generated report is stored in Supabase.

Each report record can include:

* `client_name`
* `platform`
* `campaign_name`
* `reporting_period`
* `report_content`

The database automatically generates:

* `id`
* `created_at`

---

## Storage Flow

```text
Generated Report
       ↓
Supabase Insert
       ↓
Reports Table
       ↓
Historical Record
```

This allows report information to persist after the workflow execution is complete.

---

# Step 10 — Send Report Email

### Node Type

**Gmail**

### Operation

Send

### Email Format

HTML

---

The completed report is delivered directly to the client.

The email includes:

* A report-specific subject line
* Client reporting information
* The generated HTML report

The report is placed directly inside the email body.

```text
Generated HTML Report
        ↓
Gmail
        ↓
Client Inbox
        ↓
Client Views Report
```

No attachment is required for the client to view the report.

---

# Main Workflow Data Flow

The complete workflow can be summarized as:

```text
Manual Trigger
       │
Weekly Schedule
       │
       ▼
Read Campaign Data
       ↓
Validate Data
       ↓
Group by Client + Platform
       ↓
Identify Current + Previous Period
       ↓
Calculate Metrics
       ↓
Calculate Performance Changes
       ↓
Prepare Structured AI Input
       ↓
Google Gemini
       ↓
Parse AI Response
       ↓
Generate HTML Report
       ↓
       ├──────────────────┐
       ▼                  ▼
Convert to File        Supabase
(Local Preview)        Store Report
                              ↓
                           Gmail
                              ↓
                        Client Delivery
```

---

# Error Handler Workflow

The project includes a separate workflow:

```text
AgencyReport AI - Error Handler
```

This workflow is configured as the main workflow's error workflow.

---

## Error Flow

```text
Main Workflow
      ↓
Workflow-Level Failure
      ↓
Error Trigger
      ↓
Capture Error Information
      ↓
Send Email Alert
      ↓
Administrator
```

---

# Step 1 — Error Trigger

### Node Type

**Error Trigger**

The Error Trigger activates automatically when the configured main workflow encounters a workflow-level failure.

The error workflow receives information about the failure.

This can include:

* Workflow name
* Error message
* Failure timestamp

---

# Step 2 — Send Error Notification

### Node Type

**Gmail**

The workflow sends an error notification email to the administrator.

The notification helps ensure that workflow failures are visible and can be investigated.

Example:

```text
Workflow Error Detected

Workflow:
AgencyReport AI

Error:
[Error Message]

Time:
[Timestamp]
```

---

# Error Handling Summary

The system includes multiple layers of protection.

| Failure                       | Protection                              |
| ----------------------------- | --------------------------------------- |
| Missing required fields       | Invalid records are filtered out        |
| Invalid numeric values        | Records are removed before calculations |
| Division by zero              | `safeDiv()` returns a safe value        |
| Temporary AI API failure      | Retry logic                             |
| Individual AI request failure | Continue On Fail                        |
| Invalid AI JSON response      | Parse fallback handling                 |
| Workflow-level failure        | Dedicated Error Handler workflow        |
| Workflow failure notification | Gmail alert                             |

---

# Workflow Design Principles

The workflow follows several key principles.

## 1. Validate Early

```text
Raw Data
   ↓
Validation
   ↓
Processing
```

Invalid data is removed before it can affect downstream logic.

---

## 2. Calculate Before AI

```text
Campaign Data
      ↓
Workflow Calculations
      ↓
AI Interpretation
```

The workflow performs deterministic calculations.

The AI generates written interpretation and recommendations.

---

## 3. Keep Nodes Focused

Each workflow node has a specific responsibility.

```text
Google Sheets
      ↓
Read Data

Filter
      ↓
Validate Data

Code
      ↓
Prepare Data

Code
      ↓
Calculate Metrics

HTTP Request
      ↓
Generate AI Insights

Code
      ↓
Generate Report

Supabase
      ↓
Store Report

Gmail
      ↓
Deliver Report
```

This makes the workflow easier to:

* Debug
* Test
* Modify
* Extend

---

## 4. Handle Errors Explicitly

The workflow does not assume every integration will always succeed.

It includes:

* Input validation
* Safe calculations
* AI retry logic
* Continue On Fail
* JSON parsing fallback
* Dedicated workflow error notifications

---

## 5. Keep Sensitive Credentials Separate

API keys and authentication credentials are managed through n8n's credential system.

The workflow does not require secrets to be hardcoded into the repository.

---

# Summary

AgencyReport AI is an end-to-end marketing reporting automation workflow.

It transforms:

```text
Campaign Data
      ↓
Validated Data
      ↓
Calculated Metrics
      ↓
Period Comparison
      ↓
AI Performance Insights
      ↓
HTML Report
      ↓
Database Storage
      ↓
Client Email Delivery
```

The workflow demonstrates practical automation engineering by combining:

* Data processing
* Marketing metric calculations
* AI integration
* Safe AI prompting
* Structured AI output
* HTML generation
* Database storage
* Email automation
* Production-style error handling

The result is a complete automation pipeline that demonstrates how recurring marketing reporting can be streamlined with workflow automation and AI.
