# AI Lead Qualification System — Core

**Day 33 — Development: Core Build**

## 1. Project Overview

The **AI Lead Qualification System — Core** is the deterministic foundation of a lead qualification automation workflow built in **n8n**.

The Day 33 implementation focuses on building the non-AI backbone of the system before introducing AI-based qualification and enrichment in later stages.

The workflow accepts leads from two different sources:

* Real-time website/form submissions through a Webhook
* Manual lead entries through Google Sheets

Both sources are normalized into a common lead structure and processed through validation, duplicate detection, deterministic business rules, database storage, spreadsheet synchronization, and error handling.

---

## 2. Problem Statement

Businesses can receive leads from multiple sources, making it difficult to consistently validate, organize, score, and store incoming lead information.

Without an automated process, leads may:

* Contain missing or invalid information
* Be entered multiple times
* Require manual qualification
* Be stored inconsistently across systems
* Fail silently when a database or storage operation encounters an error

This workflow provides a reliable deterministic foundation that handles these tasks automatically before AI-based qualification is introduced.

---

## 3. Objective

The main objectives of the Day 33 core build are to:

* Accept leads from multiple sources
* Normalize different input formats into one common schema
* Validate essential lead information
* Reject invalid leads safely
* Detect duplicate leads using email
* Apply deterministic business rules
* Calculate a preliminary lead score
* Classify leads as Hot, Warm, or Cold
* Store new leads in Supabase
* Update existing leads when duplicates are detected
* Maintain a Google Sheets CRM mirror
* Log validation and database errors
* Return appropriate responses to webhook requests
* Follow clean workflow and node-naming standards
* Keep credentials and secrets outside the workflow logic

---

## 4. Workflow Architecture

```text
                    ┌─────────────────────────┐
                    │ Webhook - New Lead      │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ Normalize Webhook       │
                    └────────────┬────────────┘
                                 │
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ Merge Lead Sources      │
                    └────────────┬────────────┘
                                 ▲
                                 │
                    ┌────────────┴────────────┐
                    │ Normalize Sheet Row     │
                    └────────────▲────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │ Sheet Trigger           │
                    │ Poll Manual Leads       │
                    └─────────────────────────┘

                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ Validate Lead Fields    │
                    └────────────┬────────────┘
                                 │
                         ┌───────┴───────┐
                         │               │
                      Valid           Invalid
                         │               │
                         ▼               ▼
                Check Duplicate     Log Validation
                     Lead               Error
                         │               │
                         ▼               ▼
                Merge Duplicate    Set 400 Response
                     Result               │
                         │               │
                         ▼               │
                Apply Business Rules      │
                         │               │
                         ▼               │
                Existing Lead?             │
                   /       \              │
                 Yes        No             │
                  │          │             │
                  ▼          ▼             │
               Update      Insert          │
                 Lead        Lead           │
                  │          │             │
                  └────┬─────┘             │
                       │                   │
                       ▼                   │
                Write Sheet Mirror         │
                       │                   │
                       ▼                   │
                Set Success Response       │
                       │                   │
                       └─────────┬─────────┘
                                 │
                                 ▼
                     Merge Response Paths
                                 │
                                 ▼
                       Triggered by Webhook?
                          /             \
                        Yes             No
                         │               │
                         ▼               ▼
                 Respond to Webhook   No Response
```

---

## 5. Technologies Used

* **n8n** — Workflow automation and orchestration
* **Supabase** — Lead database and error logging
* **Google Sheets** — Manual lead input and CRM mirror
* **JavaScript** — Data normalization, validation, and deterministic business rules
* **Webhooks** — Real-time lead intake
* **REST-style JSON payloads** — Lead data exchange

---

## 6. Workflow Nodes

### Trigger Nodes

#### Webhook - New Lead

Receives new leads through an HTTP POST request.

**Configuration:**

* Method: `POST`
* Path: `lead-intake`
* Response Mode: `Using Respond to Webhook Node`

---

#### Sheet Trigger - Poll Manual Leads

Checks Google Sheets for newly added manual leads.

**Configuration:**

* Polling interval: Every 5 minutes
* Event: Row Added

---

### Data Normalization

#### Normalize - Webhook Payload

Converts webhook data into the common lead schema.

Normalized fields:

```text
name
email
company
title
phone
message
source
raw_source
```

The email address is also trimmed and converted to lowercase.

---

#### Normalize - Sheet Row

Converts Google Sheets column names into the same common schema used by webhook leads.

This allows both sources to use the same downstream validation and processing logic.

---

### Source Combination

#### Merge - Combine Lead Sources

Combines the normalized webhook and Google Sheets streams into one processing path.

This ensures downstream nodes do not need separate logic for each source.

---

## 7. Lead Validation

### Validate Lead Fields

The validation node checks the following required information:

* Name
* Email
* Company

It also validates the email format using a JavaScript regular expression.

