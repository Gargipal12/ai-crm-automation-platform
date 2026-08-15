# AI CRM & Sales Automation Platform

> n8n workflow automation (Sales & Customer Relationship Management)

An AI-powered CRM and sales automation platform that captures leads from a webhook intake, scores them for quality and intent using Google Gemini, assigns them fairly to sales representatives based on live workload, automatically follows up on stale leads, and centrally logs every notification and error for auditability. Built entirely in [n8n](https://n8n.io/), self-hosted via Docker, using Google Sheets as a lightweight CRM data store.

## Architecture

Five independent, interconnected workflows share a single Google Sheets document as the CRM data store. WF1 and WF2 chain directly as sub-workflow calls; WF4 and WF6 run on their own triggers, reading and writing the same shared data — loosely coupled so any one workflow can change without breaking the rest.

flowchart TD
    subgraph WF1["WF1 — Lead Collection & Registration"]
        A1["Webhook POST"] --> A2["Edit Fields<br/>build leadID, clean data, status=New"] --> A3["Append row<br/>Leads sheet"] --> A4["Call WF2<br/>(sub-workflow)"]
    end

    subgraph WF2["WF2 — AI Lead Scoring"]
        B1["Execute Workflow Trigger"] --> B2["Basic LLM Chain<br/>Gemini 2.5 Flash"] --> B3["Structured Output Parser<br/>{score, reasoning}"] --> B4["Edit Fields<br/>recombine score + leadID"] --> B5["Append/Update<br/>score, status=Scored"]
    end

    subgraph WF3["WF3 — Sales Rep Assignment"]
        C1["Execute Workflow Trigger"] --> C2["Get Lead Details"]
        C1 --> C3["Get Reps"] --> C4["Sort by activeLeadCount"] --> C5["Limit 1"]
        C2 --> C6["Merge"]
        C5 --> C6
        C6 --> C7{"score >= 70 ?"}
        C7 -->|"TRUE"| C8["Hot Lead priority"]
        C7 -->|"FALSE"| C9["Standard priority"]
        C8 --> C10["Append/Update<br/>assignedRep, status=Assigned"]
        C9 --> C10
        C10 --> C11["Increment rep activeLeadCount"] --> C12["Log to Notifications"]
    end

    subgraph WF4["WF4 — Follow-ups & Meeting Scheduling"]
        D1["Schedule Trigger<br/>daily"] --> D2["Get rows<br/>status=Assigned"] --> D3["Loop Over Items"] --> D4["Edit Fields<br/>follow-up message"] --> D5["Log to Notifications"] --> D6["Update status=FollowUp"]
        D6 -.loop back.-> D3
    end

    subgraph WF6["WF6 — Error Handling & Logging"]
        E1["Error Trigger"] --> E2["Append row<br/>ErrorLog sheet"]
    end

    STORE[("Google Sheets<br/>Leads / Reps / Proposals<br/>Notifications / ErrorLog")]

    WF1 -->|invokes| WF2
    WF1 --> STORE
    WF2 --> STORE
    STORE -.read/write.-> WF3
    STORE -.read/write.-> WF4
    E2 --> STORE
    WF1 -.on error.-> WF6
    WF2 -.on error.-> WF6
    WF3 -.on error.-> WF6
    WF4 -.on error.-> WF6

    classDef teal fill:#d6f0ee,stroke:#0f7a70,stroke-width:2px,color:#1a1a1a
    classDef purple fill:#e6def7,stroke:#7a5fc7,stroke-width:2px,color:#1a1a1a
    classDef coral fill:#fbe0da,stroke:#d97a5f,stroke-width:2px,color:#1a1a1a
    classDef gray fill:#e8e8ea,stroke:#8a8a90,stroke-width:2px,color:#1a1a1a

    class WF1,WF2,WF3 teal
    class WF4 purple
    class WF6 coral
    class STORE gray


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
