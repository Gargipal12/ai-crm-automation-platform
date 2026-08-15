# AI CRM & Sales Automation Platform

> n8n workflow automation (Sales & Customer Relationship Management)

An AI-powered CRM and sales automation platform that captures leads from a webhook intake, scores them for quality and intent using Google Gemini, assigns them fairly to sales representatives based on live workload, automatically follows up on stale leads, and centrally logs every notification and error for auditability. Built entirely in [n8n](https://n8n.io/), self-hosted via Docker, using Google Sheets as a lightweight CRM data store.

## Architecture

Five independent, interconnected workflows share a single Google Sheets document as the CRM data store. WF1 and WF2 chain directly as sub-workflow calls; WF4 and WF6 run on their own triggers, reading and writing the same shared data — loosely coupled so any one workflow can change without breaking the rest.

## Workflows

| **Workflow** | **Trigger** | **Purpose** | **Nodes** |
|---|---|---|---:|
| **WF1** — Lead Collection & Registration | Webhook (POST) | Captures and stores a new lead | 4 |
| **WF2** — AI Lead Scoring | Called by WF1 | Scores lead quality 0–100 via Gemini | 6 |
| **WF3** — Sales Rep Assignment | Called independently | Assigns to least-busy rep, flags priority | 10 |
| **WF4** — Follow-ups & Meeting Scheduling | Schedule (daily cron) | Reminds on stale, unprogressed leads | 6 |
| **WF6** — Error Handling & Logging | Error Trigger | Central failure logging for WF1–WF4 | 2 |

**28 nodes total**, covering the complete CRM automation workflow.

## Tech Stack

- **n8n** — self-hosted via Docker, workflow orchestration
- **Google Sheets** — CRM data store (Leads, Reps, Proposals, Notifications, ErrorLog tabs)
- **Google Gemini** (`gemini-3.1-flash`) — AI lead scoring via a Basic LLM Chain + Structured Output Parser
- **Google Sheets API** (OAuth2) — read/write access to the data store
- **Docker** — local/self-hosted n8n deployment

## Repository Structure

```text
ai-crm-automation-platform/
├── workflows/                
│   ├── WF1 - Lead Collection & Registration.json
│   ├── WF2 - AI Lead Scoring.json
│   ├── WF3 - Sales Rep Assignment.json
│   ├── WF4 - Follow-ups & Meeting Scheduling.json
│   └── WF6 - Error Handling & Logging.json
│
├── Screenshots/
│   ├── WF-1.png
│   ├── WF-2.png
│   ├── WF-3.png
│   ├── WF-4.png
│   └── WF6.png
│
├── docs/
│   ├── AI_CRM_Automation_Platform_Documentation (1).docx
│   └── AI_CRM_Automation_Platform.key
│
└── README.md