The node generates:

```text
isValid
validationErrors
```

Example:

```json
{
  "isValid": true,
  "validationErrors": []
}
```

If validation fails, the lead is routed to the validation error branch.

---

## 8. Validation Error Handling

Invalid leads are sent to:

### Log Validation Error

The invalid lead is stored in the `lead_errors` Supabase table.

The error log contains information such as:

* Lead email
* Lead name
* Error type
* Validation details
* Source trigger
* Raw payload
* Timestamp

The workflow then creates a `400` response using:

### Set Response - Validation Failed

Example response status:

```text
400 Bad Request
```

This prevents invalid leads from continuing into the database qualification and storage process.

---

## 9. Duplicate Lead Detection

### Check Duplicate Lead

Valid leads are checked against the Supabase `leads` table using the lead's email address.

The filter is:

```text
email = {{ $json.email }}
```

The node uses `alwaysOutputData` so that a lead with no existing database match can continue through the workflow instead of terminating the branch.

---

### Merge Duplicate Check Result

The result is converted into:

```text
duplicateFound
existingId
```

Example for an existing lead:

```json
{
  "duplicateFound": true,
  "existingId": "existing-record-uuid"
}
```

Example for a new lead:

```json
{
  "duplicateFound": false,
  "existingId": null
}
```

## 11. Existing vs New Lead

### IF - Existing Lead Record

The workflow checks:

```text
duplicateFound == true
```

### Existing Lead

If the lead already exists, the workflow sends it to:

**Update Lead Record**

The existing record is updated using its UUID.

### New Lead

If no matching record exists, the workflow sends it to:

**Insert Lead Record**

A new Supabase record is created.

---

## 12. Supabase Database

The workflow uses two Supabase tables.

### `leads`

The main lead storage table should contain fields corresponding to the data used by the workflow.

Recommended structure:

| Column                 | Type      |
| ---------------------- | --------- |
| `id`                   | UUID      |
| `name`                 | Text      |
| `email`                | Text      |
| `company`              | Text      |
| `title`                | Text      |
| `phone`                | Text      |
| `message`              | Text      |
| `source`               | Text      |
| `rule_based_score`     | Numeric   |
| `preliminary_tier`     | Text      |
| `rule_reasoning`       | Text      |
| `qualification_status` | Text      |
| `status`               | Text      |
| `created_at`           | Timestamp |
| `updated_at`           | Timestamp |

The `id` field should remain a UUID and should be generated by Supabase.

---

### `lead_errors`

This table stores validation and database-related errors.

The column names must match the fields configured in the corresponding n8n Supabase nodes.

For the current workflow, ensure that the fields used by both error logging nodes exist and are consistent.

Example fields include:

```text
email
name
error_type
error_message
lead_email
lead_name
error_detail
source_trigger
raw_payload
occurred_at
```

> If the workflow is standardized before deployment, the validation-error and database-error nodes should use one consistent naming convention.

---

## 13. Google Sheets Integration

Google Sheets is used for two purposes:

### Manual Lead Input

New rows can be entered manually into the spreadsheet.

Expected input fields include:

```text
Name
Email
Company
Title
Phone
Message
Source
```

The Sheet Trigger detects newly added rows and sends them into the same processing pipeline.

### CRM Mirror

After successful Supabase storage, the workflow uses:

**Write Sheet Mirror**

to keep the spreadsheet synchronized with the processed lead data.

The spreadsheet uses the `Email` column as the matching field for append/update behavior.

---

## 14. Error Handling

The workflow includes separate handling for different failure types.

### Validation Failure

```text
Invalid Lead
     ↓
Log Validation Error
     ↓
Set Response - Validation Failed
     ↓
400 Response
```

### Database Write Failure

Both insert and update operations have an error output.

```text
DB Write Error
     ↓
Log DB Write Error
     ↓
Set Response - Storage Failed
```

The storage failure response uses:

```text
500
```

and informs the caller that the lead could not be saved.

### Webhook vs Sheet Execution

The final branch checks:

```text
raw_source == "webhook"
```

If the workflow was triggered by a webhook, it sends the result using:

**Respond - Send Result**

If the workflow came from Google Sheets, it uses:

**No Response Needed - Sheet Source**

This prevents the Sheet-triggered execution from attempting to respond to a webhook that does not exist.

---

## 15. Test Cases

### Test Case 1 — Valid New Lead

**Input:**

```json
{
  "name": "Ahmed Khan",
  "email": "ahmedkhan@example.com",
  "company": "Tech Solutions Ltd",
  "title": "Head of Operations",
  "phone": "+923001234567",
  "message": "We are interested in automating our lead qualification and customer follow-up process.",
  "source": "Website"
}
```

**Expected result:**

* Validation passes
* Duplicate check returns no existing record
* Business rules calculate the preliminary score
* Lead is classified
* New lead is inserted into Supabase
* Google Sheets mirror is updated
* Successful webhook response is returned

