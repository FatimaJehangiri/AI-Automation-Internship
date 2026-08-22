## Day 21 --- HR Policy RAG Assistant
## Task: Build a Retrieval-Augmented Generation (RAG) assistant that 
answers HR policy questions strictly from a private knowledge base,
provides a safe fallback when information is unavailable, and cites the
relevant policy section.

## 1. Project Overview

The HR Policy RAG Assistant is an n8n-based Retrieval-Augmented
Generation workflow designed to answer employee questions from a
controlled HR policy knowledge base.

The assistant covers:

Leave Policy

Working Hours

Attendance Policy

Internship Guidelines

## Code of Conduct

The assistant is intentionally grounded: it is instructed to use only
the information retrieved from the HR policy knowledge base. When the
requested information is not available, it returns a predefined fallback
instead of guessing.

## Core objective

Answer HR policy questions using retrieved company policy content,
avoid invented information, provide the relevant source section, and
safely handle unknown questions.

This workflow uses n8n's LangChain nodes for document preparation,
embeddings, vector retrieval, and the final Retrieval QA chain.

## 2. Architecture Overview

The workflow contains two logical paths that use the same in-memory
vector store under the key hr_policy_kb.

Path A --- Knowledge Base Indexing

This path is used initially and whenever the policy content is changed.

Manual Trigger
     │
     ▼
HR Policy Document
     │
     ▼
Store in Vector DB (Insert)
     ▲
     │
Prepare Documents (Data Loader)
     ▲
     │
Split Into Chunks
     ▲
     │
Gemini Embeddings

Conceptually, the indexing process is:

HR Policy Text
      ↓
Document preparation
      ↓
Chunking
      ↓
Gemini embeddings
      ↓
In-memory vector store

Path B --- Question Answering

This path is used whenever a user submits a question.

Chat Trigger
     │
     ▼
HR Policy QA Chain
     ▲        ▲
     │        │
Retriever   Gemini Chat Model
     ▲
     │
Vector Store
     ▲
     │
Gemini Embeddings

Conceptually:

User Question
      ↓
Embedding / Similarity Retrieval
      ↓
Top Relevant Policy Chunks
      ↓
Retrieval QA Chain
      ↓
Gemini Chat Model
      ↓
Grounded Answer + Source

The workflow's actual node connections use n8n AI connection types such
as ai_document, ai_textSplitter, ai_embedding, ai_vectorStore,
ai_retriever, and ai_languageModel.

## 3. Node-by-Node Explanation

#                Node              Type              Purpose

1                 Load Knowledge  Manual Trigger    Starts the indexing
Base (Run Once)                   process.

2                 HR Policy       Set / Edit Fields Contains the HR
Document                          policy text in the
text field. The
content is organized
with section headers
such as
[LEAVE POLICY] and
[WORKING HOURS].

3                 Prepare         Default Data      Converts the raw
Documents (Data   Loader            text into LangChain
Loader)                           Document objects and
attaches source
metadata.

4                 Split Into      Recursive         Splits the document
Chunks          Character Text    into smaller chunks
Splitter          so retrieval can
return relevant
portions instead of
the entire handbook.

5                 Embeddings      Google Gemini     Converts document
Model           Embeddings        chunks into vector
representations for
similarity search.
The same embeddings
connection is also
used during
retrieval.

6                 Store in Vector In-Memory Vector  Stores the embedded
DB (Insert)     Store             policy chunks in
memory under
hr_policy_kb.

7                 When Chat       Chat Trigger      Provides the
Message                             built-in n8n chat
Received                          interface for
submitting HR
questions.

8                 Retrieve        In-Memory Vector  Retrieves relevant
Relevant Policy   Store             vectors/documents
Chunks                            from the same
hr_policy_kb
store.

9                 Vector Store    Vector Store      Connects the vector
Retriever       Retriever         store to the QA
chain and requests
the top 4 relevant
chunks (topK: 4).

