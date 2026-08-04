HR Candidate Screening Automation (n8n)
Overview

This project is an AI-powered HR Candidate Screening Automation built with n8n. It automates the initial recruitment 
process by retrieving candidate records from Google Sheets, validating applications, categorizing candidates based on 
predefined screening criteria, updating their status, and sending personalized email notifications. The workflow significantly
reduces manual effort while ensuring a consistent and efficient screening process.

Workflow
Manual Trigger
Starts the workflow execution manually.
Fetch Candidate Records
Retrieves candidate information from a Google Sheets document.
Validate & Screen Candidates
Validates mandatory fields (Name, Email, Degree).
Applies screening rules based on:
Relevant degree
Work experience
Candidate availability
Assigns one of three categories:
Shortlisted
Needs Review
Not Eligible
Categorize Candidates
Routes candidates into different workflow branches according to their assigned category.
Update Google Sheets
Updates the candidate's screening result in the original spreadsheet using the email address as the unique identifier.
Email Validation
Ensures an email address exists before sending notifications.
Send Automated Emails
Sends personalized emails based on the candidate's status:
✅ Shortlisted
🟡 Needs Review
❌ Not Eligible

Technologies Used
n8n
Google Sheets
Gmail API
JavaScript (n8n Code Node)

Features
Automated candidate data retrieval
Candidate validation and screening
Rule-based categorization
Automatic Google Sheets updates
Personalized email notifications
Modular and scalable workflow design

Workflow Architecture
Manual Trigger
      │
      ▼
Fetch Candidate Records (Google Sheets)
      │
      ▼
Validate & Screen Candidates (JavaScript)
      │
      ▼
Categorize Candidate
      ├──────────────► Shortlisted
      │                   │
      │                   ▼
      │          Update Google Sheet
      │                   │
      │                   ▼
      │          Send Shortlisted Email
      │
      ├──────────────► Needs Review
      │                   │
      │                   ▼
      │          Update Google Sheet
      │                   │
      │                   ▼
      │          Send Review Email
      │
      └──────────────► Not Eligible
                          │
                          ▼
                 Update Google Sheet
                          │
                          ▼
                 Send Rejection Email

Outcome

This workflow automates the initial HR screening process by validating candidate information, categorizing applications according 
to predefined recruitment rules, updating recruitment records in real time, and sending automated email notifications. It improves
recruitment efficiency, minimizes manual work, and provides a consistent screening process for all applicants.
























