Day 4 — Logic & Data Transformation

Petalnex Pvt. Ltd. — AI Automation Internship

Objective

Route and reshape data using branching, filtering, merging, and looping — the core logic patterns behind every real n8n workflow.

Concepts Covered
IF node — single condition, two-way branch
Switch node — single condition, multi-way branch (used here for 3 groups)
Filter node — keeps/removes items without branching
Merge node — recombines multiple branches into one stream
Loop Over Items (SplitInBatches) — processes items one at a time; enables controlled, per-item logic
Edit Fields (Set node) — add/rename/remove fields on each item
Practical Task

Built a routing workflow over 10 sample records with a numeric score field.

Flow:
Manual Trigger → Code (10 records) → Loop Over Items → Switch (by score)
                                                          ├── Excellent → Edit Fields
                                                          ├── Good → Edit Fields
                                                          └── Needs Improvement → Edit Fields
                                                                    ↓
                                                                  Merge → back to Loop

Routing logic:

Score Range	Group	Next Action
≥ 85	Excellent	Assign to advanced track / fast-track review
70–84	Good	Provide feedback session for improvement
< 70	Needs Improvement	Schedule mentoring / remedial plan

Each record exits the workflow with two new fields attached: group and next_action.

Files
day4_routing_workflow.json — importable n8n workflow
screenshot_routed_branches.png — execution output showing all 3 branches
Key Takeaway

Switch replaces nested IF nodes once you have 3+ outcomes — cleaner logic, easier to maintain. Looping isn't just for repetition; it gives per-item control before merging results back into a single dataset, which is exactly the pattern used for real client workflows (e.g., processing leads or records individually before batch output).
