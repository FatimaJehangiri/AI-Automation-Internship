# Day 10 — Webhook Form Intake Automation

**Intern:** Fatima Wajid
**Program:** AI Automation Internship — Petalnex Pvt. Ltd.
**Module:** Day 10 — Webhooks / Event-Driven Workflows
**Workflow file:** `Day10-Webhook-Form-Intake.json`

---

## 1. Objective

Trigger a workflow the instant a form is submitted, instead of checking on a schedule. This replaces polling-based automation (Day 8's Schedule Trigger) with an event-driven pattern.

## 2. What This Workflow Does

A form submission (simulated via Postman) is sent as a POST request to an n8n Webhook URL. The workflow:

1. Receives the submission
2. Validates the required fields
3. If valid → saves the record and emails a confirmation to the submitter
4. If invalid → returns an error response listing what's missing

## 3. Architecture

```
Website/Form (Postman)
        |  POST
        v
  Webhook Node
        |
  Validate Submission (Code node)
        |
      IF Valid?
    /          \
  Yes            No
   |              |
Save to        Respond:
Google Sheets   400 error
   |
Send Confirmation
Email (Gmail)
   |
Respond: 200 success
```

## 4. Node-by-Node Breakdown

| Node | Type | Purpose |
|---|---|---|
| Webhook - Form Submitted | Webhook | Listens for POST requests at a unique, secret path. Entry point of the workflow. |
| Validate Submission | Code | Reads `name`, `email`, `message` from the request body. Checks presence and validates email format with a regex. Outputs `isValid` and an `errors` array. |
| IF Valid? | IF | Branches the workflow based on `isValid`. |
| Save to Google Sheets | Google Sheets | Appends a new row (Name, Email, Message, SubmittedAt) to the tracking sheet. Runs only on the valid branch. |
| Send Confirmation Email | Gmail | Sends an acknowledgment email to the address the user submitted. Runs only after a successful save. |
| Respond - Success | Respond to Webhook | Returns `200 OK` with a success message back to the sender. |
| Respond - Validation Error | Respond to Webhook | Returns `400 Bad Request` with the specific validation errors. |

## 5. Security Notes

- **Secret path:** The webhook path (`intake-form`) is a non-obvious string rather than something guessable like `/webhook/form`, reducing the chance of unsolicited requests reaching the workflow.
- **No hardcoded credentials:** Google Sheets and Gmail both authenticate through n8n's built-in **Credentials manager**. No API keys, tokens, or passwords appear anywhere in the workflow JSON — credential nodes reference stored credential IDs only.
- **Input validation before any side effect:** No data is written and no email is sent until the submission passes validation, preventing malformed or malicious input from reaching storage or the notification channel.
- **Recommended next step (not yet implemented):** Add Header Auth on the Webhook node to require a shared secret header, so only trusted senders (e.g. the real website form) can trigger the workflow at all.

## 6. Setup Instructions

1. Import `Day10-Webhook-Form-Intake.json` into n8n via **Workflows → Import from File**.
2. Create a Google Sheet with column headers: `Name`, `Email`, `Message`, `SubmittedAt`.
3. Open the **Save to Google Sheets** node and replace `PLACEHOLDER_GOOGLE_SHEET_ID` with the real sheet ID (found in the sheet's URL).
4. Reconnect credentials on both the **Save to Google Sheets** and **Send Confirmation Email** nodes using n8n's Credentials manager (OAuth2 sign-in — nothing is typed or pasted directly into the workflow).
5. Activate the workflow to generate the **Production URL** on the Webhook node.

## 7. Testing

**Tool used:** Postman
**Method:** POST
**Endpoint:** Production Webhook URL from step 5 above

**Sample valid request body:**
```json
{
  "name": "Ayesha Khan",
  "email": "ayesha.khan@example.com",
  "message": "Hi, I'd love to learn more about your automation services."
}
```

**Expected success response (200):**
```json
{
  "status": "success",
  "message": "Thanks! Your submission was received."
}
```

**Expected error response (400):**
```json
{
  "status": "error",
  "errors": ["email is required"]
}
```

## 8. Deliverables Checklist

- [x] Workflow JSON — `Day10-Webhook-Form-Intake.json`
- [ ] Screenshot: sample POST request + response in Postman
- [ ] Screenshot: new row saved in Google Sheets
- [ ] Screenshot: confirmation email received
- [x] README — this document

## 9. Key Takeaways

- Webhooks turn workflows from time-based (polling) to event-based (instant reaction), which is more efficient and scales better for real client use cases.
- Validating input *before* it reaches storage or a notification step is a core production pattern — it prevents bad data from propagating downstream.
- Credential handling followed the same rule established in Day 8: all authentication lives in n8n's Credentials manager, never hardcoded in node parameters or the exported JSON.
