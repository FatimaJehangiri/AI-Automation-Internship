# Day 22 — AI Agent with Knowledge Base & Memory

## AI Automation Internship 

## 1. Project Overview

This project demonstrates how to build a simple, goal-driven **AI Agent in n8n** that can use an external tool and maintain conversation context through memory.

Unlike a traditional automation workflow that follows a fixed sequence of steps, an AI Agent can interpret a user's goal, decide whether it needs a connected tool, use that tool, observe the result, and generate an appropriate response.

The workflow was developed as part of **Day 22 — AI Agents: Tools, Memory & Reasoning** of the AI Automation Internship.

### Core Concepts Demonstrated

* AI Agent architecture
* Tool calling
* Knowledge Base search
* Conversation memory
* System prompts and agent boundaries
* Goal-driven decision making
* No-hallucination behavior
* Conversational follow-up handling

---

# 2. Objective

The objective of this task is to move from fixed automation workflows to **goal-driven AI agents**.

The agent should be able to:

1. Receive a user's message through a chat interface.
2. Understand the user's goal.
3. Decide whether it needs information from the Knowledge Base.
4. Call the Knowledge Base Search tool when appropriate.
5. Generate a response based on the available information.
6. Remember relevant information from previous conversation turns.
7. Ask for clarification when the user's request is unclear.
8. Avoid inventing information that is not available in the Knowledge Base.

---

# 3. Workflow Architecture

The workflow follows this architecture:

```text
┌──────────────────────────────┐
│   When Chat Message Received │
└──────────────┬───────────────┘
               │
               ▼
       ┌────────────────┐
       │    AI Agent    │
       └───────┬────────┘
               │
       ┌───────┼────────┐
       │       │        │
       ▼       ▼        ▼
   Chat Model Memory  Knowledge
                      Base Tool
       │       │        │
       └───────┼────────┘
               ▼
        Agent Response
```

### Main Components

| Component             | Purpose                                                 |
| --------------------- | ------------------------------------------------------- |
| Chat Trigger          | Receives user messages                                  |
| AI Agent              | Understands the request and decides how to respond      |
| Chat Model            | Provides language understanding and generation          |
| Conversation Memory   | Remembers previous conversation turns                   |
| Knowledge Base Search | Provides information from the predefined knowledge base |

---

# 4. Technology Stack

* **n8n** — Workflow automation platform
* **Groq Chat Model** — LLM provider used for the AI Agent
* **JavaScript** — Knowledge Base search logic
* **n8n AI Agent** — Agent reasoning and tool selection
* **n8n Memory** — Conversation memory
* **n8n Chat Trigger** — User interaction interface

> The original workflow can also use another supported chat model. Groq is used here because the OpenAI API account had no remaining credits.

---

# 5. Why an AI Agent?

A traditional workflow might look like:

```text
Input
  ↓
Step 1
  ↓
Step 2
  ↓
Step 3
  ↓
Output
```

Every input follows the same predefined sequence.

An AI Agent is different:

```text
User Goal
    ↓
AI Agent
    ↓
Understand Request
    ↓
Decide Whether a Tool Is Needed
    ↓
Use Tool
    ↓
Observe Result
    ↓
Generate Response
```

The agent has the ability to make decisions about how to handle the request instead of simply executing a fixed sequence.

---

# 6. AI Agent vs Standard Workflow

| Standard Workflow            | AI Agent                    |
| ---------------------------- | --------------------------- |
| Fixed sequence               | Goal-driven                 |
| Predetermined steps          | Can choose available tools  |
| Limited decision capability  | Dynamic decision capability |
| Usually deterministic        | Model-driven reasoning      |
| No tool selection            | Can select tools            |
| Context is explicitly passed | Can use memory              |
| Mostly automation logic      | Combines automation and AI  |

An AI Agent is therefore more than a chatbot.

An agent can have:

* Instructions
* Tools
* Memory
* Context
* Boundaries
* Decision capability

---

# 7. System Prompt

The AI Agent uses the following strict system prompt:

```text
You are a helpful AI assistant for Petalnex Pvt. Ltd. AI Automation Internship knowledge.

Your job is to answer the user's questions accurately.

RULES:

1. Use the Knowledge Base Search tool when you need information from the knowledge base.
2. Never invent facts, information, project details, or answers.
3. If the answer is not available in the knowledge base, clearly say:
"I don't have enough information in my knowledge base to answer that accurately."
4. If the user's question is unclear, ask a short clarifying question.
5. Use conversation memory when the user refers to something mentioned earlier.
6. Keep answers clear, concise, and professional.
7. Do not pretend to have used a tool if you did not use it.

You have access to one knowledge base search tool and conversation memory.
```

### Purpose of the Prompt

The system prompt establishes clear boundaries for the agent.

It instructs the agent to:

* Use the available tool when appropriate.
* Avoid hallucinating information.
* Ask for clarification.
* Use conversation memory.
* Keep responses professional.
* Be transparent about tool usage.

