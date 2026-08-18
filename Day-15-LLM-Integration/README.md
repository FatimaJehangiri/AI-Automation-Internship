# Day 15 — AI Email Classification Workflow

**AI Automation Internship Program**
**Module:** LLM Integration inside n8n
**Author:** Fatima Wajid

---

## 1. Objective

Wire a large language model into an n8n workflow and route downstream logic based on the model's output. This workflow receives an incoming email, uses an LLM (via Groq) to classify it into one of six predefined categories, and routes it to a category-specific action using a Switch node.

---

## 2. What This Workflow Does

A new email arrives → an LLM reads the subject and body → the LLM returns a single category → the workflow routes the email down a different path depending on that category → the result is logged and a response is returned.

This replaces manual, rule-based email sorting (e.g. keyword matching) with a model that understands intent and context, the same way a human reading the inbox would.

**Categories:** Sales · Support · Complaint · Invoice · Spam · General

---

## 3. Architecture

```
New Email (Webhook)
        │
        ▼
Normalize Email Data (Set)
        │
        ▼
Classify Email (LLM Chain) ◄── Groq Chat Model (sub-node, ai_languageModel connection)
        │
        ▼
Validate & Extract Category (Code)
        │
        ▼
Route by Category (Switch)
   ├── Sales Route
   ├── Support Route
   ├── Complaint Route
   ├── Invoice Route
   ├── Spam Route
   └── General Route
        │  (all branches converge)
        ▼
Log Classification (Google Sheets)
        │
        ▼
Send Response (Respond to Webhook)
```

---

## 4. Tech Stack

| Component | Tool |
|---|---|
| Automation platform | n8n |
| LLM provider | Groq (`openai/gpt-oss-120b`) |
| LLM node pattern | Basic LLM Chain + Chat Model sub-node (LangChain cluster nodes) |
| Trigger | Webhook (simulates "New Email") |
| Logging | Google Sheets (OAuth2) |
| Testing | Postman |

---

## 5. Concept: Basic LLM Chain vs. AI Agent

| | Basic LLM Chain | AI Agent |
|---|---|---|
| Behavior | Single prompt in → single response out | Reasons in a loop, can call tools mid-task |
| Analogy | A cashier taking one order | A manager who checks inventory, calls a supplier, then responds |
| Used here because | Classification is a single judgment call — no external lookups or multi-step reasoning required | N/A for this task |

---

## 6. Node-by-Node Breakdown

| # | Node | Type | Purpose |
|---|---|---|---|
| 1 | New Email (Webhook) | `n8n-nodes-base.webhook` | Entry point; receives `from`, `subject`, `body` as JSON |
| 2 | Normalize Email Data | `n8n-nodes-base.set` | Standardizes field names regardless of incoming payload shape |
| 3 | Groq Chat Model | `@n8n/n8n-nodes-langchain.lmChatGroq` | Language model sub-node; connects to the chain via `ai_languageModel` |
| 4 | Classify Email (LLM Chain) | `@n8n/n8n-nodes-langchain.chainLlm` | Sends the prompt + email data to Groq, returns the classification text |
| 5 | Validate & Extract Category | `n8n-nodes-base.code` | Parses the model's raw text output, validates it against the allowed category list, falls back to `General` on any anomaly |
| 6 | Route by Category | `n8n-nodes-base.switch` | Routes the item down one of 6 branches based on `category` |
| 7–12 | [Category] Route | `n8n-nodes-base.set` | Assigns the downstream action for that category (ticket creation, CRM forward, escalation, etc.) |
| 13 | Log Classification | `n8n-nodes-base.googleSheets` | Appends the result to a tracking sheet |
| 14 | Send Response | `n8n-nodes-base.respondToWebhook` | Returns the classification result as the webhook's HTTP response |

---

## 7. Prompt Used

```
You are an email classification assistant for a business inbox.

Classify the following email into EXACTLY ONE of these categories:
- Sales
- Support
- Complaint
- Invoice
- Spam
- General

Rules:
- Respond with ONLY the category name, nothing else. No punctuation, no explanation, no extra words.
- If the email is a purchase inquiry, quote request, or pricing question, use "Sales".
- If the email is a technical question or help request about an existing product/service, use "Support".
- If the email expresses dissatisfaction, frustration, or a formal grievance, use "Complaint".
- If the email is about billing, payments, or invoices, use "Invoice".
- If the email looks like an unsolicited promotion, phishing attempt, or irrelevant bulk email, use "Spam".
- If it doesn't clearly fit any category above, use "General".

Email details:
From: {{ $json.from }}
Subject: {{ $json.subject }}
Body: {{ $json.body }}

Category:
```

