# Day 9 – JavaScript for Automation (Code Node)

## Overview

This project demonstrates how to use the **Code node** in n8n to transform and clean a messy dataset using JavaScript.
The workflow normalizes names and email addresses, calculates grades based on scores, and filters the dataset to keep only
records with grades **A** or **B**.

## Objectives

- Learn JavaScript basics for automation.
- Transform data using the n8n Code node.
- Normalize inconsistent text formatting.
- Compute grades from numeric scores.
- Filter data using JavaScript array methods.

## Workflow

```
Manual Trigger
      │
      ▼
   Code Node
      │
      ▼
Transformed Output
```

## Data Transformation

The Code node performs the following operations:

- Removes extra spaces from names using `trim()`.
- Converts names to Title Case.
- Converts email addresses to lowercase.
- Calculates grades (A–F) based on scores.
- Filters records to keep only students with grades **A** or **B**.

## JavaScript Concepts Used

- `const` and `let`
- Functions
- Objects and Arrays
- Template Literals
- Optional Chaining (`?.`)
- `.map()`
- `.filter()`

## Files

| File | Description |
|------|-------------|
| workflow.json | Exported n8n workflow |
| README.md | Project documentation |
| screenshots/ | Workflow and execution screenshots |

## Learning Outcomes

- Data transformation using JavaScript
- Working with the n8n Code node
- Array manipulation with `.map()` and `.filter()`
- JSON data processing
- Workflow automation using n8n

---
