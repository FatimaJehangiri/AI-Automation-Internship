# Day 5 — Google Sheets & Email Automation

**Petalnex Pvt. Ltd. — AI Automation Internship**

## Objective
Connect n8n to Google Sheets and Gmail to read applicant data, validate it, apply eligibility rules, write results back, and send personalized emails — a complete "read → decide → update → notify" pipeline.

## Concepts Covered
- **Google Sheets node** — reading rows, updating rows by matching column, mapping modes
- **Gmail node** — sending personalized emails using expressions
- **IF node** — validating records before acting on them, and gating actions (e.g. skipping email when no address exists)
- **Switch node** — multi-way eligibility routing
- **Merge node** — recombining branches, in both "combine" and "append" modes, and understanding when each is appropriate
- **Set (Edit Fields) node** — attaching computed fields (category, email content) per branch

## Practical Task
Built an **Internship Application Processing** workflow over 10 sample applicants pulled from a Google Sheet.

**Flow:**
```
Read Applications → Validate Data (IF)
                       ├── valid → Categorize Candidate (Switch: Shortlisted / Needs Review / Not Eligible)
                       │              → Set fields (category, email_subject, email_body per outcome)
                       └── invalid → Set "Invalid - Missing Data"
                                              ↓
                                        Merge All Records
                                          ├── Update Sheet (writes category back, matched by id)
                                          └── IF (has email?) → Send Email (personalized per category)
```

**Eligibility rules:**
| Condition | Category |
|---|---|
| Score ≥ 80 AND CGPA ≥ 3.0 | Shortlisted |
| Score ≥ 60 (not Shortlisted) | Needs Review |
| Score < 60 | Not Eligible |
| Missing name/email/score | Invalid - Missing Data |

Each candidate receives a personalized email matching their outcome, and the sheet gets updated with their final category.

## Debugging Notes (the part that actually mattered)

Building this taught more about real n8n behavior than the "happy path" ever would:

- **Type mismatches in conditions:** an IF node comparing a number field with a string-typed condition threw `Wrong type: '88' is a number but was expecting a string`. Fixed by enabling "Convert types where required" instead of hardcoding types.
- **Set node fields silently empty:** after import, three Set nodes showed "Fields to Set" as empty with `Include Other Input Fields` inconsistently toggled — meaning every downstream node was processing empty `{}` objects despite showing an item count. The fix was manually re-adding each field and confirming "Include Other Input Fields" was ON everywhere.
- **Merge node mode matters:** using "Combine" mode on mutually-exclusive Switch branches caused the merge to stall, since it waited for matching data on every input. Switched to "Append" mode, which just passes through whatever arrives on any input.
- **`pairedItem` tracking breaks across certain nodes:** referencing `$('NodeName').item.json.field` failed silently after a Google Sheets Update operation, because paired-item lineage wasn't preserved. Fixed by restructuring the workflow so Update Sheet and Send Email both branch independently off the same Merge node, rather than chaining one after the other.
- **A node can just go missing:** during heavy rewiring, the Update Sheet node itself got disconnected/deleted without an obvious error — worth periodically checking the full canvas view, not just the node you're actively debugging.

## Files
- `day5_application_workflow.json` — final, tested n8n workflow
- `sample_applications.csv` — sample sheet with categories filled in after execution
- `screenshot_sheet_updated.png` — Google Sheet showing all 10 rows categorized
- `screenshot_sent_email.png` — a personalized email in Gmail Sent folder

## Key Takeaway
Real automation work is less about connecting nodes and more about verifying data actually flows correctly at every hop — checking actual JSON output, not just green checkmarks, is what catches silent failures like empty objects or broken references.

## Resources
- [n8n Docs – Google Sheets node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)
- [n8n Docs – Gmail node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.gmail/)