Example deterministic result:

```text
Score: 85
Tier: Hot
Status: Pending AI Review
```

---

### Test Case 2 — Invalid Lead

Example:

```json
{
  "name": "Ahmed Khan",
  "email": "invalid-email",
  "company": ""
}
```

**Expected result:**

* Validation fails
* Validation errors are generated
* Lead is stored in `lead_errors`
* Lead does not continue to the main qualification/storage path
* Webhook receives a `400` response

---

### Test Case 3 — Duplicate Lead

Use the same email address as an existing lead.

**Expected result:**

* Validation passes
* Duplicate check finds the existing record
* `duplicateFound` becomes `true`
* Existing UUID is captured in `existingId`
* Existing Supabase record is updated instead of creating another record
* Google Sheets mirror is updated

---

### Test Case 4 — Database Write Failure

Simulate an invalid database configuration or database write issue.

**Expected result:**

* Database error is captured
* Error is logged in `lead_errors`
* Storage failure response is generated
* Webhook receives a `500` response when applicable

---

## 16. Setup Instructions

### Step 1 — Import the Workflow

In n8n:

```text
Workflows
→ Import from File
→ Select AI_Lead_Qualification_Core_Day_33.json
```

---

### Step 2 — Configure Supabase

Create the required Supabase tables:

```text
leads
lead_errors
```

Make sure the database column names match the fields configured in the n8n Supabase nodes.

Add the Supabase credential to the relevant nodes.

---

### Step 3 — Configure Google Sheets

Connect the Google Sheets credential.

Configure the spreadsheet used for:

* Manual lead input
* CRM mirror

Make sure the sheet contains the required columns.

---

### Step 4 — Configure Webhook

The webhook endpoint is:

```text
POST /lead-intake
```

Use the n8n-generated Test URL while testing.

For production use, use the Production URL after activating the workflow.

---

### Step 5 — Test the Workflow

Test at least:

1. Valid new lead
2. Invalid lead
3. Duplicate lead
4. Storage error

Confirm that each branch behaves as expected.

---

### Step 6 — Activate for Production Testing

After successful testing, activate the workflow if production webhook execution is required.

---

## 17. Credentials Required

Credentials are referenced by n8n and must be configured separately.

Required credentials:

```text
Supabase account
Google Sheets account
Google Sheets Trigger account
```

> **Security:** Never include API keys, passwords, access tokens, or other secret values in this repository.

---

## 18. Security Practices

The workflow follows basic credential-safety practices:

* Credentials are stored using n8n's credential system
* Secrets are not hardcoded into JavaScript Code nodes
* API keys and passwords are not included in the workflow documentation
* Database credentials are not exposed in README files
* Sensitive credential values should not be committed to GitHub

Before publishing the exported workflow JSON, verify that no secret values are embedded in the export.

---

## 19. Known Limitations

This Day 33 version intentionally has several limitations:

* AI-based lead qualification is not implemented yet
* Lead scoring is deterministic rather than AI-driven
* Lead enrichment is not included
* The business rules are manually defined
* Google Sheets is used as a CRM mirror rather than a complete CRM platform
* Error logging fields should remain consistent between validation and database error paths
* The system depends on the configured Supabase and Google Sheets services
* The workflow currently uses email as the duplicate detection key

---

## 20. Future Improvements

Future versions can extend the deterministic foundation with:

* AI-powered lead qualification
* AI-generated lead summaries
* AI-based intent detection
* Lead enrichment
* More advanced duplicate detection
* CRM integrations
* Automated email follow-up
* Human approval for sensitive/high-value decisions
* Lead prioritization
* Analytics and reporting
* Monitoring and alerting
* Retry strategies for temporary API failures
* More advanced database constraints and indexing
* Production-grade logging and observability

---

## 21. Day 33 Deliverables

The Day 33 submission includes:

### Workflow JSON

```text
AI_Lead_Qualification_Core_Day_33.json
```

Contains the complete non-AI workflow.

### Successful Execution Screenshot

Shows the core workflow executing successfully through the main processing path.

### Optional Error-Handling Screenshot

Shows an invalid lead being rejected and logged through the validation error branch.

---

## 22. Day 33 Completion Summary

The Day 33 implementation successfully establishes the deterministic backbone of the AI Lead Qualification System.

The workflow can:

```text
Receive Lead
     ↓
Normalize Data
     ↓
Validate Fields
     ↓
Check Duplicate
     ↓
Apply Business Rules
     ↓
Insert / Update Database
     ↓
Mirror to Google Sheets
     ↓
Return Result
```

Invalid and storage-failure scenarios are handled through dedicated error paths.

The architecture is intentionally designed so that the deterministic core can be extended with AI-based qualification and enrichment in later stages without rebuilding the entire workflow.

---
