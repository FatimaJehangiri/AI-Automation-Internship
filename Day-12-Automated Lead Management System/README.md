# Automated Lead Management System

**Day 12 — Week 2 Assignment**
Author: Fatima Wajid | Program: n8n Automation Training

## 1. What This Does

A webhook-driven pipeline that takes in a raw lead submission, validates it, scores it, routes it to the right place based on priority, and sends the lead an automatic acknowledgment — all with zero manual work.

```
Webhook → Validate → (invalid → 400 error response)
                   → Lead Scoring → Priority Router
                                       ├── High   → Notify Sales (Gmail)
                                       ├── Medium → Save to Lead Database (Sheets)
                                       └── Low    → Add to Nurture List (Sheets)
                                                       ↓
                                          Send Acknowledgment Email (Gmail)
                                                       ↓
                                              Respond Success (200)
```

## 2. Node-by-Node Breakdown

| # | Node | Type | Purpose |
|---|------|------|---------|
| 1 | Lead Intake Webhook | Webhook | Public POST endpoint that accepts new leads. Uses `responseNode` mode so we control exactly when/what gets returned. |
| 2 | Validate Lead Data | Code | Checks all 5 required fields exist, confirms email format with regex, confirms budget is numeric. Adds `isValid` + `validationErrors` to the item. |
| 3 | IF Valid? | IF | Branches on `isValid`. False → immediate 400 response. True → continues to scoring. |
| 4 | Lead Scoring | Code | Rule-based scoring (see §4 below). Adds `leadScore` and `priority`. |
| 5 | Priority Router | Switch | Routes the item down one of three branches based on `priority` (High / Medium / Low). |
| 6 | Notify Sales - High Priority | Gmail | Sends an internal alert email to your sales inbox — used only for High priority leads. |
| 7 | Save to Lead Database | Google Sheets | Appends the lead to a "Leads" sheet — used for Medium priority. |
| 8 | Add to Nurture List | Google Sheets | Appends the lead to a "Nurture List" sheet — used for Low priority. |
| 9 | Send Acknowledgment Email | Gmail | Sends every lead (regardless of tier) a "thanks, we got your message" email. All 3 branches converge here. |
| 10 | Respond Success | Respond to Webhook | Returns a 200 JSON response confirming processing + the assigned priority. |
| 11 | Respond Error | Respond to Webhook | Returns a 400 JSON response listing what was missing/invalid. |

## 3. Required Fields

| Field | Type | Required |
|-------|------|----------|
| `name` | string | Yes |
| `email` | string (valid email) | Yes |
| `company` | string | Yes |
| `service` | string | Yes |
| `budget` | number | Yes |

## 4. Lead Scoring Logic

This is intentionally simple and rule-based (no AI call needed) — it's fast, free, and predictable.

**Budget**
- ≥ $10,000 → +3
- $5,000–$9,999 → +2
- $1,000–$4,999 → +1
- < $1,000 → +0

**Service type** (edit the `highValueServices` list in the Code node to match your actual offerings)
- Matches a high-value keyword (e.g. "AI agent", "enterprise") → +2
- Any other non-empty service → +1

**Company provided**
- Non-empty company name → +1

**Final tier**
- Score ≥ 5 → **High**
- Score 3–4 → **Medium**
- Score < 3 → **Low**

## 5. Setup Instructions

1. **Import the workflow**: In n8n, go to *Workflows → Import from File* and select `Lead-Management-System.json`.
2. **Set up credentials** (never hardcode these — always use n8n's Credentials manager):
   - Add a **Gmail OAuth2** credential and attach it to both Gmail nodes (`Notify Sales - High Priority`, `Send Acknowledgment Email`).
   - Add a **Google Sheets OAuth2** credential and attach it to both Sheets nodes.
3. **Replace placeholders**:
   - `PLACEHOLDER_SALES_EMAIL@yourcompany.com` → your real sales inbox.
   - `PLACEHOLDER_SPREADSHEET_ID` → the ID of your Google Sheet (found in its URL).
   - Make sure your sheet has two tabs named exactly `Leads` and `Nurture List`, each with header row: `Name | Email | Company | Service | Budget | Lead Score | Priority | Date Received`.
4. **Activate the workflow.** Copy the webhook's production URL from the Webhook node.
5. **Test it** using the sample request in §6 below.

## 6. Sample API Request

**Request**

```bash
curl -X POST https://your-n8n-instance.com/webhook/lead-intake \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sarah Ahmed",
    "email": "sarah@brightpath.io",
    "company": "BrightPath Consulting",
    "service": "AI agent development",
    "budget": 12000
  }'
```

**Successful Response (200)**

```json
{
  "status": "success",
  "message": "Lead received and processed",
  "priority": "High",
  "leadScore": 6
}
```

**Validation Failure Response (400)**

```json
{
  "status": "error",
  "message": "Validation failed",
  "errors": [
    "Missing required field: company",
    "Invalid email format"
  ]
}
```

## 7. Error Handling & Reliability Notes

- **Validation happens before anything else** — bad data never reaches scoring, notifications, or the database.
- **Gmail/Sheets nodes should have "Continue on Fail" reviewed**: for a production client, wrap the notification branches in n8n's built-in retry (node-level "Retry On Fail", 2–3 attempts) so a transient Gmail/Sheets API hiccup doesn't silently drop a lead.
- **Recommended addition for production**: an Error Trigger workflow that logs any failed execution to a Slack channel or a "Failed Leads" sheet, so nothing is lost silently.
- **Rate limits**: Gmail API and Google Sheets API both have per-minute quotas. For high lead volume, consider batching Sheets writes or adding a Wait node between rapid-fire requests.

## 8. Security Notes

- No credentials are stored in this JSON — all Gmail/Sheets auth is handled through n8n's Credentials manager and referenced by ID only.
- Consider adding a shared-secret header check (e.g. `X-API-Key`) in the Validate node if this webhook will be public, to prevent spam submissions.
- Sanitize/escape any lead-supplied text before it's inserted into emails if you extend this to render lead-provided HTML.
