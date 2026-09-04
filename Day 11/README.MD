# Sales - Lead Processing - v1

## Project Name

Sales - Lead Processing - v1

---

## Problem Statement

Sales teams often receive new leads through forms or websites and need to manually validate, enrich, store, and notify the relevant team members about each lead.

This manual process can be time-consuming and may result in inconsistent data handling or delayed notifications.

This project automates the lead processing pipeline using n8n.

---

## Objective

The objective is to build an automated lead processing workflow that:

1. Receives a new lead through a webhook.
2. Validates the submitted lead information.
3. Enriches the lead using an external API.
4. Stores the processed lead in Google Sheets.
5. Sends a notification email to the sales team.

---

## Workflow Architecture

The workflow follows this architecture:

Webhook
↓
Validate Lead Information
↓
Enrich Lead Data using HTTP Request
↓
Save Enriched Lead to Google Sheets
↓
Notify Sales Team through Gmail

### Workflow Flow

```text
New Lead
   ↓
Receive New Lead (Webhook)
   ↓
Validate Lead Information (IF)
   ↓
Enrich Lead Data (HTTP Request)
   ↓
Save Enriched Lead (Google Sheets)
   ↓
Notify Sales Team (Gmail)