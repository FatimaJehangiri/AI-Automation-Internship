## Day 14 — Prompt Engineering
## Petalnex Pvt. Ltd. — AI Automation Internship
## Overview

Day 14 focused on Prompt Engineering for AI automation. The objective was to design prompts that are specific, constrained, reusable, and reliable enough to drive automation workflows in n8n.

The workflow was designed to test 5 different automation tasks using structured prompts and strict JSON output contracts.

## Objectives
Understand the ROLE → CONTEXT → TASK → RULES → OUTPUT FORMAT → EXAMPLES prompt framework.
Create reusable prompts for automation workflows.
Use explicit rules to reduce inconsistent AI responses.
Enforce structured JSON output from an LLM.
Test prompts against multiple inputs.
Parse and validate AI-generated JSON responses in n8n.
Track successful and failed test cases.
Automation Tasks

## Five prompt types were implemented:

Lead Qualification
Classifies leads based on supplied business information.
Returns qualification, priority, score, reason, and next action.
Email Classification
Categorizes incoming business emails.
Identifies category, priority, sentiment, summary, and reason.
CV Analysis
Extracts structured candidate information.
Identifies skills, education, experience, summary, and missing information.
Complaint Classification
Categorizes customer complaints.
Determines priority, sentiment, department, and summary.
Support Reply
Generates concise customer-support responses.
Identifies tone, missing information, and whether human review is required.
Prompt Structure

Each prompt follows a consistent structure:

ROLE
CONTEXT
TASK
RULES
OUTPUT FORMAT
EXAMPLES

The prompts were designed to:

Prevent the model from inventing missing information.
Restrict possible output values.
Require JSON-only responses.
Define the expected output schema.
Provide examples for consistent classification.
n8n Workflow

## Workflow Flow
Build 5 Prompts + 15 Tests
          ↓
Groq HTTP Request
          ↓
Parse + Validate JSON Output
          ↓
Collect All 15 Results
Nodes Used
Node	Purpose
Build 5 Prompts + 15 Tests	Creates the five prompts and 15 test cases.
Groq HTTP Request	Sends each prompt and test input to the Groq LLM.
Parse + Validate JSON Output	Parses the model response and checks whether the response follows the expected JSON structure.
Collect All 15 Results	Combines the validated test results into a single final results object.

## Testing & Validation

The workflow was designed for 15 test cases, with 3 tests for each automation task.

The final n8n execution reported:

Total Tests:      15
Completed Tests:  15
Failed Tests:      0

Each validated record contains information such as:

{
  "test_id": 1,
  "task": "Lead Qualification",
  "status": "PASS",
  "parse_status": "valid"
}

The validation step ensures that the AI response can be parsed as JSON before it is passed further into the automation.

## Key Learning

The main lesson from this task was that an AI model should not simply be given a vague instruction such as:

Analyze this lead.

Instead, an automation-ready prompt should clearly define:

Who the AI is acting as.
What information it should use.
What task it must perform.
What rules it must follow.
Exactly what format it must return.
Examples of expected behavior.

This makes the AI output much easier to process reliably inside an automation workflow.

## Deliverable

Prompt-Library.pdf

The PDF contains the five prompt designs, test cases, generated outputs, validation results, and an overview of the n8n workflow.

## Technologies Used
n8n
Groq API
LLM / Generative AI
JavaScript
JSON
HTTP Request
Prompt Engineering

## Outcome
Successfully developed and tested a structured prompt-engineering workflow in n8n. The workflow demonstrates how carefully designed prompts and strict JSON output contracts can make LLM responses more suitable for reliable, repeatable AI automation.