---

# 8. Knowledge Base

The project uses a lightweight JavaScript-based Knowledge Base.

This approach was intentionally selected to keep the Day 22 workflow simple and easy to integrate.

The Knowledge Base contains information about:

* AI Agents
* Standard Workflows
* AI Agent Tools
* Memory
* Structured AI Output
* Customer Support Triage
* RAG
* Embeddings
* n8n AI Automation

The Knowledge Base is stored as JavaScript objects:

```javascript
const knowledgeBase = [
  {
    topic: "AI Agents",
    content: "An AI agent is a goal-driven system..."
  },
  {
    topic: "Memory",
    content: "Conversation memory allows an AI agent..."
  }
];
```

---

# 9. Knowledge Base Search Logic

The Knowledge Base Tool receives a query from the AI Agent.

The query is converted to lowercase and split into search terms.

The tool then compares those terms against the topic and content of each Knowledge Base entry.

Conceptually:

```text
User Question
      ↓
Extract Search Terms
      ↓
Search Knowledge Base
      ↓
Find Matching Entries
      ↓
Return Relevant Information
```

If relevant information is found, it is returned to the AI Agent.

If no matching information is found, the tool returns:

```text
No relevant information was found in the knowledge base.
```

This allows the agent to follow the system prompt and avoid fabricating an answer.

---

# 10. Knowledge Base Tool Code

The Knowledge Base Search tool uses JavaScript.

The core search logic is:

```javascript
const searchTerms = query
  .toLowerCase()
  .split(/\s+/)
  .filter(word => word.length > 2);

const results = knowledgeBase.filter(item => {
  const searchableText =
    `${item.topic} ${item.content}`.toLowerCase();

  return searchTerms.some(term =>
    searchableText.includes(term)
  );
});
```

The matching results are then returned to the AI Agent.

---

# 11. Conversation Memory

Conversation memory allows the agent to remember information from earlier turns.

Without memory:

```text
User:
My name is Fatima.

Agent:
Nice to meet you.

User:
What is my name?

Agent:
I don't know.
```

With memory:

```text
User:
My name is Fatima.

Agent:
Nice to meet you, Fatima.

User:
What is my name?

Agent:
Your name is Fatima.
```

This demonstrates that the agent can maintain conversational context.

---

# 12. How Memory Works in This Project

The workflow uses an n8n memory sub-node connected directly to the AI Agent.

The memory stores recent conversation messages.

The configured context window is approximately:

```text
10 messages
```

This allows the agent to use recent conversation history when processing follow-up questions.

---

# 13. Groq Chat Model

The AI Agent requires a language model to understand requests and generate responses.

For this implementation, the **Groq Chat Model** is used.

The connection is:

```text
Groq Chat Model
       │
       │ AI Language Model
       ▼
   AI Agent
```

The Groq API credential is configured directly in the n8n Groq Chat Model node.

### Security

The Groq API key must never be:

* Uploaded to GitHub
* Included in the README
* Shared in screenshots
* Hardcoded into JavaScript
* Shared publicly

The n8n credential system should be used to securely store the API key.

---

# 15. Final Connections

The final workflow should contain these connections:

```text
When Chat Message Received
             │
             ▼
          AI Agent
        ▲     ▲     ▲
        │     │     │
        │     │     │
      Model Memory Tool
        │     │     │
        ▼     ▼     ▼
      Groq   Memory  Knowledge
      Model          Base Search
```

The three AI connections are:

```text
Groq Chat Model
       ↓
AI Agent
```

```text
Conversation Memory
       ↓
AI Agent
```

```text
Knowledge Base Search
       ↓
AI Agent
```

---

# 16. Testing the Workflow

The workflow should be tested using several different scenarios.

## Test Case 1 — Knowledge Base Question

### User

```text
What is an AI Agent?
```

### Expected Behavior

The agent should retrieve relevant information from the Knowledge Base and explain what an AI Agent is.

---

## Test Case 2 — Tool Usage

### User

```text
What are common AI Agent tools?
```

### Expected Behavior

The Knowledge Base Search tool should provide information about tools such as:

* Knowledge Base search
* Google Sheets
* Databases
* APIs
* Email
* Ticket creation

The agent should then summarize the result.

---

## Test Case 3 — RAG

### User

```text
What is RAG?
```

### Expected Behavior

The agent should provide an explanation based on the RAG information stored in the Knowledge Base.

---

## Test Case 4 — Unknown Information

### User

```text
What is the exact population of the fictional planet Zorax?
```

### Expected Behavior

The agent should not invent an answer.

It should respond that it does not have enough information in its Knowledge Base.

This demonstrates the no-hallucination boundary.

---

# 17. Memory Test

This is one of the most important tests for the assignment.

### First Message

```text
My name is Fatima and I am learning AI automation.
```

The agent responds normally.

### Follow-Up Message

```text
What is my name?
```

### Expected Result