10                Gemini Chat     Google Gemini     Generates the final
Model           Chat Model        answer using the
retrieved
information.
Temperature is set
to 0 for more
deterministic
responses.

## 4. Knowledge Base

The current knowledge base is stored directly in the HR Policy
Document node.

It contains five policy areas:

Leave Policy

Includes:

Casual leave allowance

Sick leave allowance

Leave application timing

Emergency leave reporting

Internship leave carry-forward rule

Working Hours

Includes:

Standard working hours

Lunch break

Half-day rule

Intern time-tracking requirement

Attendance Policy

Includes:

Biometric/portal attendance

Late check-in window

Unexcused absence rule

Work From Home approval requirement

Internship Guidelines

Includes:

Day-by-day curriculum deliverables

Weekly mentor progress review

Completion certificate conditions

Stipend conditions

Code of Conduct

Includes:

Confidentiality

Harassment/discrimination prohibition

Company software/account usage

Professional communication

The source text and these policy sections are already embedded in the
supplied workflow JSON.

## 5. Chunking Configuration

The Split Into Chunks node uses:

Chunk Size:     500
Chunk Overlap:   50

The purpose of chunking is to divide the policy document into smaller
retrieval units.

The overlap helps preserve information that may cross a chunk boundary.

For this small single-document exercise, the configuration is
intentionally simple and sufficient for demonstrating the RAG pipeline.

## 6. Vector Store Configuration

The workflow uses:

Vector Store: In-Memory Vector Store
Memory Key:   hr_policy_kb
Mode:         Insert / Retrieve

No external vector database is required for this Day 21 implementation.

The same hr_policy_kb key is used by the indexing and retrieval paths.

Important limitation

This is an in-memory vector store. It is intended for the learning
exercise and does not provide the persistence expected from a production
vector database.

If the n8n environment is restarted or the in-memory store is otherwise
cleared, the knowledge base may need to be indexed again.

For a production deployment, a persistent vector database such as
Qdrant, Pinecone, or Supabase/Postgres with pgvector would be a more
appropriate architecture.

## 7. Gemini Configuration

The workflow uses Google Gemini in two roles:

7.1 Embeddings Model

The Embeddings Model converts HR policy chunks into vectors used for
similarity search.

7.2 Chat Model

The Gemini Chat Model generates the final answer from the retrieved
policy context.

The same Google Gemini credential can be selected for both nodes.

## Model compatibility

The imported workflow may contain a model reference that is no longer
available to a particular Gemini account or n8n integration version.

If n8n reports a model availability error, open the Gemini Chat
Model node and select a currently available Gemini Flash model
supported by your n8n version and API account.

For this project, the chat model was updated during testing to Gemini
3.6 Flash after the original gemini-2.5-flash model returned a 404
model-availability error.

Do not hard-code API keys inside the workflow JSON. Store the key in n8n
Credentials and select that credential from the Gemini nodes.

## 8. Grounding and Anti-Hallucination Design

The main grounding logic is implemented in the system prompt of HR
Policy QA Chain.

The prompt enforces four rules:

Use only retrieved policy context.

Do not invent, assume, or guess policy details.

Add a source section to valid answers.

Return the exact fallback when the information is unavailable.

The fallback configured in the workflow is:

Information not available in the knowledge base. Please check with HR directly.

## Why this matters

A normal LLM can answer from its general training knowledge even when a
company-specific policy is unavailable.

This workflow instead supplies retrieved company policy text to the QA
chain and explicitly instructs the model to stay within that
information.

This is the key distinction between:

Generic Chatbot

and:

Grounded RAG Assistant

## Important implementation note

Prompt-based grounding is a control mechanism, not a mathematical
guarantee that an LLM can never hallucinate. The workflow therefore
combines retrieval, a constrained prompt, low temperature, source
conventions, and an explicit fallback to reduce unsupported answers.

## 9. Source Citation Design

The HR policy text contains visible section markers such as:

[LEAVE POLICY]
[WORKING HOURS]
[ATTENDANCE POLICY]
[INTERNSHIP GUIDELINES]
[CODE OF CONDUCT]

