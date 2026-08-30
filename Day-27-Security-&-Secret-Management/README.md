## Day 27 --- Security & Secret Management

### Project Name
Secure AI Customer Support Triage Webhook

A security-enhanced version of the Week 3 AI Customer Support Triage
workflow, built in n8n. The workflow validates an incoming webhook
request before allowing it to continue to AI processing.

## Problem Statement

AI automation workflows often receive requests through webhooks and
interact with external services using API credentials. If a webhook is
left unprotected, unauthorized users may be able to submit requests to
the workflow. Similarly, exposing API keys, passwords, tokens, or
customer information in workflow files, screenshots, documentation, or
public repositories can create serious security risks.

The original customer-support automation needed an additional security
layer to ensure that incoming requests were authenticated before
reaching the AI processing stage.

## Objective

The objective of Day 27 was to audit and improve the security of the
Week 1--4 n8n automation work.

The main objectives were to:

Review workflows for hard-coded credentials and secrets.

Ensure API credentials are managed through n8n's credential system
where applicable.

Protect a webhook using a secret HTTP header.

Reject requests with a missing or incorrect secret.

Allow only authenticated requests to continue to AI processing.

Review handling of PII and sensitive customer information.

Prevent credentials and secrets from being exposed in GitHub files,
documentation, or screenshots.

Apply basic authentication, least-privilege, and
secure-data-handling principles.

## Workflow Architecture

Secured Workflow

Customer / Client
       |
       v
+----------------+
|    Webhook     |
+-------+--------+
        |
        v
+-------------------------+
| Validate Webhook Secret |
+-----------+-------------+
            |
       +----+----+
       |         |
     TRUE      FALSE
       |         |
       v         v
+------------+  +----------------------+
| AI Analysis|  | Respond to Webhook   |
+-----+------+  | HTTP 401 Unauthorized|
      |         +----------------------+
      v
+------------------+
| Structured JSON  |
+--------+---------+
         |
         v
+------------------+
| Switch/Category  |
+--------+---------+
         |
         v
+------------------+
| Store / Notify   |
+------------------+

## Security Boundary

The key security boundary is between the Webhook and the existing AI
processing workflow.

A request must provide the expected X-Webhook-Secret header before it
can continue to the AI processing stage.

## Technologies Used

n8n --- Workflow automation and orchestration

Webhook --- Receives incoming HTTP requests

IF node --- Validates the incoming security header

Respond to Webhook --- Returns an unauthorized response

AI/LLM integration --- Existing Week 3 customer-support analysis

JSON --- Structured request and AI output format

Postman --- Used to test webhook authentication

Git/GitHub --- Workflow version control and project sharing,
with security review before publication

## Nodes Used

Existing Nodes

The original Week 3 workflow contains the customer-support processing
nodes, including the AI analysis, structured output, routing, ticket
storage, acknowledgment, and/or alerting nodes.

## Security Nodes Added

## 1. Webhook

Purpose:
Receives the customer-support request.

Method:
POST

The webhook receives customer-support data and HTTP headers.

## 2. IF --- Validate Webhook Secret

Purpose:
Checks whether the incoming request contains the expected security
header.

The header being validated is:

X-Webhook-Secret

The expression used to read the header is:

{{$json.headers["x-webhook-secret"]}}

The value is compared against the configured test value in the workflow.

Never publish the actual secret value. A production implementation
should use secure secret management rather than exposing a real secret
directly in workflow logic.

## 3. Respond to Webhook

Purpose:
Handles requests that fail authentication.

The unauthorized branch returns:

{
  "success": false,
  "error": "Unauthorized request"
}

with HTTP status:

401 Unauthorized

This prevents unauthorized requests from continuing to the AI-processing
branch.

## 4. Existing AI Processing Nodes

After successful authentication, the request continues through the
existing Week 3 customer-support workflow.

The exact downstream nodes may include:

AI analysis

Structured JSON parsing/validation

Category/priority routing

Ticket storage

