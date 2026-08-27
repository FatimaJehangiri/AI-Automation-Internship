# Day 25 — Database CRUD Operations (n8n + Supabase)
---
## Problem Statement
Support and helpdesk systems need a reliable way to log, retrieve, update, and remove customer tickets from a central database. Manually managing this data is slow and error-prone, and there is no automated pipeline connecting a workflow automation tool to a persistent database for ticket tracking.

---

## Objective
To build an automated workflow using **n8n** that performs all four fundamental database operations — **Create, Read, Update, Delete (CRUD)** — on a `tickets` table hosted in a **Supabase PostgreSQL** database, demonstrating a complete, secure, and testable data pipeline.

---

## Workflow Architecture

```
Manual Trigger
      ↓
Edit Fields (Create Sample Ticket)
      ↓
Postgres – INSERT Ticket
      ↓
Postgres – READ Tickets
      ↓
Postgres – UPDATE Ticket Status
      ↓
Postgres – DELETE Test Ticket
```

Data flows from a manually triggered n8n workflow, through a field-mapping step, into a Supabase PostgreSQL table via the Postgres node, with each subsequent node performing one CRUD operation in sequence and passing results downstream for verification.

---

## Technologies Used
- **n8n** — workflow automation platform
- **Supabase** — hosted PostgreSQL database
- **PostgreSQL** — relational database engine
- **Supabase Connection Pooler (Supavisor)** — IPv4-compatible pooled DB connection

---

## Nodes Used
| Node Name | Type | Purpose |
|---|---|---|
| Manual Trigger | Trigger | Starts the workflow on demand |
| Create Sample Ticket | Edit Fields (Set) | Defines sample ticket data to insert |
| INSERT Ticket | Postgres (Insert) | Adds a new row to the `tickets` table |
| READ Tickets | Postgres (Select) | Retrieves all tickets from the table |
| UPDATE Ticket Status | Postgres (Update) | Changes a ticket's `id` field |
| DELETE Test Ticket | Postgres (Delete) | Removes a dedicated test row |

---

## Setup Instructions

### 1. Create the `tickets` table in Supabase
Run the following in Supabase's **SQL Editor**:
```sql
CREATE TABLE IF NOT EXISTS tickets (
    ticket_id BIGSERIAL PRIMARY KEY,
    customer_name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    category VARCHAR(100),
    priority VARCHAR(50),
    status VARCHAR(50) DEFAULT 'Open',
    ai_summary TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 2. Get Supabase connection details
From **Project Settings → Database → Connection Pooling**, note the pooler host, port, database name, and username (direct connections may fail with `ENETUNREACH` on IPv6-only networks — use the pooler instead).

### 3. Create the Postgres credential in n8n
In n8n, go to **Credentials → Add Credential → Postgres**, and enter the connection details from Supabase (host, database, user, password, port, SSL mode). Save and test the connection.

### 4. Build the workflow
Add the nodes listed above in sequence, connect them, and assign the saved credential to every Postgres node.

### 5. Execute and verify
Run the workflow manually and check the **Supabase Table Editor** after each step to confirm the expected row changes.

---

## Credentials Required
- **Postgres account** — Supabase PostgreSQL connection credential used by all Postgres nodes in this workflow

---

## Workflow Explanation
1. **Manual Trigger** starts the workflow for testing purposes.
2. **Create Sample Ticket** defines the ticket fields (`customer_name`, `email`, `category`, `priority`, `status`, `ai_summary`) to be inserted.
3. **INSERT Ticket** writes this data into the `tickets` table; `ticket_id` and `created_at` are generated automatically by PostgreSQL.
4. **READ Tickets** retrieves all rows to confirm the insert succeeded.
5. **UPDATE Ticket Status** changes the `status` field of a specific ticket (filtered by `ticket_id`) from `Open` to `In Progress`.
6. **READ Updated Ticket** re-fetches the same ticket to visually confirm the update.
7. **DELETE Test Ticket** removes a separate, dedicated test row (filtered by `customer_name = 'DELETE TEST'`) so the original sample ticket is never destructively removed.

---

## Test Cases
| Test Case | Action | Expected Result |
|---|---|---|
| TC1 | Insert a new ticket with valid fields | New row appears in `tickets` with an auto-generated `ticket_id` |
| TC2 | Read all tickets after insert | Newly inserted ticket is present in the result set |
| TC3 | Update `status` of an existing `ticket_id` | Ticket's `status` changes from `Open` to `In Progress` |
| TC4 | Read the same `ticket_id` after update | Returned row reflects the new `status` value |
| TC5 | Insert a dedicated test row, then delete it by `customer_name` | Test row insert succeeds, then delete removes only that row |
| TC6 | Attempt update/delete with a non-existent `ticket_id` | No rows affected; workflow does not error unexpectedly |

---

## Error Handling
- **`ENETUNREACH` connection errors**: Caused by Supabase's direct database host requiring IPv6, which many hosting environments cannot route. Resolved by switching the Postgres credential to Supabase's **Connection Pooler** host (IPv4-compatible).
- **Empty/greyed-out Table dropdown with warning icon**: Indicates the table list failed to load — usually due to Row Level Security (RLS) restrictions, incorrect schema selection, or insufficient database user permissions. Resolved by confirming the table exists in the `public` schema, adjusting RLS policies, or manually entering the table name.
- **Field mismatch errors**: The Postgres node requires incoming field names to exactly match the destination column names; an Edit Fields node is used upstream to guarantee this.
- **Missing filter on Update/Delete**: Filters (e.g. `ticket_id`, `customer_name`) are explicitly required before executing Update or Delete operations to avoid unintended changes to unrelated rows.

---

## Known Limitations
- The workflow is triggered manually and is not yet event-driven (e.g. no webhook trigger from a support form or chat widget).
- No automated rollback or transaction handling exists if a step fails mid-sequence.
- The `ai_summary` field is currently populated manually rather than generated by an AI model.
- No input validation (e.g. email format, required fields) is enforced before insertion.
- Row Level Security is not configured for production-grade access control.

---

## Future Improvements
- Replace the Manual Trigger with a **Webhook Trigger** connected to a support form or chatbot for real-time ticket creation.
- Integrate an AI node (e.g. an LLM call) to auto-generate the `ai_summary` field from raw customer messages.
- Add proper **Row Level Security (RLS)** policies in Supabase for production use.
- Add validation and error-branching logic (e.g. IF nodes) to handle failed inserts/updates gracefully.
- Extend the workflow to support ticket assignment, notifications (email/Slack), and SLA tracking.
- Wrap the CRUD operations in a reusable **sub-workflow** so they can be called from multiple parent workflows.

---
