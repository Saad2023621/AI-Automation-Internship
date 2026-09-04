# AI-Automation

# HR - Candidate Screening - v1

## Problem Statement

Manually screening internship applications is time-consuming and prone to human error. Recruiters must review candidate information, categorize applicants, update records, and notify candidates individually.

---

## Objective

To automate the candidate screening process using n8n by reading candidate information from Google Sheets, validating the data, applying screening rules, updating the screening result, and sending personalized email notifications.

---

## Workflow Architecture

Manual Trigger
↓

Google Sheets (Fetch Candidate Records)
↓

Validate Required Information

↓

Apply Eligibility Rules

↓

Categorize Candidate

↓

Update Google Sheet

↓

Send Personalized Email

---

## Technologies Used

- n8n
- Google Sheets
- Gmail
- Google OAuth

---

## Nodes Used

- Manual Trigger
- Google Sheets (Get Rows)
- IF
- Edit Fields
- Google Sheets (Update Row)
- Gmail

---

## Setup Instructions

1. Create a Google Sheet with candidate information.
2. Connect Google Sheets credentials.
3. Connect Gmail credentials.
4. Import the workflow.json into n8n.
5. Execute the workflow.

---

## Credentials Required

- Google Sheets OAuth2
- Gmail OAuth2

(No passwords or API keys are included.)

---

## Workflow Explanation

The workflow retrieves candidate records from Google Sheets and validates that all required fields are present. It then applies screening rules based on degree, experience, and availability. Each candidate is categorized as Shortlisted, Needs Review, or Not Eligible. The workflow updates the corresponding Google Sheet row with the screening result and sends a personalized email notification to the candidate.

---

## Test Cases

### Test Case 1

Experience: 2 Years

Availability: Yes

Degree: Data Science

Expected Result:

Shortlisted

---

### Test Case 2

Experience: 0 Year

Availability: Yes

Degree: BBA

Expected Result:

Not Eligible

---

### Test Case 3

Experience: 3 Years

Availability: Yes

Degree: BS Software Engineering

Expected Result:

Shortlisted

---

## Error Handling

- Invalid records are ignored.
- Empty mandatory fields are filtered out.
- Workflow only processes valid candidate records.

---

## Known Limitations

- Screening rules are fixed.
- Only Google Sheets is supported.
- Email templates are static.

---

## Future Improvements

- AI-powered resume screening
- Resume PDF parsing
- HR dashboard
- Slack notifications
- Interview scheduling
- Database integration
