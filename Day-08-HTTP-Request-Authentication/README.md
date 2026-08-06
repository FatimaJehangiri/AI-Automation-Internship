## Day 8 – HTTP Request Node & Authentication

## Overview

This project demonstrates how to integrate an external REST API into an n8n workflow using the HTTP Request node.
The workflow automatically retrieves weather forecast data, evaluates whether rain is expected, and sends a notification based
on the result.

## Objectives

- Learn the HTTP Request node in n8n.
- Perform GET requests to an external API.
- Work with query parameters and JSON responses.
- Understand common API authentication methods.
- Build an automated weather notification workflow.

## Workflow

```
Schedule Trigger
        │
        ▼
HTTP Request (Weather API)
        │
        ▼
Set (Extract Weather Code)
        │
        ▼
IF (Rain Expected?)
      │        │
     Yes       No
      │
      ▼
Send Notification
```

## Technologies Used

- n8n
- Open-Meteo Weather API
- REST API
- JSON

## Project Files

| File | Description |
|------|-------------|
| workflow.json | Exported n8n workflow |
| auth-method.md | API authentication note |
| screenshots/ | Workflow and execution screenshots |
| weather-response/ | Sample API response |

## Learning Outcomes

- HTTP Request node
- REST API integration
- JSON data handling
- Conditional workflow logic
- API authentication concepts
- Workflow automation using n8n
