# Day 26 — Error Handling & Reliability

## Problem Statement
Real-world APIs fail: they time out, return `429` rate-limit responses, go down entirely, or send back malformed data. A workflow with no error handling simply stops silently, leaving failures undetected and unresolved until someone notices missing data downstream.

---

## Objective
To harden an existing n8n workflow so it can survive API failures gracefully — by retrying transient errors automatically, branching cleanly on success vs. failure, logging every failure to a database, and alerting an admin — using a dedicated, reusable **Error Workflow**.

---

## Workflow Architecture

**Main Workflow**
```
Manual Trigger
      ↓
API Request (with Retry + Error Output)
      ↓                          ↓
  Success                    Failure
      ↓                          ↓
Continue                   Log Error
                                 ↓
                      Log Error to Supabase
                                 ↓
                          Notify Admin
```

**Error Workflow** (global safety net — catches failures the main workflow's own branching doesn't, e.g. crashes, trigger failures, unhandled exceptions)
```
Error Trigger
      ↓
Format Error Details
      ↓
Log Error to Supabase
      ↓
Notify Admin
```

---

## Technologies Used
- **n8n** — workflow automation platform
- **Supabase (PostgreSQL)** — error log storage
- **Email** — admin alerting (Slack or Telegram can substitute)
- **httpstat.us** — public test API used to deliberately simulate failures (e.g. `/500`, `/429`)

---

## Nodes Used

### Main Workflow
| Node Name | Type | Purpose |
|---|---|---|
| Manual Trigger | Trigger | Starts the workflow on demand |
| API Request | HTTP Request | Calls the target API; configured with retry + error output |
| Continue - Success | NoOp | Placeholder for the normal success path |
| Log Error | Edit Fields (Set) | Structures the error details for logging |
| Log Error to Supabase | Postgres | Inserts the failure record into `error_logs` |
| Notify Admin | Send Email | Alerts the admin of the failure |

### Error Workflow
| Node Name | Type | Purpose |
|---|---|---|
| Error Trigger | Error Trigger | Fires automatically when any linked workflow fails |
| Format Error Details | Edit Fields (Set) | Extracts workflow name, failed node, error message, execution ID |
| Log Error to Supabase | Postgres | Inserts the failure record into `error_logs` |
| Notify Admin | Send Email | Alerts the admin of the failure |

---

## Setup Instructions

### 1. Create the error log table in Supabase
Run in Supabase **SQL Editor**:
```sql
CREATE TABLE IF NOT EXISTS error_logs (
    log_id BIGSERIAL PRIMARY KEY,
    workflow_name VARCHAR(255),
    failed_node VARCHAR(255),
    error_message TEXT,
    status_code VARCHAR(50),
    logged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 2. Import both workflows into n8n
- Go to **n8n → Workflows → Import from File**.
- Import `day26_main_workflow.json`.
- Import `day26_error_workflow.json`.

### 3. Reassign credentials
n8n does not export credential values (by design). After importing:
- Open each **Postgres** node → select your existing **Postgres account** credential (same one from Day 25).
- Open each **Send Email** node → select or create an **SMTP account** credential (or swap the node for Slack/Telegram if you prefer).

### 4. Link the error workflow to the main workflow
- Open the **main workflow** (Day 26 - Error Handling Demo) → **Settings** (top-right menu) → **Error Workflow** → select **"Day 26 - Error Workflow (Logger + Notifier)"**.
- Save.

This ensures that even failures the main workflow's own branching doesn't catch (e.g. the trigger itself failing) still get logged and alerted.

### 5. Activate the error workflow
- Open the error workflow and toggle it **Active** (top-right switch). It must be active to catch errors from other workflows automatically.

---

## Credentials Required
- **Postgres account** — Supabase PostgreSQL connection (reused from Day 25)
---

## Workflow Explanation
1. **API Request** calls `https://httpstat.us/500`, a public test endpoint that deliberately returns a server error — this simulates a real API failure for testing.
2. The node is configured with **Retry On Fail** (3 attempts, 2-second gaps) to absorb transient/temporary failures automatically.
3. If all retries are exhausted, the node's **"On Error" behavior is set to "Continue Using Error Output"**, so instead of crashing the workflow, it routes the failure down a second, separate output branch.
4. The **success branch** continues normally; the **failure branch** flows into **Log Error**, which extracts a clean error message, node name, and timestamp.
5. **Log Error to Supabase** inserts that record into the `error_logs` table for a permanent audit trail.
6. **Notify Admin** sends an email summarizing the failure.
7. Separately, the **Error Workflow** acts as a global backstop: if any workflow linked to it fails in a way the workflow's own logic doesn't catch (e.g. the trigger itself errors, or an unrelated node throws unexpectedly), the **Error Trigger** fires automatically, and the same log + notify pattern runs.

