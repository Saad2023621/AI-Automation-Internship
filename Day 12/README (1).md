# Automated Lead Management System
# Demo Video
 https://mega.nz/file/9YNCzBrI#8BPtkjrXLXPq5-ZeWTaN-U7Tlo29Uvh1WmbdHuNjPlE

## Project Name

Automated Lead Management System

## Problem Statement

Manual lead handling can cause delays, inconsistent prioritization,
missed follow-ups, and unnecessary manual data entry. This workflow
automates lead intake, validation, scoring, routing, storage,
notification, and acknowledgment.

## Objective

Build an end-to-end lead-management automation that: - Accepts leads
through a webhook. - Validates required information. - Processes and
scores each lead. - Categorizes leads as High Priority, Medium Priority,
or Low Priority. - Notifies the sales team for High Priority leads. -
Saves Medium Priority leads to a Google Sheets lead database. - Adds Low
Priority leads to a follow-up/nurture list. - Sends an automatic
acknowledgment email to every valid lead.

## Workflow Architecture

``` text
Lead Submission
      |
      v
Webhook
      |
      v
Validate Lead
      |
      v
Process Lead
      |
      v
Calculate Lead Score
      |
      v
Switch / Priority Router
   /        |        \
High      Medium      Low
  |          |          |
  v          v          v
Notify     Save to    Add to Nurture
Sales      Lead DB      List
  \          |          /
   \         |         /
    ---- Merge ----
          |
          v
   Acknowledgment
       Email
```

## Technologies Used

-   n8n
-   Webhooks
-   REST/HTTP APIs
-   Google Sheets
-   Gmail
-   Postman
-   JSON
-   n8n Expressions
-   JavaScript/Code node where applicable

## Input Data

Each lead contains:

  Field       Description
  ----------- ----------------------
  `name`      Lead's name
  `email`     Lead's email
  `company`   Lead's company
  `service`   Requested service
  `budget`    Lead's stated budget

Example:

``` json
{
  "name": "Ahmed Khan",
  "email": "ahmed.khan@gmail.com",
  "company": "Tech Solutions",
  "service": "Enterprise Software",
  "budget": 120000
}
```

## Nodes Used

### Lead Webhook

Receives the incoming POST request containing lead information.

### Validate Lead

Checks that the required fields are present before processing continues.

### Process Lead

Prepares the submitted lead data for scoring and routing.

### Calculate Lead Score

Calculates the lead score and assigns a priority according to the
scoring rules implemented in the workflow. The resulting data includes
fields such as `score` and `priority`.

### Switch / Priority Router

Routes the lead to one of three branches: - High Priority - Medium
Priority - Low Priority

### Notify Sales Team

Sends an immediate Gmail notification for High Priority leads.

### Save Medium-Priority Lead

Appends Medium Priority leads to the lead database Google Sheet.

### Add Low-Priority Lead to Nurture List

Appends Low Priority leads to the follow-up/nurture list.

### Merge

Combines the priority branches before the common acknowledgment step.

### Acknowledgment Email

Sends confirmation to the lead using the submitted email address:

``` text
{{ $json.email }}
```

## Setup Instructions

1.  Open the n8n workspace.
2.  Create the workflow **Sales - Automated Lead Management - v1**.
3.  Add and connect the nodes according to the architecture above.
4.  Configure the Webhook for POST requests.
5.  Configure Google Sheets credentials.
6.  Configure Gmail credentials.
7.  Create the lead database and nurture-list sheets.
8.  Configure validation and scoring rules.
9.  Configure the Switch with High, Medium, and Low Priority outputs.
10. Configure the sales notification email.
11. Configure the Google Sheets append operations.
12. Configure the acknowledgment email to use `{{ $json.email }}`.
13. Test with Postman.
14. Verify routing, sheet updates, and emails.
15. Activate/publish the workflow after successful testing.

## Credentials Required

Credential names only; never store credential values: - Google Sheets
account - Gmail account - n8n workspace/account

## Workflow Explanation

A new lead submits its information through the webhook. The workflow
validates the required fields, processes the data, and calculates a lead
score and priority.

The Switch node routes each lead: - **High Priority:** notify the sales
team immediately. - **Medium Priority:** save the lead to the lead
database. - **Low Priority:** save the lead to the follow-up/nurture
list.

The branches are then combined and a common acknowledgment email is sent
to the lead.

## Priority Routing

  Priority          Action
  ----------------- -------------------------------
  High Priority     Immediate sales notification
  Medium Priority   Save to lead database
  Low Priority      Add to follow-up/nurture list
  All valid leads   Send acknowledgment email

## Sample API Request

**Method:** `POST`

**Header:**

``` text
Content-Type: application/json
```

**Body:**

``` json
{
  "name": "Ahmed Khan",
  "email": "ahmed.khan@gmail.com",
  "company": "Tech Solutions",
  "service": "Enterprise Software",
  "budget": 120000
}
```

Do not include private webhook URLs, passwords, API keys, or tokens in
this README.

## Test Cases

### Test Case 1 --- High Priority

Submit a lead whose data produces a High Priority result under the
configured scoring rules.

Expected: - Lead is validated. - High Priority is assigned. - Sales
notification is sent. - Acknowledgment email is sent.

### Test Case 2 --- Medium Priority

Submit a lead whose data produces a Medium Priority result.

Expected: - Lead is routed only to Medium Priority. - Lead is appended
to the lead database. - Acknowledgment email is sent.

### Test Case 3 --- Low Priority

Submit a lead whose data produces a Low Priority result.

Expected: - Lead is routed only to Low Priority. - Lead is appended to
the nurture/follow-up list. - Acknowledgment email is sent.

### Test Case 4 --- Missing Required Field

Submit a lead with a required field missing.

Expected: - Validation identifies the incomplete record. - The record
does not continue through normal priority processing.

## Error Handling

-   Required fields are validated before processing.
-   Invalid records are stopped before normal routing.
-   Switch rules explicitly match the three priority values.
-   Credentials are stored securely in n8n.
-   Secrets are never hard-coded.
-   Failed executions should be reviewed in n8n.

## Known Limitations

-   The scoring rules are designed for this internship exercise and are
    not a production sales-scoring model.
-   A test email recipient may be used when a dedicated sales-team
    mailbox is unavailable.
-   Google Sheets is used as the demonstration data store.
-   The workflow depends on external services such as Gmail and Google
    Sheets.

## Future Improvements

-   Add webhook authentication or a secret header.
-   Add duplicate-lead detection.
-   Use a production database or CRM.
-   Add more sophisticated multi-factor lead scoring.
-   Add automated nurture/follow-up sequences.
-   Add retry and error-notification workflows.
-   Add monitoring and analytics.
-   Add AI-based lead qualification.

## Security Notes

Never include passwords, API keys, access tokens, or other secrets in
the project files or screenshots. Use n8n credentials for connected
services and use test data instead of sensitive customer information.

## Submission Structure

``` text
automated-lead-management-system/
│
├── workflow/
│   └── workflow.json
│
├── screenshots/
│   ├── 01-webhook-input.png
│   ├── 02-validation.png
│   ├── 03-lead-scoring.png
│   ├── 04-switch-routing.png
│   ├── 05-high-priority-notification.png
│   ├── 06-medium-priority-sheet.png
│   ├── 07-low-priority-nurture-sheet.png
│   └── 08-acknowledgment-email.png
│
├── sample-data/
│   └── sample-api-request.json
│
└── README.md
```

The complete project folder can be compressed into a ZIP file and
submitted to the internship manager through the required submission
channel.