Customer acknowledgment

Urgent notification

## Setup Instructions

## Prerequisites

Before importing or running the workflow, make sure you have:

An n8n instance.

Access to the required AI/service credentials.

Postman or another HTTP client for testing.

## The Week 3 customer-support workflow.

Any database, messaging, or AI services required by the existing
workflow.

Step 1 --- Import the Workflow

Import the workflow JSON into n8n.

After importing, inspect all nodes and credentials before activating the
workflow.

Step 2 --- Configure Credentials

Open each node that connects to an external service and select the
appropriate n8n credential.

Do not place API keys, passwords, tokens, or other secrets directly
inside Code nodes, JSON files, README files, screenshots, or public
repositories.

Step 3 --- Configure the Webhook

The webhook receives a POST request.

The request should include the required customer-support information and
the security header:

X-Webhook-Secret

Do not document or publish the actual secret value.

Step 4 --- Configure the Security Check

The IF node reads:

{{$json.headers["x-webhook-secret"]}}

and compares the incoming value with the configured test secret.

The two branches are:

TRUE  → Continue to AI processing
FALSE → Respond to Webhook

Step 5 --- Configure Unauthorized Response

The Respond to Webhook node should return:

{
  "success": false,
  "error": "Unauthorized request"
}

with:

HTTP 401

Step 6 --- Test the Workflow

Use Postman to send requests to the webhook.

Use synthetic test data only.

Example:

{
  "customer_name": "Test Customer",
  "customer_email": "test@example.com",
  "message": "I need help with my order."
}

## Credentials Required

Names only --- never store or publish credential values.

Depending on the existing Week 3 workflow, the following credentials may
be required:

AI/LLM API Credential

Groq API Credential, if used by the workflow

Email/SMTP Credential, if used for notifications

## Workflow Explanation

1. Webhook Receives Request

The workflow starts when a client sends a POST request to the webhook.

The request contains customer-support information.

2. Security Validation

Before AI processing starts, the workflow checks the X-Webhook-Secret
header.

This creates an authentication layer at the beginning of the workflow.

3. Invalid Request

If the secret is:

Missing

Incorrect

Invalid

the IF node routes the request to the unauthorized branch.

The Respond to Webhook node returns:

401 Unauthorized

The request does not continue to the AI processing stage.

4. Valid Request

If the supplied security header matches the configured test value, the
request follows the TRUE branch.

The request then continues to the existing customer-support AI workflow.

5. AI Processing

The existing AI component analyzes the customer message and produces the
structured support information required by the workflow.

The security layer does not replace the existing AI logic; it protects
the entry point before AI processing begins.

6. Downstream Processing

After successful AI analysis, the workflow continues with its existing
routing, storage, acknowledgment, and alerting logic.

Test Cases

Test Case 1 --- Missing Secret

Request:

No X-Webhook-Secret header is provided.

Expected result:

IF → FALSE
    ↓
Respond to Webhook
    ↓
HTTP 401 Unauthorized

Result: PASS

The unauthorized branch executed and the request did not proceed through
the AI-processing branch.

Test Case 2 --- Incorrect Secret

Request:

An incorrect value is supplied in the X-Webhook-Secret header.

Expected result:

IF → FALSE
    ↓
Respond to Webhook
    ↓
HTTP 401 Unauthorized

Result: PASS

The incorrect-secret request was rejected.

Test Case 3 --- Correct Secret

Request:

The correct test secret is supplied in the X-Webhook-Secret header.

Expected result:

IF → TRUE
    ↓
AI Analysis
    ↓
Existing Customer Support Workflow

Result: PASS

The authenticated request continued through the existing workflow.

## Error Handling

The workflow handles webhook authentication failures using a dedicated
rejection branch.

Unauthorized Request

Condition:

Missing or incorrect webhook secret

Action:

Respond to Webhook

Response:

HTTP 401 Unauthorized

This prevents unauthenticated requests from reaching the AI processing
stage.

Additional Security Practices