The QA prompt instructs Gemini to use the relevant section marker when
citing an answer.

Example:

Employees get 12 casual leave days per year.

Source: [LEAVE POLICY]

This approach keeps the implementation simple: no additional Code node
is required just to turn the section name into a source label.

## 10. Setup Instructions

Step 1 --- Import the workflow

Open your n8n instance.

Create/open a workflow.

Import Day21_HR_Policy_RAG_Assistant.json.

Confirm that the 11 nodes are present.

Save the workflow.

Step 2 --- Configure the Gemini credential

Open:

5. Embeddings Model

and select your Google Gemini credential.

Then open:

10. Gemini Chat Model

and select the same credential.

If the imported JSON contains:

REPLACE_WITH_YOUR_GEMINI_CREDENTIAL_ID

do not manually guess an ID. Select the correct credential through the
n8n UI.

Step 3 --- Verify the model

Open 10. Gemini Chat Model.

Confirm that:

A valid Gemini credential is selected.

A currently supported Gemini model is selected.

Temperature is 0.

If the selected model returns a 404 Not Found model-availability
error, choose a currently supported Gemini model from the dropdown.

Step 4 --- Index the knowledge base

Run:

## 1. Load Knowledge Base (Run Once)

The indexing path should process:

HR Policy Document
→ Data Loader
→ Text Splitter
→ Gemini Embeddings
→ Vector Store

Verify that the relevant indexing nodes execute successfully.

Step 5 --- Open the chat

Open:

7. When Chat Message Received

Use the built-in n8n chat interface.

Step 6 --- Test a known question

Enter:

How many casual leaves are allowed?

The answer should be based on the Leave Policy and should include:

Source: [LEAVE POLICY]

Step 7 --- Test the fallback

Enter:

What is the maternity leave policy?

Because maternity leave is not included in the supplied knowledge base,
the expected fallback is:

Information not available in the knowledge base. Please check with HR directly.

Step 8 --- Save the final workflow

After successful testing:

Save the workflow.

Keep the final JSON export.

Capture the required screenshots.

Include this README with the submission.

## 11. QA Test Set

Use the following five questions for the Day 21 evidence screenshots.

#                Question          Expected Topic    Expected Source

1                 How many casual Casual leave      [LEAVE POLICY]
leaves are
allowed?

2                 What are the    Working hours     [WORKING HOURS]
standard working
hours?

3                 How is          Attendance        [ATTENDANCE POLICY]
attendance
marked?

4                 When does the   Internship        [INTERNSHIP GUIDELINES]
weekly progress   guidelines
review take
place?

12. Updating the Knowledge Base

To update or expand the policies:

Open 2. HR Policy Document.

Edit the text field.

Keep the section markers in the format:

[SECTION NAME]
Policy content...

Save the workflow.

Re-run the indexing path.

Example:

[LEAVE POLICY]
Casual Leave: Employees get 12 casual leave days per year.

[WORKING HOURS]
Standard working hours are 9:30 AM to 6:30 PM, Monday to Friday.

After changing the source content, re-index the vector store before
testing the updated policy.

## Re-indexing note

Because the project uses an in-memory vector store and does not
implement a production-grade document ID/upsert strategy, repeated
indexing should be treated as a refresh operation for the exercise. For
a persistent production vector database, add explicit
deduplication/upsert or store-clearing logic.

## 13. Production Considerations

The current implementation is intentionally lightweight and suitable for
demonstrating the Day 21 RAG objective.

For production use, the following improvements would be appropriate:

Current Exercise Design             Production Improvement

In-memory vector store              Persistent vector DB such as
Qdrant, Pinecone, or
Supabase/pgvector

HR policy stored in a Set node      Load from controlled sources such
as Google Drive, SharePoint,
Notion, or a document repository

Manual re-indexing                  Scheduled or event-driven indexing

No explicit document versioning     Add policy version, effective date,
document ID, and update timestamp
metadata

