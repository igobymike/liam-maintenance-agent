<p align="left">
  <img src="https://img.shields.io/badge/Status-Active%20Development-22c55e?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Built%20by-BAITEKS-0ea5e9?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Stack-Copilot%20Studio-6366f1?style=for-the-badge" />
</p>

# LIAM — LaPorte Intelligent Asset Management

**AI-powered maintenance planning assistant for industrial plants.**

---

## The Problem

Maintenance planners at industrial plants spend **2–4 hours writing a single job plan**. Every time a pump fails, a valve leaks, or a motor burns out, a planner has to manually research what was done last time, piece together repair steps from memory and scattered SAP records, estimate labor and duration, and format the whole thing for the crew. Meanwhile:

- **Historical work order data sits unused in SAP** — thousands of completed jobs with detailed repair procedures, but no way to search or synthesize them
- **Institutional knowledge walks out the door** when experienced planners retire — the guy who knew exactly how to overhaul P-102 took that knowledge with him
- **Inconsistent procedures across planners** lead to rework, missed steps, and quality issues
- **SAP task lists exist but are outdated** — nobody maintains them because the process is painful

The result: poorly planned jobs take **30–50% longer** than well-planned ones. At a chemical plant running 24/7, that's real money.

---

## The Solution

LIAM searches historical work orders exported from SAP PM, finds similar completed jobs on the same equipment, extracts the actual repair steps that were used, normalizes them into a structured format, estimates labor and duration from historical data, and delivers a ready-to-use job plan — in seconds, not hours. Planners interact with LIAM through Microsoft Teams using natural language. No SAP access required.

---

## Key Features

- **Instant job plan generation** — Give LIAM an equipment ID and failure description. It searches historical WOs, extracts and normalizes repair steps, estimates duration and crew size, and returns a structured job plan ready for the crew
- **Proactive job plan library** — Pre-built job plans for the top failure modes on known equipment. Planners retrieve them instantly without waiting for a new work order
- **Natural language Q&A** — Ask maintenance history questions in plain English: *"When did P-102 last have a seal failure?"*, *"How many pump overhauls did we do in Unit 100 this year?"*
- **Daily maintenance summaries** — Automated plain-English summary of yesterday's maintenance activity delivered to Teams every morning at 6 AM
- **PM interval recommendations** — Flags equipment where failure frequency exceeds the current PM schedule and recommends interval adjustments
- **Zero SAP integration required** — Works entirely from Excel exports uploaded to SharePoint. No SAP API, no write-back, no IT gatekeeping

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│  Layer 1 — User Interaction                     │
│  Teams Chat • Copilot Studio Chat Canvas        │
├─────────────────────────────────────────────────┤
│  Layer 2 — AI Copilot Agent                     │
│  Copilot Studio • Skill Routing • Intent Detect │
├─────────────────────────────────────────────────┤
│  Layer 3 — Processing & Analysis                │
│  Job Plan Engine • WO Intelligence • Reporting  │
├─────────────────────────────────────────────────┤
│  Layer 4 — Data Integration                     │
│  Power Automate Flows • SharePoint Connector    │
├─────────────────────────────────────────────────┤
│  Layer 5 — Data Storage                         │
│  SharePoint Data Hub (7 Excel Files)            │
│  JobPlanLibrary.xlsx                            │
└─────────────────────────────────────────────────┘
```

Full architecture specification: **[docs/LIAM-ARCHITECTURE.md](docs/LIAM-ARCHITECTURE.md)**

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Agent | Microsoft Copilot Studio | Conversational AI agent, skill routing, intent detection |
| Automation | Power Automate | Data flows, scheduled reports, Excel parsing |
| Data | SharePoint Online | Data hub — 7 Excel files exported from SAP PM |
| Source | SAP PM (IW47, IP19) | Batch export only — no API, no write-back |
| Delivery | Microsoft Teams | Chat interface, daily summaries, alerts |
| Reporting | Power BI | Dashboard visualizations (future) |

---

## Build Status

| Phase | Description | Status |
|-------|------------|--------|
| Phase 1 | Data Foundation — SAP export, SharePoint data hub | Not Started |
| Phase 2 | Copilot Agent Setup — basic Q&A on WO history | Not Started |
| Phase 3 | Job Plan Generation Engine — core skill | Not Started |
| Phase 4 | Automation & Reporting — daily summaries, alerts | Not Started |
| Phase 5 | Job Plan Library Buildout — top 20 failure modes | Not Started |

---

## Documentation

- **[Architecture Specification](docs/LIAM-ARCHITECTURE.md)** — Full system architecture (14 sections, 600+ lines)
- **[Functional Requirements](docs/FR-TR.md)** — Requirements traceability
- **[Build Gameplan](docs/GAMEPLAN.md)** — Phase-by-phase implementation plan

---

## Author

**Mike Birklett** — [BAITEKS](https://baiteks.com) — Business & AI Technology Solutions

AI Engineer • Industrial AI • Maintenance Intelligence

[![LinkedIn](https://img.shields.io/badge/LinkedIn-michaelbirklett-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/michaelbirklett)
[![Website](https://img.shields.io/badge/Web-baiteks.com-0ea5e9?style=flat-square)](https://baiteks.com)