```text
Your name is Fatima.
```

### Second Follow-Up

```text
What am I learning?
```

### Expected Result

```text
You are learning AI automation.
```

This demonstrates that the Conversation Memory is functioning.

---

# 18. Recommended Test Transcript

A good screenshot for the final submission can show:

```text
User:
My name is Fatima and I am learning AI automation.

AI Agent:
Nice to meet you, Fatima. AI automation is a useful area to explore.

User:
What is my name?

AI Agent:
Your name is Fatima.

User:
What am I learning?

AI Agent:
You are learning AI automation.
```

This provides clear evidence that memory is working.

---

# 19. Boundary Test

Another important test is to verify that the agent does not hallucinate.

Ask:

```text
Tell me something about a topic that is not in the knowledge base.
```

The agent should not confidently create fictional information.

The expected behavior is:

```text
I don't have enough information in my knowledge base to answer that accurately.
```

This validates the strict system prompt.

---

# 20. Troubleshooting

## Groq Authentication Error

If the Groq node reports an authentication error:

* Check the API key.
* Verify that the correct Groq credential is selected.
* Create a new API key if necessary.
* Do not paste the key into the workflow JavaScript.

---

## Knowledge Base Syntax Error

If you receive an error such as:

```text
SyntaxError: Unexpected identifier
```

check that the JavaScript was pasted into the correct **Code Tool / Tool Code JavaScript field**.

Do not paste JavaScript into the tool description field.

Also make sure quotation marks have not been converted into invalid characters.

---

## Agent Does Not Use the Tool

Check:

1. The Knowledge Base node is actually an AI Tool.
2. It is connected to the AI Agent's **Tool** input.
3. The tool has a useful description.
4. The system prompt tells the agent when to use it.
5. The user's question is relevant to the Knowledge Base.

---

## Memory Does Not Work

Check:

1. Memory is connected to the AI Agent's **Memory** input.
2. The same chat session is being used.
3. The conversation was not restarted between tests.
4. The memory session ID is consistent.
5. The memory window is large enough to include the previous message.

---

# 21. Security Considerations

The project uses API credentials, therefore credentials should be handled securely.

### Never commit:

```text
API keys
Passwords
Tokens
Private credentials
```

to GitHub.

Use n8n's credential management system instead.

If an API key is accidentally exposed publicly, revoke it and generate a new one.

---

# 22. Limitations

This project intentionally uses a simple JavaScript Knowledge Base rather than a production vector database.

Therefore, the search mechanism is based on keyword matching rather than semantic similarity.

For example:

```text
AI Agent
```

will match Knowledge Base content containing terms related to:

```text
AI
Agent
```

A production RAG system would typically use:

```text
Documents
    ↓
Text Extraction
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector Database
    ↓
Semantic Search
    ↓
AI Agent
```

This more advanced architecture will be covered in the RAG and Vector Database tasks.

---

# 23. Why This Implementation Is Appropriate 

The goal of Day 22 is to understand the fundamentals of AI Agents rather than build a complex production RAG platform.

This implementation therefore intentionally keeps the architecture simple while satisfying all assignment requirements.

It demonstrates:

### Tool

The AI Agent can call:

**Knowledge Base Search**

### Memory

The AI Agent can remember previous conversation turns.

### Reasoning / Decision Capability

The AI Agent decides when it needs to use the Knowledge Base tool.

### Instructions

The AI Agent follows a strict system prompt.

### Boundaries

The agent is instructed not to invent information.

### Conversation

The user interacts with the agent through an n8n Chat Trigger.

---


# 24. Expected Final Workflow

```text
                 ┌──────────────────────────┐
                 │ When Chat Message        │
                 │ Received                 │
                 └────────────┬─────────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │     AI Agent      │
                    │                   │
                    │ System Prompt     │
                    │ Reasoning         │
                    │ Tool Selection    │
                    └───────┬───────────┘
                            │
              ┌─────────────┼──────────────┐
              │             │              │
              ▼             ▼              ▼
       ┌────────────┐ ┌────────────┐ ┌──────────────────┐
       │ Groq Chat  │ │ Conversation│ │ Knowledge Base   │
       │ Model      │ │ Memory      │ │ Search Tool      │
       └────────────┘ └────────────┘ └──────────────────┘
                            │
                            ▼
                    ┌───────────────────┐
                    │ Final AI Response │
                    └───────────────────┘
```

---

# 25. Conclusion

This project demonstrates the fundamental architecture of an AI Agent using n8n.

The system moves beyond a fixed automation sequence by giving the AI Agent:

* A goal
* System instructions
* A language model
* A Knowledge Base tool
* Conversation memory
* Clear behavioral boundaries

The completed workflow demonstrates how an AI Agent can dynamically use a tool and maintain context across multiple conversation turns.

This provides the foundation for more advanced agentic automation systems involving databases, APIs, RAG pipelines, email, ticketing systems, and multiple tools.
