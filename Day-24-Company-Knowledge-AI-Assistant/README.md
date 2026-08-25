# Company Knowledge AI Assistant

A Retrieval-Augmented Generation (RAG) assistant that answers employee and customer questions about TechNova Solutions using a company knowledge base, built entirely in n8n.

## Objective

Deliver a complete RAG/agent assistant over real company knowledge: ingest company documents into a vector database, then answer natural-language questions with retrieval + generation, grounded strictly in the ingested content.

---

## Architecture

```
Company Knowledge → Ingestion → Embeddings → Vector DB
                                                  │
User Question → RAG / Agent → Retrieve ──────────┘
                     │
                  Generate
                     │
                Return Response (with source citation)
```

The system is two independent n8n workflows connected through a shared Supabase vector table:

| Workflow | Role | Trigger |
|---|---|---|
| **1. Knowledge Ingestion** | Chunks, embeds, and stores company knowledge in Supabase | Manual (run once, or whenever the knowledge base changes) |
| **2. RAG Assistant** | Embeds the user's question, retrieves relevant chunks, generates a grounded answer | Chat message |

---

## Tech Stack

- **Orchestration:** n8n
- **Embeddings:** Cohere (`embed-english-v3.0`)
- **LLM:** Groq (`llama-3.3-70b-versatile`)
- **Vector Database:** Supabase (pgvector)
- **Memory:** n8n Buffer Window Memory (conversation context across turns)

---

## Knowledge Base

22 knowledge entries covering:

- **General** — company overview, mission
- **HR** — working hours, remote work, leave policy, annual/sick leave, training
- **Finance** — salary payments, expense reimbursement
- **Security** — information security, password policy, customer data handling
- **Services** — AI automation, software development, cloud services
- **Customer Support** — response times, urgent escalation
- **Projects** — client communication, scope change process
- **IT** — data backup, employee equipment

Source file: `knowledge_base.json`

---

## Setup Instructions

### 1. Supabase

Enable the `vector` extension, create the `company_documents` table (1024 dimensions for Cohere embeddings), and create the `match_company_documents` SQL function used for similarity search.

### 2. Credentials (in n8n)

- Cohere API key (embeddings)
- Groq API key (LLM)
- Supabase API credentials (Project URL + service key)

### 3. Import Workflows

1. Import `Day24_Company_Knowledge_Ingestion_FIXED.json` → set Cohere and Supabase credentials → run once via Manual Trigger to populate the vector table.
2. Import `Day24_Company_Knowledge_RAG_Assistant.json` → set Cohere, Groq, and Supabase credentials → open the chat panel to start asking questions.

---

## Guardrails

- The agent **always searches the knowledge base first** before answering.
- If no relevant match is found, it returns a fixed fallback response instead of guessing:
  > "I don't have that information in the company knowledge base. Please check with the relevant department or your manager."
- Every grounded answer cites its source category and document title.

## Deliverables

- [x] Workflow 1 JSON — Knowledge Ingestion
- [x] Workflow 2 JSON — RAG Assistant
- [x] Knowledge source file
- [x] 10 example queries with results
- [x] README (this file)
- [x] Demo video (2–5 min)

---