Simple source label                 Return document title, section,
version, and/or document URL

Single static knowledge base        Separate documents by policy,
department, region, or employee
type where required

Prompt-based fallback               Add retrieval relevance thresholds
and deterministic validation where
appropriate

Simple chat testing UI              Secure internal HR frontend with
authentication and authorization

No access control in the workflow   Restrict HR documents and answers
according to employee permissions

## 14. Security and Privacy Notes

This project uses HR policy content, so production implementations
should treat the knowledge base as controlled organizational
information.

## Recommended production safeguards include:

Store API keys in n8n Credentials, never in workflow text.

Restrict access to the workflow and chat endpoint.

Avoid putting confidential employee records into a general-purpose
test knowledge base.

Add authentication before exposing the chat publicly.

Apply authorization if different employees should have access to
different policies.

Consider data retention and logging requirements before storing user
questions or answers.

Keep policy documents versioned so answers can be traced to the
correct policy version.

## 15. Troubleshooting

Error: 404 Not Found for a Gemini model

Cause: The selected Gemini model is unavailable to the current
account/API configuration.

Fix:

Open 10. Gemini Chat Model.

Open the model selector.

Choose a currently supported model.

Save the node.

Test again.

Do the same for the embeddings node if the error specifically identifies
the embeddings model.

Error: Information not available... for a question that should be answered

Check:

The knowledge base was indexed successfully.

The question relates to content actually present in the HR policy
document.

The retriever is connected to the vector store.

The retriever has a non-zero topK value.

The embeddings credential/model is working.

The vector store uses the same hr_policy_kb memory key for
indexing and retrieval.

Error: Chat works but answers are not grounded

Check:

HR Policy QA Chain is receiving the retriever.

Gemini Chat Model is connected to the QA chain.

The system prompt still contains the grounding instructions.

The retrieved chunks actually contain the required policy
information.

The model is not being given an unrelated source of information that
conflicts with the policy context.

Error: Knowledge base is empty after restart

This is expected behavior for an in-memory vector store.

Run:

1. Load Knowledge Base (Run Once)

again to rebuild the vector store.

For production, use a persistent vector database.

## 16. Deliverables

Day21_HR_Policy_RAG_Assistant/
│
├── Day21_HR_Policy_RAG_Assistant.json
├── README.md
│
└── screenshots

containing the five test questions and expected behavior.

## 17. Day 21 Learning Outcomes

This project demonstrates the following concepts:

RAG Architecture

Understanding how:

Documents
→ Chunks
→ Embeddings
→ Vector Store
→ Retriever
→ LLM
→ Grounded Answer

work together.

Vector Search

The workflow converts policy text into embeddings and retrieves relevant
chunks based on semantic similarity.

Retriever + Vector Store + Chat Model

The three major AI components are wired into a Retrieval QA Chain:

Vector Store
      ↓
Retriever
      ↓
QA Chain ← Gemini Chat Model

Grounded Prompting

The model receives retrieved policy context and is instructed not to
invent information outside that context.

Safe Fallback

The assistant has a deterministic target response for questions that are
not covered by the knowledge base.

Source Awareness

The response includes a source section marker such as:

Source: [LEAVE POLICY]

Practical AI Automation

The complete system is implemented inside n8n without requiring a custom
frontend or a separate backend service for this exercise.

## 19. Conclusion

The Day 21 HR Policy RAG Assistant demonstrates a complete, practical
RAG workflow in n8n.

Instead of asking Gemini to answer HR questions from its general
knowledge, the system first retrieves relevant policy information from a
dedicated knowledge base and then instructs the model to answer from
that retrieved information only.

The result is a small but complete architecture covering:

Knowledge Ingestion
       ↓
Chunking
       ↓
Embeddings
       ↓
Vector Storage
       ↓
Retrieval
       ↓
Grounded Generation
       ↓
Source Citation
       ↓
Safe Fallback

This implementation satisfies the core Day 21 objective while keeping
the architecture simple enough to understand, test, demonstrate, and
extend.
