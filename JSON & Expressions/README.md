# Day 3 — JSON, Data & Expressions (n8n)

## Objective
Understand n8n's data model and use expressions to classify records automatically.

## What this workflow does
- Generates 5 sample student/employee records (Name, Email, Department, Score)
- Uses an Edit Fields (Set) node with an expression to classify each record:
  - Score ≥ 80 → Excellent
  - Score 60–79 → Good
  - Score < 60 → Needs Improvement

## Key concept used
{{ $json.Score >= 80 ? 'Excellent' : $json.Score >= 60 ? 'Good' : 'Needs Improvement' }}

## How to run
1. Import `workflow.json` into n8n
2. Click "Execute Workflow"