The project also follows these practices:

Do not expose credentials in workflow files.

Do not expose secrets in screenshots.

Do not commit real secrets to GitHub.

Do not place passwords in README files.

Do not use real customer PII in public test data.

Review downloaded workflow JSON before sharing it.

Rotate/revoke a credential if a real secret has been exposed.

## Security Audit Checklist

## Security Check                      Status

Webhook authentication added        ✅ Completed

Missing secret rejected             ✅ Tested

Incorrect secret rejected           ✅ Tested

Correct secret accepted             ✅ Tested

Unauthorized requests prevented     ✅ Completed
from reaching AI processing

API credentials reviewed            ✅ Audited

Hard-coded secrets reviewed         ✅ Audited

Sensitive test data reviewed        ✅ Audited

Screenshots reviewed for exposed    ✅ Required before submission
secrets

## Known Limitations

This implementation is designed as an internship-level security
demonstration and has limitations:

A static shared secret is a basic authentication mechanism and is
not a complete production authentication system.

The example does not implement user identity, roles, or fine-grained
authorization.

Rate limiting and abuse protection are not implemented in the
workflow.

The webhook may require additional infrastructure-level protection
for production use.

Secret rotation is not automated.

Logging and monitoring of repeated unauthorized attempts are not
implemented.

HTTPS/TLS should be used in production to protect credentials and
request data in transit.

A production system should use a dedicated secret-management
approach appropriate to its deployment environment.

## Future Improvements

Possible improvements include:

Use stronger authentication mechanisms such as signed requests or
OAuth where appropriate.

Add rate limiting and abuse detection.

Add IP/network restrictions where appropriate.

Implement automated secret rotation.

Add security monitoring and alerting for repeated unauthorized
requests.

Add request validation and schema validation before processing.

Encrypt sensitive data where required.

Apply role-based authorization for different users or systems.

Add audit logs for security-relevant events.

Add automated security checks before workflow deployment.

Use a dedicated secrets manager for production deployments.

Add webhook replay protection using timestamps/nonces or request
signatures where appropriate.

Security and Privacy Notice

Never include actual passwords, API keys, access tokens, OAuth
secrets, database credentials, webhook secrets, or other confidential
values in this repository.

Do not expose sensitive information in:

README files

Workflow JSON

Screenshots

PDFs

GitHub commits

Demo videos

Test-data files

Use placeholders or credential names instead.

For example:

AI_API_KEY

may be referenced as a credential name, but the actual value must never
be written into the README.

Use synthetic data such as:

Test Customer
test@example.com

for public demonstrations.

## Deliverables

The Day 27 internship task includes:

1. Security-Audit.pdf

A security audit containing:

Findings

Security risks

Fixes implemented

Webhook security configuration

Test results

Security checklist

Conclusion

2. Secured Webhook Screenshot

A screenshot showing the security layer in the n8n workflow:

Webhook
   ↓
Validate Webhook Secret
   ├── TRUE  → Existing AI Workflow
   └── FALSE → 401 Unauthorized

The screenshot must not contain actual secrets, passwords, API keys,
tokens, or private customer information.

## Project Outcome

The Week 3 AI Customer Support Triage workflow was enhanced with a
webhook authentication layer.

The final implementation ensures that:

Requests without the required secret are rejected.

Requests with an incorrect secret are rejected.

Requests with the correct test secret can continue.

Unauthorized requests do not reach AI processing.

Security practices for credentials, PII, screenshots, documentation,
and GitHub publication are explicitly addressed.

This demonstrates the transition from a basic automation workflow toward
a more secure and trustworthy automation system.

## Learning Outcome

Through this task, the following security concepts were practiced:

API key and credential protection

Secret management

Authentication

Authorization concepts

Least privilege

Webhook security

PII awareness

Secure documentation

Secure GitHub practices

Error handling for unauthorized requests

Security testing

## References

n8n Credentials Documentation

OWASP API Security Top 10

n8n Webhook and Respond to Webhook documentation
