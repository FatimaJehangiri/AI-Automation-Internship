## 🎯 Objective

Build the pipeline that turns raw documents into searchable vectors — the ingestion half of a Retrieval Augmented Generation (RAG) system.

## 🧩 Pipeline

```
Document → Extract Content → Clean Content → Split into Chunks → Generate Embeddings → Store in Vector DB
```

| Stage | n8n Node | What it does |
|---|---|---|
| Trigger | `Run Ingestion` | Manual trigger to kick off a batch ingestion run |
| Extract | `Load Knowledge Entries` | Loads 12 knowledge-base entries sourced from FAQ, help, docs, glossary, and policy files |
| Clean | `Clean Content` | Strips control characters, normalises whitespace, adds a char count for QA |
| Split | `Split into Chunks` | Recursive character text splitter — 500 char chunks, 50 char overlap |
| Embed | `Gemini Embeddings` | Google Gemini embedding model, 3072-dimension vectors |
| Load | `Document Loader` | Maps each chunk's clean text to the embedding call and attaches metadata (`id`, `title`, `category`, `source`) |
| Store | `Store in Supabase` | Inserts each chunk + embedding + metadata into a Postgres/pgvector table |

## 🗂️ Source Documents

12 knowledge-base entries across 5 source files, standing in for real FAQ/help/docs/policy content:

| File | Entries |
|---|---|
| `faq.md` | Company info, business hours, free trial, support contact |
| `help & policy.txt` | Password reset instructions |
| `docs.md` | Supported vector DBs, how chunking works, integrations |
| `glossary.txt` | What are embeddings, what is RAG |

## 🛠️ Tech Stack

- **n8n** — workflow orchestration (LangChain nodes)
- **Supabase** — Postgres + `pgvector` extension as the vector store
- **Google Gemini** — embedding generation
- **JavaScript (Code nodes)** — content loading and cleaning logic

## ⚙️ Setup

1. Enable the `pgvector` extension in Supabase and create a `documents` table (`id`, `content`, `metadata jsonb`, `embedding vector(3072)`) plus a `match_documents` SQL function for similarity search.
2. Import `Day20_Document_Ingestion_Embeddings.json` into n8n.
3. Connect a Supabase credential (Project URL + **service_role** key — not the anon key) and a Google Gemini API credential.
4. Run the workflow via the `Run Ingestion` trigger.
5. Verify rows in Supabase → Table Editor → `documents`.

Full step-by-step instructions, SQL, and troubleshooting are in `Day20_Setup_Guide.md`.

## ✅ Result

- 12/12 knowledge entries processed end-to-end (load → clean → chunk → embed → store) with no errors
- Rows confirmed in the Supabase `documents` table with populated `content`, `metadata`, and `embedding` columns

## 📌 Key Learnings

- n8n's LangChain vector store nodes handle splitting, embedding, and document loading as **sub-node connections** (`ai_textSplitter`, `ai_embedding`, `ai_document`) into the vector store node, not as sequential main-line steps.
- The embedding model's output dimension must exactly match the vector column size in the database — a mismatch (e.g. 768 vs 3072) fails inserts even when everything else is configured correctly.
- Supabase inserts need the **service_role** key, not the anon/public key, since RLS otherwise blocks writes.

## 📁 Repo Contents

```
├── Day20_Document_Ingestion_Embeddings.json   # n8n workflow
├── README.md                                  # this file
├── faq.md
├── help.txt
├── docs.md
├── glossary.txt
├── policy.pdf
└── screenshots/
    ├── supabase-table-rows.png
    └── workflow-execution.png