**Model configuration:**
- Model: `openai/gpt-oss-120b`
- Sampling Temperature: `0.0` — deterministic output; the same email always classifies the same way, which matters for a decision-routing task
- Maximum Number of Tokens: `150` — see Section 9 for why this value matters specifically for this model family

---

## 8. Setup Instructions

1. **Import** `Day15_AI_Email_Classifier.json` into n8n via *Workflows → Import from File*.
2. **Groq Chat Model node** — attach your Groq API credential (Bearer auth, handled automatically by n8n's credential type).
3. **Log Classification node** — attach your Google Sheets OAuth2 credential and point it at a sheet with header row: `from | subject | category | action | receivedAt`.
4. **Activate** the workflow to enable the webhook listener.
5. **Test** using the payloads in Section 10 via Postman, POSTing to the webhook URL.
6. Confirm results in the **Executions** tab, then in your Google Sheet.

---

## 9. Debugging Log — Issues Found & Fixed

Documenting these because they reflect real production failure modes, not just workflow-building steps.

**Issue 1 — Output field mismatch**
The Basic LLM Chain node's output shape differs across n8n versions (`{ text }` vs. `{ response: { text } }`). The Code node was updated to check all known shapes defensively rather than assuming one.

**Issue 2 — Deprecated model ID**
The workflow originally used `llama-3.3-70b-versatile`, which Groq deprecated and shut down on 08/16/2026 (announced 06/17/2026). Requests to a deprecated model ID return errors; the n8n node surfaced this as an empty `text` output rather than a visible error, which made it look like a logic bug rather than a config/infra issue. **Fix:** migrated to `openai/gpt-oss-120b`, Groq's recommended replacement.

**Issue 3 — Token limit too low for a reasoning model**
`openai/gpt-oss-120b` is a reasoning model — it spends tokens on internal chain-of-thought before producing the final answer. With `Maximum Number of Tokens` set to `10`, the model exhausted its budget mid-reasoning (`finish_reason: "length"`) and never wrote the category word. **Fix:** raised the token limit to `150`, giving the model room to reason and still return a clean one-word answer.

**Takeaway:** non-reasoning models (Llama 3.x) and reasoning models (GPT-OSS, DeepSeek-R1, o1-style) have different token economics for the same task. A limit that's safe for one can silently starve the other.

---

## 10. Test Payloads & Expected Results

| Payload | Expected Category |
|---|---|
| `{"from": "priya@retailcorp.com", "subject": "Bulk pricing inquiry", "body": "Hi, we're interested in ordering 200 units. Could you send a quote?"}` | Sales |
| `{"from": "angryuser@gmail.com", "subject": "Extremely disappointed", "body": "This is the third time my order has arrived damaged. I want a refund immediately."}` | Complaint |
| `{"from": "winbig@lottery-promo.xyz", "subject": "YOU WON $1,000,000!!!", "body": "Click here now to claim your prize before it expires!!!"}` | Spam |

---

## 11. Error Handling & Production Recommendations

- **Retry on Fail** should be enabled on the Groq Chat Model node (Node Settings) to absorb transient rate-limit or timeout errors.
- **Fallback category:** the Code node defaults to `General` on any unparseable or unexpected model output, ensuring the Switch node never breaks the workflow.
- **Switch fallback output:** set to `General` as a second safety net.
- **Credentials:** stored exclusively in n8n's Credentials manager — never hardcoded in node parameters, per program standard.
- **Model pinning:** given Issue 2 above, model IDs should be reviewed periodically against the provider's deprecation page rather than assumed stable indefinitely.

---

## 12. Deliverables Checklist

- [x] Workflow JSON — `Day15_AI_Email_Classifier.json`
- [x] Prompt used — Section 7
- [ ] Screenshots of at least 3 different classifications (Sales / Complaint / Spam) — attach after test run
- [x] README (this document)