---

## Test Cases
| Test Case | Action | Expected Result |
|---|---|---|
| TC1 | Run workflow against a working URL (e.g. `https://httpstat.us/200`) | Success branch executes; no error is logged |
| TC2 | Run workflow against `https://httpstat.us/500` | Retries occur 3 times, then the failure branch fires; a row appears in `error_logs`; an email is sent |
| TC3 | Run workflow against `https://httpstat.us/429` | Simulates rate limiting; same failure branch and logging occurs |
| TC4 | Manually throw an error inside an unrelated test workflow linked to the Error Workflow | Error Trigger fires; failure is logged and alerted without any error-handling logic in that workflow itself |
| TC5 | Check Supabase after a forced failure | `error_logs` table contains a new row with correct workflow name, node, message, and timestamp |
| TC6 | Check inbox/Slack after a forced failure | Admin alert is received with matching failure details |

---

## Error Handling Strategy
- **Retries**: Transient errors (timeouts, brief API blips) are retried automatically (3 attempts, 2s apart) before being treated as a true failure.
- **Error output branching**: The HTTP Request node's "Continue Using Error Output" setting prevents a single failed API call from crashing the entire execution — success and failure paths are handled explicitly.
- **Centralized logging**: All failures, whether caught locally or globally, are written to the same `error_logs` table for a single audit trail.
- **Admin notification**: Every logged failure triggers an email so issues are surfaced immediately rather than discovered later.
- **Global safety net**: The Error Workflow setting ensures failures outside the main workflow's own try-catch logic (e.g. crashes in the trigger or in unrelated workflows) are still caught.

---

## Known Limitations
- Retries are fixed at 3 attempts with a flat 2-second delay — no exponential backoff yet.
- Rate-limit (`429`) responses are treated the same as generic failures; there's no logic to read a `Retry-After` header and pause accordingly.
- Email is used for alerting; there's no escalation (e.g. repeated failures within an hour triggering a more urgent channel).
- No deduplication — if the same API fails repeatedly, one alert is sent per failure rather than being batched.
- The error workflow assumes all connected workflows use the same `error_logs` schema; a workflow producing a different error shape may log incomplete fields.

---

## Future Improvements
- Implement **exponential backoff** for retries instead of a fixed delay.
- Add logic to detect `429` responses specifically and pause based on the API's `Retry-After` header.
- Add **Slack/Telegram** as a redundant notification channel alongside email.
- Add a **dashboard** (e.g. a simple Supabase view or external tool) to visualize failure trends over time.
- Add **alert throttling/deduplication** so repeated identical failures don't spam the admin.
- Extend the Error Workflow to tag severity levels (warning vs. critical) based on the failing node or workflow.

---

## Deliverables Checklist
- [x] **Workflow JSON** — `day26_main_workflow.json`
- [x] **Error Workflow JSON** — `day26_error_workflow.json`
- [ ] **Screenshot of a logged/alerted failure** — after running the workflow against `https://httpstat.us/500`:
  1. Screenshot the n8n execution showing the failure branch firing (red/error indicator on the API Request node, success on the downstream Log Error path).
  2. Screenshot the new row in Supabase's `error_logs` table.
  3. Screenshot the received admin alert email.
