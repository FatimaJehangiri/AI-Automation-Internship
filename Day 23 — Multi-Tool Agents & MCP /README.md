# Day 23 — Multi-Tool Agent & MCP

## Overview

A multi-tool AI agent built with n8n that intelligently routes user requests to specialized tools. Features include HTTP API integration, data store access, arithmetic calculations, MCP remote capabilities, and human-in-the-loop approval for sensitive actions.

---

## Features

- **Intelligent Tool Routing**: Agent selects the most appropriate tool for each request
- **Weather API**: Fetch real-time weather data via Open-Meteo
- **Data Store**: Query employee records from Google Sheets
- **Calculator**: Perform arithmetic operations
- **MCP Integration**: Connect to remote MCP servers for extended capabilities
- **Human Approval**: Sensitive actions require explicit human confirmation
- **Conversation Memory**: Maintains context across multiple interactions

---

## Architecture
User Input → Chat Trigger → Multi-Tool Agent → Tool Selection → Response
↓
┌─────────┼─────────┐
↓ ↓ ↓
Weather Sheets Calculator
↓ ↓
MCP Human Approval


---

## Prerequisites

- n8n instance (v1.0+)
- Groq API Key
- Google Sheets account (for data store)
- Optional: MCP server endpoint

---

## Quick Setup

### 1. Clone the Workflow

Import the provided `workflow.json` into your n8n instance.

### 2. Configure Credentials

| Credential | Required | Where to Get |
|------------|----------|--------------|
| Groq API Key | ✅ Yes | [console.groq.com](https://console.groq.com) |
| Google Sheets OAuth | For Data Store | Google Cloud Console |
| MCP Server URL | For MCP Tools | Your MCP server |

### 3. Update Google Sheet ID

1. Create a Google Sheet with employee data
2. Copy the Sheet ID from the URL
3. Update in the "Get row(s) in sheet" node

### 4. Activate Workflow

Toggle the workflow to "Active" status.

---

## Testing the Tools

| Test Case | Command |
|-----------|---------|
| Calculator | `Calculate 25 * 4` |
| Weather | `What's the weather in London?` |
| Data Store | `Find employee Alice` |
| Sensitive Action | `Create a new record with title "Test"` |
| MCP (if configured) | `Use MCP to [your request]` |

---

## Tool Descriptions

| Tool | Description | Autonomous |
|------|-------------|------------|
| Public Weather HTTP | Fetch current weather data | ✅ Yes |
| Calculator | Perform arithmetic operations | ✅ Yes |
| Google Sheets | Query employee records | ✅ Yes |
| Remote MCP Client | Extended capabilities via MCP | ✅ Yes |
| Sensitive Demo Write | Create demo records | ❌ Human approval required |

## Troubleshooting

### ❌ "Tool not configured"
- Verify all credentials are added
- Check tool connections to agent node
- Ensure workflow is active

### ❌ Google Sheets not working
- Verify Sheet ID and name are correct
- Check OAuth permissions
- Ensure sheet is shared with service account

### ❌ MCP connection failed
- Verify MCP server URL is correct
- Check server is running
- Review authentication headers

### ❌ Human approval not triggering
- Use the chat interface
- Type "approve" or "reject" exactly
- Check timeout settings

### Quick Reference Card
┌─────────────────────────────────────────────────────────┐
│ DAY 23 — MULTI-TOOL AGENT │
├─────────────────────────────────────────────────────────┤
│ │
│ 🛠 TOOLS AVAILABLE │
│ ───────────────────────────────────── │
│ 🌤 Weather API → Public HTTP │
│ 📊 Data Store → Google Sheets │
│ 🧮 Calculator → Arithmetic │
│ 🔌 MCP Client → Remote MCP Server │
│ 🔒 Sensitive Action → Human Approval Required │
│ │
│ 🚀 QUICK START │
│ ───────────────────────────────────── │
│ 1. Activate workflow │
│ 2. Use chat interface │
│ 3. Test: "Calculate 25*4" │
│ │
│ 🔑 CREDENTIALS NEEDED │
│ ───────────────────────────────────── │
│ ✓ Groq API Key │
│ ✓ Google Sheets OAuth (for data store) │
│ ✓ MCP Server URL (optional) │
│ │
└─────────────────────────────────────────────────────────┘


