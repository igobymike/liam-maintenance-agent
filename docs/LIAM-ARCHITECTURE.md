# LIAM — Architecture Specification

**LaPorte Intelligent Asset Management**
**Version:** 1.0
**Date:** March 2026
**Author:** Mike Birklett — BAITEKS

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [What LIAM Is NOT](#3-what-liam-is-not)
4. [System Architecture](#4-system-architecture)
5. [Data Architecture — SharePoint Data Hub](#5-data-architecture--sharepoint-data-hub)
6. [Job Plan Generation Engine](#6-job-plan-generation-engine)
7. [Proactive Job Plan Library](#7-proactive-job-plan-library)
8. [Additional Skills](#8-additional-skills)
9. [Copilot Studio Architecture](#9-copilot-studio-architecture)
10. [Power Automate Flows](#10-power-automate-flows)
11. [Security & Access](#11-security--access)
12. [Implementation Roadmap](#12-implementation-roadmap)
13. [Integration with BAITEKS Ecosystem](#13-integration-with-baiteks-ecosystem)
14. [Success Metrics](#14-success-metrics)

---

## 1. Executive Summary

LIAM (LaPorte Intelligent Asset Management) is an AI-powered maintenance planning assistant built on Microsoft Copilot Studio. It serves maintenance planners at industrial chemical plants by automating the most time-consuming part of their job: writing structured job plans for corrective maintenance work orders.

LIAM connects to historical maintenance data exported from SAP PM and stored in SharePoint Excel files. When a planner receives a new work order — for example, "overhaul pump P-102, mechanical seal failure" — LIAM searches thousands of historical work orders for similar completed jobs on the same equipment, extracts the repair steps that were actually performed, normalizes them into a structured format with standard safety steps and logical ordering, estimates labor duration and crew size from historical patterns, and delivers a ready-to-use job plan through Microsoft Teams. What previously took a planner 2–4 hours now takes under 15 minutes, including review and adjustment.

Beyond reactive job plan generation, LIAM maintains a proactive library of pre-built job plans for the most common failure modes at the plant, answers natural language questions about maintenance history, delivers daily maintenance summaries to Teams, and flags equipment where failure frequency suggests the preventive maintenance interval needs adjustment.

LIAM is part of the BAITEKS industrial AI ecosystem alongside P6 Intelligence (Primavera P6 schedule analysis). Together, they cover the planning lifecycle: LIAM generates the job plan (what work to do), and P6 Intelligence handles schedule optimization (when to do it). LIAM operates exclusively within the Microsoft 365 stack — Copilot Studio, Power Automate, SharePoint, and Teams — with no direct SAP integration. All SAP data is consumed via batch Excel exports, making LIAM deployable at sites where SAP API access is restricted or unavailable.

---

## 2. Problem Statement

### 2.1 Manual Job Plan Creation Is Slow and Inconsistent

At a typical industrial plant, maintenance planners write 5–15 job plans per week. Each job plan requires the planner to:

1. **Research history** — Look up the equipment in SAP, find previous work orders, read through long text descriptions and completion notes to understand what was done before
2. **Extract repair steps** — Mentally parse unstructured SAP long text (which mixes narrative, abbreviations, and P6 notation) into discrete, ordered work steps
3. **Estimate labor** — Determine crew size and duration based on experience and historical patterns
4. **Format the plan** — Write out each operation with work center assignment, short and long text descriptions, and duration estimates
5. **Add safety requirements** — Include lockout/tagout procedures, confined space entry if applicable, hot work permits, and other safety steps

This process takes **2–4 hours per job plan** for an experienced planner. For a junior planner without institutional knowledge of the equipment, it can take an entire day.

### 2.2 Historical Data Is Rich but Inaccessible

SAP PM stores thousands of completed work orders with detailed repair procedures buried in long text fields. This data represents years of accumulated maintenance knowledge — what was done, how long it took, what complications arose, what parts were used. But accessing it requires:

- Running SAP transactions (IW47, IW39) with specific selection criteria
- Reading through individual work orders one at a time
- Mentally correlating information across orders, notifications, and task lists

There is no search interface, no summarization capability, and no way to ask "what did we do last time this equipment failed?" The data exists but is effectively invisible to planners who need it most.

### 2.3 Institutional Knowledge Loss

When an experienced planner retires or transfers, decades of equipment-specific knowledge leave with them. This planner knew:

- Which pumps are problematic and what the real fix is (vs. the SAP task list)
- How long a job actually takes (vs. the standard time in the system)
- Which failure modes are recurring and what the root cause usually is
- What special tools are needed, what access issues exist, what gotchas to watch for

This knowledge is not documented anywhere. It exists only in the planner's head. When they leave, the next planner starts from scratch, producing inferior job plans that lead to longer job durations and more rework.

### 2.4 Inconsistent Procedures

Without a standardized job plan library, each planner writes procedures differently. Planner A writes detailed step-by-step instructions; Planner B writes "overhaul pump per OEM manual." The result:

- Crews receive inconsistent quality of planning support
- The same job on the same equipment gets planned differently each time
- Safety steps may be included by one planner and omitted by another
- Duration estimates vary widely for identical work scopes

### 2.5 Outdated SAP Task Lists

SAP has a task list system designed for exactly this purpose — standardized step-by-step procedures attached to equipment. In practice, these task lists are rarely maintained because:

- Updating them requires SAP access and knowledge of SAP configuration
- The update process is manual and tedious
- There is no feedback loop from completed WOs back to task lists
- Many task lists were created during plant commissioning and never revised

The result is that task lists exist in SAP but are so outdated that planners ignore them and write their own procedures from scratch.

### 2.6 Dollar Impact

At a chemical plant running continuous operations:

- A **well-planned job** with clear procedures, correct parts staged, and accurate duration estimates completes on time with minimal rework
- A **poorly planned job** takes **30–50% longer** due to missing information, wrong parts ordered, unclear procedures requiring crew to stop and ask questions, and unplanned safety requirements
- At fully burdened craft labor rates of **$85–120/hour** and typical crew sizes of 2–4 people, a poorly planned 8-hour job can easily cost **$500–1,500 more** than the same job with good planning
- Across 250–400 corrective work orders per year, the aggregate cost of poor planning is **$125,000–600,000 annually** in excess labor alone — not counting production losses from extended equipment downtime

---

## 3. What LIAM Is NOT

To prevent scope creep and set correct expectations, here is what LIAM explicitly does not do:

### 3.1 Not a Real-Time SAP Integration

LIAM does not connect to SAP via API, RFC, BAPI, or any other interface. All SAP data is consumed through **batch Excel exports** uploaded to SharePoint. This is a deliberate design decision:

- Kuraray SAP is read-only for the maintenance planning team
- SAP API access requires IT governance approval that takes months
- Excel export is available today via standard SAP transactions (IW47, IP19, IE05)
- The batch approach is sufficient — maintenance data does not change in real time

LIAM will **never write back to SAP**. Job plans generated by LIAM are delivered to planners, who manually enter them into SAP if they choose to use them.

### 3.2 Not a Failure Prediction Engine

LIAM does not predict equipment failures. Failure pattern detection, reliability analysis, and predictive maintenance are handled by **Pinnacle Reliability Newton**, a separate tool already deployed at the plant. LIAM's role is downstream: once a failure has occurred (or is anticipated), LIAM helps the planner build the job plan to fix it.

The one exception is LIAM's PM interval recommendation feature, which flags equipment where failure frequency exceeds the preventive maintenance interval. This is simple frequency analysis, not predictive modeling.

### 3.3 Not a Scheduling Tool

LIAM generates job plans — it does not schedule them. Scheduling (resource leveling, critical path analysis, calendar management) is handled by **P6 Intelligence**, the companion BAITEKS tool for Primavera P6. The workflow is: LIAM generates the plan (what to do), P6 Intelligence optimizes the schedule (when to do it).

### 3.4 Not a Replacement for Planners

LIAM is an assistant, not an automation. It removes the tedious research and formatting work so planners can focus on judgment calls:

- Does this job plan look right for this specific situation?
- Do we need additional safety precautions given current plant conditions?
- Should we combine this with other work on the same equipment?
- Is the duration estimate realistic for our crew's skill level?

The planner always reviews and approves the output before it goes to the crew.

---

## 4. System Architecture

### 4.1 Five-Layer Architecture

LIAM is built on a five-layer architecture that separates user interaction, AI processing, business logic, data integration, and storage.

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LAYER 1 — USER INTERACTION                                      │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                      │
│  │  Microsoft Teams  │  │  Copilot Studio  │                      │
│  │  Chat Interface   │  │  Chat Canvas     │                      │
│  └────────┬─────────┘  └────────┬─────────┘                      │
│           │                      │                                │
│           └──────────┬───────────┘                                │
│                      ▼                                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LAYER 2 — AI COPILOT AGENT                                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐        │
│  │  Microsoft Copilot Studio                            │        │
│  │                                                      │        │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐    │        │
│  │  │  Intent      │ │  Skill      │ │  Response    │    │        │
│  │  │  Detection   │ │  Routing    │ │  Generation  │    │        │
│  │  └─────────────┘ └─────────────┘ └─────────────┘    │        │
│  │                                                      │        │
│  │  Skills:                                             │        │
│  │  • GenerateJobPlan  • AskMaintenance                 │        │
│  │  • GetLibraryPlan   • DailySummary                   │        │
│  │  • PMRecommendation                                  │        │
│  └──────────────────────────────────────────────────────┘        │
│                      │                                            │
│                      ▼                                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LAYER 3 — PROCESSING & ANALYSIS                                 │
│                                                                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐              │
│  │  Job Plan     │ │  WO          │ │  Reporting   │              │
│  │  Generation   │ │  Intelligence│ │  Engine      │              │
│  │  Engine       │ │  Engine      │ │              │              │
│  └──────────────┘ └──────────────┘ └──────────────┘              │
│  ┌──────────────┐                                                │
│  │  PM           │                                                │
│  │  Recommendation│                                              │
│  │  Engine       │                                                │
│  └──────────────┘                                                │
│                      │                                            │
│                      ▼                                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LAYER 4 — DATA INTEGRATION                                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐        │
│  │  Power Automate Flows                                │        │
│  │                                                      │        │
│  │  • SharePoint Connector (read Excel files)           │        │
│  │  • Excel Table Parser (structured data extraction)   │        │
│  │  • Copilot Action Triggers (on-demand queries)       │        │
│  │  • Schedule Triggers (daily/weekly automation)        │        │
│  │  • Teams Connector (message delivery)                │        │
│  └──────────────────────────────────────────────────────┘        │
│                      │                                            │
│                      ▼                                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LAYER 5 — DATA STORAGE                                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────┐        │
│  │  SharePoint Document Library — LIAM Data Hub          │        │
│  │                                                      │        │
│  │  ┌────────────────┐ ┌────────────────┐               │        │
│  │  │ WorkOrders     │ │ TaskLists      │               │        │
│  │  │ .xlsx          │ │ .xlsx          │               │        │
│  │  └────────────────┘ └────────────────┘               │        │
│  │  ┌────────────────┐ ┌────────────────┐               │        │
│  │  │ Notifications  │ │ Preventive     │               │        │
│  │  │ .xlsx          │ │ Maintenance.xlsx│              │        │
│  │  └────────────────┘ └────────────────┘               │        │
│  │  ┌────────────────┐ ┌────────────────┐               │        │
│  │  │ Equipment      │ │ FailureCodes   │               │        │
│  │  │ Master.xlsx    │ │ .xlsx          │               │        │
│  │  └────────────────┘ └────────────────┘               │        │
│  │  ┌────────────────┐                                  │        │
│  │  │ JobPlanLibrary │                                  │        │
│  │  │ .xlsx          │                                  │        │
│  │  └────────────────┘                                  │        │
│  └──────────────────────────────────────────────────────┘        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Layer Descriptions

**Layer 1 — User Interaction:** Planners interact with LIAM through Microsoft Teams (primary) or the Copilot Studio embedded chat canvas (secondary). Teams is the preferred channel because planners already use it daily and it supports rich card formatting for job plan output. The chat canvas provides an alternative for users who prefer a standalone interface.

**Layer 2 — AI Copilot Agent:** The Copilot Studio agent handles natural language understanding, intent detection, skill routing, and response generation. It determines what the planner is asking for (generate a job plan, answer a question, retrieve a library plan) and routes the request to the appropriate skill. The agent maintains conversation context so planners can ask follow-up questions.

**Layer 3 — Processing & Analysis:** Four engines handle the core business logic:
- **Job Plan Generation Engine** — The primary skill. Searches, extracts, normalizes, and assembles job plans from historical data.
- **WO Intelligence Engine** — Answers natural language questions about maintenance history, failure patterns, and repair timelines.
- **Reporting Engine** — Generates daily summaries, weekly backlogs, and ad-hoc reports.
- **PM Recommendation Engine** — Analyzes failure frequency vs. PM intervals and flags mismatches.

**Layer 4 — Data Integration:** Power Automate flows handle all data movement between SharePoint, the AI engines, and output destinations. Flows read Excel files from SharePoint, parse them into structured data, pass data to the Copilot agent for processing, and deliver results to Teams channels or individual users.

**Layer 5 — Data Storage:** All data lives in a SharePoint document library as Excel files. Seven files form the data hub, each exported from a specific SAP PM transaction. The JobPlanLibrary.xlsx is the only file that LIAM writes to (with lead planner approval). All others are read-only, refreshed via periodic batch export from SAP.

---

## 5. Data Architecture — SharePoint Data Hub

### 5.1 WorkOrders.xlsx

**Purpose:** Historical work order records from SAP PM. The primary data source for job plan generation.

**SAP Source:** Transaction IW47 (work order list), batch export

**Update Frequency:** Weekly (automated SAP batch report, uploaded to SharePoint)

**Read Method:** Power Automate flow → SharePoint connector → List Rows in Excel Table

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| WO_Number | Text | SAP work order number (12 digits) |
| WO_Type | Text | Order type (PM01=Corrective, PM02=Preventive, PM03=Emergency) |
| Equipment_ID | Text | SAP equipment number (e.g., P-102, V-301) |
| Equipment_Desc | Text | Equipment description from master data |
| Functional_Location | Text | SAP functional location (plant area hierarchy) |
| Work_Center | Text | Responsible work center (VN-PM, VN-EC) |
| Priority | Text | Work order priority (1=Emergency, 2=Urgent, 3=Normal, 4=Planned) |
| Status | Text | System status (CRTD, REL, TECO, CLSD) |
| Short_Text | Text | Work order short description (40 chars) |
| Long_Text | Text | Detailed work description and procedures (up to 10,000 chars) |
| Completion_Notes | Text | Post-completion notes from crew/planner |
| Failure_Code | Text | SAP failure code (catalog/code group/code) |
| Failure_Desc | Text | Failure code description |
| Planned_Start | Date | Planned start date |
| Planned_End | Date | Planned end date |
| Actual_Start | Date | Actual start date |
| Actual_End | Date | Actual end date |
| Planned_Duration | Number | Planned duration in hours |
| Actual_Duration | Number | Actual duration in hours |
| Crew_Size | Number | Number of craft workers assigned |
| Created_By | Text | Planner who created the WO |
| Created_Date | Date | WO creation date |
| Notification | Text | Linked SAP notification number |

### 5.2 TaskLists.xlsx

**Purpose:** SAP standard task list operations — predefined step-by-step procedures attached to equipment or functional locations.

**SAP Source:** Transactions IA05/IA06 (task list display), batch export

**Update Frequency:** Monthly (task lists change infrequently)

**Read Method:** Power Automate flow → SharePoint connector → List Rows in Excel Table

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| TaskList_ID | Text | SAP task list number |
| TaskList_Type | Text | Type (A=Equipment, T=Functional Location) |
| Group_Counter | Text | Task list group counter |
| Operation_Number | Text | Operation sequence number (0010, 0020, etc.) |
| Work_Center | Text | Work center for this operation |
| Short_Text | Text | Operation short description |
| Long_Text | Text | Detailed operation instructions |
| Duration | Number | Standard duration in hours |
| Duration_Unit | Text | Unit of measure (H=Hours, MIN=Minutes) |
| Crew_Size | Number | Number of workers for this operation |
| Equipment_ID | Text | Linked equipment number |
| Functional_Location | Text | Linked functional location |
| Status | Text | Task list status (Active, Inactive) |
| Last_Modified | Date | Last modification date |

### 5.3 Notifications.xlsx

**Purpose:** SAP maintenance notifications — problem reports, failure descriptions, and malfunction records that trigger work orders.

**SAP Source:** Transaction IW28 (notification list), batch export

**Update Frequency:** Weekly (aligned with WorkOrders.xlsx refresh)

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| Notification_Number | Text | SAP notification number |
| Notification_Type | Text | Type (M1=Malfunction, M2=Maintenance Request, M3=Activity Report) |
| Equipment_ID | Text | Equipment number |
| Equipment_Desc | Text | Equipment description |
| Functional_Location | Text | Functional location |
| Short_Text | Text | Problem short description |
| Long_Text | Text | Detailed problem description |
| Failure_Code | Text | Catalog/code group/code |
| Failure_Desc | Text | Failure code description |
| Damage_Code | Text | Damage code |
| Cause_Code | Text | Cause code |
| Priority | Text | Notification priority |
| Status | Text | Status (OSNO=Outstanding, NOPR=In Process, NOCO=Completed) |
| Created_Date | Date | Creation date |
| Created_By | Text | Creator |
| WO_Number | Text | Linked work order (if exists) |
| Required_Start | Date | Required start date |
| Required_End | Date | Required end date |

### 5.4 PreventiveMaintenance.xlsx

**Purpose:** PM schedule data — maintenance plans, intervals, and call dates for recurring preventive work.

**SAP Source:** Transaction IP19 (maintenance plan list), batch export

**Update Frequency:** Monthly

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| PM_Plan_Number | Text | SAP maintenance plan number |
| PM_Plan_Desc | Text | Plan description |
| Plan_Type | Text | Type (Time-based, Performance-based) |
| Equipment_ID | Text | Equipment number |
| Equipment_Desc | Text | Equipment description |
| Functional_Location | Text | Functional location |
| Cycle_Duration | Number | Interval value |
| Cycle_Unit | Text | Interval unit (MON=Months, WK=Weeks, DAY=Days) |
| Task_List_ID | Text | Linked task list |
| Work_Center | Text | Responsible work center |
| Last_Call_Date | Date | Last time this PM was triggered |
| Next_Call_Date | Date | Next scheduled call date |
| Status | Text | Plan status (Active, Inactive, Deleted) |
| Priority | Text | Default WO priority |

### 5.5 EquipmentMaster.xlsx

**Purpose:** Equipment hierarchy and master data — the canonical reference for all equipment at the plant.

**SAP Source:** Transaction IE05 (equipment list), batch export

**Update Frequency:** Monthly (equipment master changes infrequently)

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| Equipment_ID | Text | SAP equipment number |
| Equipment_Desc | Text | Equipment description |
| Equipment_Category | Text | Category (M=Machines, E=Electrical, I=Instrumentation) |
| Equipment_Class | Text | Classification (Centrifugal Pump, Gate Valve, Motor, etc.) |
| Manufacturer | Text | OEM manufacturer |
| Model | Text | Model number |
| Serial_Number | Text | Serial number |
| Functional_Location | Text | Installed functional location |
| Plant_Section | Text | Plant area (Unit 100, Unit 200, Utilities, etc.) |
| Work_Center | Text | Responsible work center |
| Construction_Year | Text | Year of manufacture |
| Install_Date | Date | Installation date |
| Status | Text | Equipment status (INST=Installed, DLFL=Decommissioned) |
| Criticality | Text | Criticality rating (A=Critical, B=Essential, C=General) |
| Weight | Number | Equipment weight (for rigging planning) |
| Weight_Unit | Text | Weight unit (KG, LB) |

### 5.6 FailureCodes.xlsx

**Purpose:** SAP failure code catalog — standardized codes for classifying equipment failures, damage types, and root causes.

**SAP Source:** Transaction OIS2 (catalog maintenance), batch export

**Update Frequency:** Quarterly (failure codes change rarely)

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| Catalog_Type | Text | Catalog type (5=Damage, B=Cause, C=Object Part) |
| Code_Group | Text | Code group |
| Code | Text | Individual code |
| Code_Desc | Text | Code description |
| Parent_Group | Text | Parent code group (for hierarchy) |
| Status | Text | Active/Inactive |

### 5.7 JobPlanLibrary.xlsx

**Purpose:** LIAM-generated and planner-curated job plans. The only file LIAM writes to (with approval).

**SAP Source:** None — this is LIAM-native data

**Update Frequency:** As new plans are generated and approved

**Read/Write Method:** Power Automate flow → SharePoint connector → Add/Update Row in Excel Table

**Fields:**

| Field | Type | Description |
|-------|------|-------------|
| Plan_ID | Text | Unique job plan identifier (JP-001, JP-002, etc.) |
| Equipment_Class | Text | Equipment classification this plan applies to |
| Failure_Mode | Text | Failure mode this plan addresses |
| Plan_Title | Text | Descriptive title (e.g., "Centrifugal Pump Mechanical Seal Replacement") |
| Version | Number | Plan version number |
| Status | Text | Draft, Approved, Superseded |
| Operations | Text | JSON array of operation objects (see Section 6.5 for schema) |
| Total_Duration | Number | Estimated total duration in hours |
| Crew_Size | Number | Recommended crew size |
| Safety_Requirements | Text | Required permits and safety procedures |
| Special_Tools | Text | Special tools or equipment needed |
| Source_WOs | Text | Comma-separated list of source WO numbers |
| Created_Date | Date | Plan creation date |
| Created_By | Text | Creator (LIAM or planner name) |
| Approved_By | Text | Lead planner who approved |
| Approved_Date | Date | Approval date |
| Last_Used | Date | Last time this plan was retrieved |
| Use_Count | Number | Number of times plan has been retrieved |

---

## 6. Job Plan Generation Engine

This is the core skill of LIAM — the primary reason the system exists. This section describes the full end-to-end process of generating a structured job plan from historical work order data.

### 6.1 Input

The planner provides two pieces of information:

1. **Equipment ID** — The SAP equipment number (e.g., P-102)
2. **Failure Description** — A natural language description of the problem (e.g., "mechanical seal leak", "bearing failure", "motor tripping on overload")

Optionally, the planner can also provide:
- **Work order number** — If a WO already exists, LIAM pulls the equipment and failure info from it
- **Failure code** — If the planner knows the SAP failure code, this improves search precision
- **Scope guidance** — "This is a full overhaul" vs. "just replace the seal" to guide step selection

### 6.2 Step 1 — WO Search & Retrieval

**Objective:** Find 3–7 relevant historical work orders to use as source material for the job plan.

**Search Strategy (cascading):**

```
Search Level 1: Same Equipment + Same Failure Code
  → Filter WorkOrders.xlsx where Equipment_ID = input AND Failure_Code = input
  → Filter to Status = TECO (technically complete) only
  → If >= 3 results → proceed to ranking

Search Level 2: Same Equipment + Any Failure
  → If Level 1 returned < 3 results
  → Filter WorkOrders.xlsx where Equipment_ID = input AND Status = TECO
  → If >= 3 results → proceed to ranking

Search Level 3: Same Equipment Class + Same Failure Code
  → If Level 2 returned < 3 results
  → Look up Equipment_Class from EquipmentMaster.xlsx
  → Filter WorkOrders.xlsx where Equipment_Class matches AND Failure_Code = input
  → Filter to Status = TECO
  → Proceed to ranking with whatever results are available

Search Level 4: Same Equipment Class + Keyword Match
  → If Level 3 returned < 3 results
  → Search Short_Text and Long_Text for keywords from the failure description
  → Filter to same Equipment_Class and Status = TECO
  → Proceed to ranking
```

**Ranking Criteria (applied to search results):**

| Priority | Criterion | Rationale |
|----------|----------|-----------|
| 1 | Same failure code | Most relevant — same failure = same repair |
| 2 | Most recent | More recent procedures reflect current best practices |
| 3 | Longest actual duration | More complex jobs have more detailed long text — better source material |
| 4 | Has completion notes | WOs with completion notes provide post-job insights |

**Selection:** Take the top 3–7 WOs after ranking. If fewer than 3 are available after all search levels, LIAM informs the planner that limited historical data exists and generates a best-effort plan with a disclaimer.

**Output of this step:** A set of 3–7 historical WO records with full Long_Text, Completion_Notes, and metadata.

### 6.3 Step 2 — Step Extraction

**Objective:** Extract discrete repair steps from unstructured work order text.

**Input:** Long_Text and Completion_Notes from each source WO, plus any matching operations from TaskLists.xlsx.

**Process:**

1. **Task List Cross-Reference:**
   - Check TaskLists.xlsx for active task lists linked to the equipment or functional location
   - If a relevant task list exists, its operations form the baseline step list
   - Task list steps are used as a scaffold — historical WO text adds detail and corrections

2. **Long Text Parsing:**
   For each source WO, the AI parses the Long_Text field to identify discrete work steps. SAP long text is notoriously unstructured — it may contain:
   - Narrative paragraphs ("Arrived on site, met with operator, reviewed JSA...")
   - Numbered lists (sometimes)
   - Abbreviations and shorthand ("R&R mech seal", "ck alignment", "inst new brgs")
   - P6 notation mixed with free text
   - Timestamps and crew names mixed with procedures
   - Multiple jobs documented in one text block

   The AI identifies action-oriented statements and extracts them as individual steps, filtering out narrative, timestamps, and administrative notes.

3. **Completion Notes Mining:**
   Completion notes often contain the most valuable information:
   - What actually happened vs. what was planned
   - Complications encountered ("found shaft scored, had to machine")
   - Parts that were substituted ("used Flowserve seal kit FK-1234 instead of standard")
   - Duration notes ("job took 2 hours longer due to access issues")

   The AI extracts corrective actions and lessons learned from completion notes and integrates them into the step list.

4. **Abbreviation Expansion:**

   | Abbreviation | Expansion |
   |-------------|-----------|
   | R&R | Remove and Replace |
   | R&I | Remove and Install |
   | ck / chk | Check / Inspect |
   | inst | Install |
   | adj | Adjust |
   | brg / brgs | Bearing / Bearings |
   | mech seal | Mechanical Seal |
   | cplg | Coupling |
   | bfv | Butterfly Valve |
   | iso | Isolate |
   | LOTO | Lockout/Tagout |
   | CSE | Confined Space Entry |
   | JSA | Job Safety Analysis |
   | OEM | Original Equipment Manufacturer |
   | BOM | Bill of Materials |
   | NDE | Non-Destructive Examination |

**Output of this step:** A raw list of extracted steps from all source WOs, with source WO attribution.

### 6.4 Step 3 — Step Normalization

**Objective:** Merge, deduplicate, standardize, and logically order the extracted steps into a coherent job plan.

**Process:**

1. **Deduplication:**
   - Identify semantically equivalent steps across source WOs (e.g., "Remove coupling guard" and "Take off cplg guard" are the same step)
   - Merge duplicates, retaining the most detailed version
   - Note which source WOs contributed each step (for traceability)

2. **Language Standardization:**
   All steps are rewritten in a consistent format:
   - **Active verb + noun** structure: "Remove coupling guard", "Inspect shaft seal faces", "Install new bearings"
   - Present tense, imperative mood
   - No abbreviations in the final output
   - Technical terminology preserved (do not simplify "mechanical seal" to "seal")
   - Specific part numbers and measurements retained from source WOs

3. **Safety Step Injection:**
   Standard safety steps are added based on the work scope:

   | Condition | Safety Steps Added |
   |-----------|-------------------|
   | All jobs | Review JSA, Verify LOTO, Confirm equipment isolation |
   | Pump/vessel work | Verify depressurized, Drain and flush, Check for residual hazardous materials |
   | Confined space | CSE permit, Atmospheric monitoring, Rescue plan verification |
   | Hot work involved | Hot work permit, Fire watch assignment |
   | Elevated work | Scaffold inspection, Fall protection verification |
   | Electrical work | Verify zero energy state, Test with voltmeter |
   | Crane/rigging | Lift plan review, Rigging inspection, Load test verification |

4. **Logical Ordering:**
   Steps are arranged in standard maintenance sequence:

   ```
   Phase 1 — Preparation & Safety
     → JSA review
     → LOTO / isolation
     → Verify zero energy state
     → Area preparation

   Phase 2 — Access & Disassembly
     → Remove guards, covers, insulation
     → Disconnect piping, wiring, instrumentation
     → Disassemble equipment

   Phase 3 — Inspection & Assessment
     → Inspect components for wear/damage
     → Measure clearances, runout, alignment
     → Document findings
     → Determine additional work scope if needed

   Phase 4 — Repair / Replacement
     → Replace failed components
     → Install new parts per OEM specifications
     → Apply torque values, adhesives, coatings as required

   Phase 5 — Reassembly & Restoration
     → Reassemble equipment
     → Reconnect piping, wiring, instrumentation
     → Reinstall guards, covers, insulation
     → Verify alignment

   Phase 6 — Testing & Startup
     → Remove LOTO
     → Perform functional test
     → Check for leaks
     → Verify operating parameters
     → Return to operations
   ```

**Output of this step:** An ordered, deduplicated, standardized list of operations ready for formatting.

### 6.5 Step 4 — Duration & Crew Estimation

**Objective:** Estimate total job duration and recommended crew size from historical data.

**Process:**

1. **Duration Analysis:**
   - Collect Actual_Duration from all source WOs
   - Remove outliers (durations > 2x median or < 0.5x median)
   - Calculate **median** duration (not average — averages are skewed by outliers like emergency weekend jobs or jobs delayed by parts availability)
   - If source WOs span different work scopes (some partial, some full overhaul), weight toward WOs matching the current scope

2. **Crew Size Analysis:**
   - Collect Crew_Size from all source WOs
   - Use the **mode** (most common value) — crew size is typically standardized by craft
   - If mixed (e.g., some WOs show 2, some show 3), note both and recommend the larger crew for initial estimate

3. **Contingency Buffer:**
   - Add **15% to duration** for first-time-on-equipment jobs (planner has not previously worked on this equipment)
   - Add **10% to duration** if fewer than 3 source WOs were available (lower confidence in estimate)
   - No buffer added if the planner has generated a plan for this equipment before

4. **Per-Operation Duration:**
   - Distribute total duration across operations proportionally
   - Safety/preparation steps: ~10% of total
   - Disassembly: ~20%
   - Inspection: ~10%
   - Repair/replacement: ~35%
   - Reassembly: ~20%
   - Testing: ~5%
   - These proportions are defaults — adjusted based on historical data when available

**Output of this step:** Total estimated duration (hours), crew size, and per-operation duration allocations.

### 6.6 Step 5 — Output Assembly

**Objective:** Format the normalized steps, duration estimates, and metadata into a structured job plan ready for the planner.

**Job Plan Output Schema:**

```
╔══════════════════════════════════════════════════════════════════╗
║  JOB PLAN — Generated by LIAM                                   ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Equipment:      P-102 — Centrifugal Process Pump                ║
║  Description:    Mechanical Seal Replacement                     ║
║  Failure Mode:   Mechanical Seal Leak                            ║
║  Work Center:    VN-PM (Mechanical)                              ║
║  Est. Duration:  12 hours                                        ║
║  Crew Size:      2 Mechanics                                     ║
║  Source WOs:     WO-400012345, WO-400012890, WO-400013456,       ║
║                  WO-400014001, WO-400014567                      ║
║  Confidence:     High (5 source WOs, same equipment + failure)   ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║  OPERATIONS                                                      ║
╠══════╦═══════╦════════════════════════════════════╦══════╦═══════╣
║ Op#  ║ WC    ║ Description                        ║ Crew ║ Hrs   ║
╠══════╬═══════╬════════════════════════════════════╬══════╬═══════╣
║ 0010 ║ VN-PM ║ Review JSA and obtain work permit  ║  2   ║  0.5  ║
║ 0020 ║ VN-PM ║ Perform LOTO on pump P-102         ║  2   ║  0.5  ║
║ 0030 ║ VN-PM ║ Verify zero energy state            ║  2   ║  0.25 ║
║ 0040 ║ VN-PM ║ Drain pump and flush with water     ║  2   ║  0.5  ║
║ 0050 ║ VN-PM ║ Disconnect suction and discharge    ║  2   ║  1.0  ║
║      ║       ║ piping flanges                      ║      ║       ║
║ 0060 ║ VN-PM ║ Remove coupling guard and           ║  2   ║  0.5  ║
║      ║       ║ disconnect coupling                 ║      ║       ║
║ 0070 ║ VN-PM ║ Remove pump from baseplate          ║  2   ║  1.0  ║
║ 0080 ║ VN-PM ║ Disassemble pump casing — remove    ║  2   ║  1.5  ║
║      ║       ║ impeller, wear rings, and old seal  ║      ║       ║
║ 0090 ║ VN-PM ║ Inspect shaft for scoring, runout,  ║  2   ║  0.5  ║
║      ║       ║ and wear at seal chamber             ║      ║       ║
║ 0100 ║ VN-PM ║ Inspect impeller, wear rings, and   ║  2   ║  0.5  ║
║      ║       ║ volute for erosion or damage         ║      ║       ║
║ 0110 ║ VN-PM ║ Install new mechanical seal per     ║  2   ║  1.5  ║
║      ║       ║ OEM specifications — verify         ║      ║       ║
║      ║       ║ seal face cleanliness and setting    ║      ║       ║
║ 0120 ║ VN-PM ║ Replace wear rings if clearance     ║  2   ║  0.75 ║
║      ║       ║ exceeds OEM tolerance                ║      ║       ║
║ 0130 ║ VN-PM ║ Reassemble pump — torque casing     ║  2   ║  1.0  ║
║      ║       ║ bolts to OEM spec in star pattern    ║      ║       ║
║ 0140 ║ VN-PM ║ Reinstall pump on baseplate and     ║  2   ║  1.0  ║
║      ║       ║ reconnect piping flanges             ║      ║       ║
║ 0150 ║ VN-PM ║ Perform shaft alignment using       ║  2   ║  0.75 ║
║      ║       ║ laser alignment tool                 ║      ║       ║
║ 0160 ║ VN-PM ║ Reinstall coupling and coupling     ║  2   ║  0.25 ║
║      ║       ║ guard                                ║      ║       ║
║ 0170 ║ VN-PM ║ Remove LOTO — coordinate with       ║  2   ║  0.25 ║
║      ║       ║ operations                           ║      ║       ║
║ 0180 ║ VN-PM ║ Start pump and verify: no leaks,    ║  2   ║  0.5  ║
║      ║       ║ normal vibration, correct            ║      ║       ║
║      ║       ║ discharge pressure and flow          ║      ║       ║
║ 0190 ║ VN-PM ║ Monitor pump for 30 minutes —       ║  1   ║  0.5  ║
║      ║       ║ check seal area for leakage          ║      ║       ║
╠══════╩═══════╩════════════════════════════════════╩══════╩═══════╣
║  TOTAL ESTIMATED DURATION: 11.75 hours (12 hours with rounding)  ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  SAFETY REQUIREMENTS                                             ║
║  • LOTO required — pump motor, suction valve, discharge valve    ║
║  • JSA review with crew before work begins                       ║
║  • Drain and flush — verify no hazardous residual materials      ║
║  • Eye protection, chemical-resistant gloves for seal handling   ║
║  • Rigging equipment for pump removal (verify rated capacity)    ║
║                                                                  ║
║  SPECIAL TOOLS                                                   ║
║  • Laser alignment tool (Fixturlaser or equivalent)              ║
║  • Torque wrench — verify calibration current                    ║
║  • Mechanical seal setting tool (per OEM)                        ║
║  • Dial indicator for shaft runout measurement                   ║
║                                                                  ║
║  REQUIRED PERMITS                                                ║
║  • Safe Work Permit                                              ║
║  • LOTO Permit                                                   ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

### 6.7 Planner Interaction — Sample Dialogue

The following shows how a planner interacts with LIAM through Microsoft Teams to generate a job plan:

```
┌─────────────────────────────────────────────────────────────┐
│  Microsoft Teams — LIAM Chat                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PLANNER:                                                    │
│  I need a job plan for P-102. Mechanical seal is leaking.    │
│                                                              │
│  LIAM:                                                       │
│  I found 5 historical work orders for P-102 with seal-       │
│  related failures:                                           │
│                                                              │
│  • WO-400012345 (Mar 2024) — Mech seal replacement, 10 hrs  │
│  • WO-400012890 (Aug 2024) — Seal leak repair, 8 hrs        │
│  • WO-400013456 (Jan 2025) — Overhaul w/ seal, 14 hrs       │
│  • WO-400014001 (Jun 2025) — Mech seal R&R, 11 hrs          │
│  • WO-400014567 (Nov 2025) — Seal failure, 12 hrs           │
│                                                              │
│  I'm generating a job plan based on these. One moment...     │
│                                                              │
│  LIAM:                                                       │
│  ✅ Job plan generated — 19 operations, 12 hours estimated,  │
│  2-person crew.                                              │
│                                                              │
│  [View Full Job Plan]  [Save to Library]  [Modify]           │
│                                                              │
│  PLANNER:                                                    │
│  This looks good. But we're also going to replace the        │
│  bearings while we have it apart. Can you add that?          │
│                                                              │
│  LIAM:                                                       │
│  Updated. I added bearing replacement operations between     │
│  the inspection and reassembly phases:                       │
│                                                              │
│  • Op 0105: Remove old bearings using bearing puller         │
│  • Op 0106: Inspect bearing housing for wear or scoring      │
│  • Op 0107: Install new bearings — press fit to OEM spec     │
│                                                              │
│  Total duration updated to 14 hours.                         │
│                                                              │
│  [View Updated Plan]  [Save to Library]  [Export]            │
│                                                              │
│  PLANNER:                                                    │
│  Save it to the library.                                     │
│                                                              │
│  LIAM:                                                       │
│  Saved as JP-042: "P-102 Centrifugal Pump — Mechanical       │
│  Seal and Bearing Replacement." Status: Draft — needs lead   │
│  planner approval before it's available in the library.      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.8 Edge Cases and Fallbacks

| Scenario | LIAM Behavior |
|----------|---------------|
| No historical WOs found for equipment | Search by equipment class. If still no results, inform planner and suggest checking SAP directly. Offer to generate a generic plan from the failure code. |
| Only 1-2 source WOs available | Generate plan with disclaimer: "Low confidence — based on limited history. Review carefully." Add 10% duration buffer. |
| Conflicting procedures across source WOs | Present both approaches and ask planner to select. E.g., "Source WOs show two approaches for seal installation — in-situ vs. bench replacement. Which do you prefer?" |
| Equipment not in EquipmentMaster.xlsx | Ask planner to confirm equipment ID. If correct, note that equipment master data is missing and generate plan from WO history only. |
| Planner provides WO number instead of equipment | Look up equipment from WorkOrders.xlsx using the WO number. Proceed normally. |
| Very long WO long text (>5000 chars) | AI processes in chunks, extracting steps from each chunk and merging. No truncation of source data. |

---

## 7. Proactive Job Plan Library

### 7.1 Concept

Beyond generating job plans on demand, LIAM maintains a curated library of pre-built job plans for the most common failure modes at the plant. This allows planners to retrieve a proven, approved job plan instantly without waiting for LIAM to search and synthesize from historical data.

### 7.2 Library Structure

Plans are organized by **equipment class + failure mode**:

| Equipment Class | Failure Mode | Plan ID | Title |
|----------------|-------------|---------|-------|
| Centrifugal Pump | Mechanical Seal Failure | JP-001 | Centrifugal Pump Mechanical Seal Replacement |
| Centrifugal Pump | Bearing Failure | JP-002 | Centrifugal Pump Bearing Replacement |
| Centrifugal Pump | Impeller Wear | JP-003 | Centrifugal Pump Impeller Replacement |
| Centrifugal Pump | Full Overhaul | JP-004 | Centrifugal Pump Complete Overhaul |
| Gate Valve | Packing Leak | JP-005 | Gate Valve Packing Replacement |
| Gate Valve | Seat Failure | JP-006 | Gate Valve Seat Lapping / Replacement |
| Motor (Electric) | Bearing Failure | JP-007 | Electric Motor Bearing Replacement |
| Motor (Electric) | Winding Failure | JP-008 | Electric Motor Rewind (Send to Shop) |
| Heat Exchanger | Tube Leak | JP-009 | Shell & Tube Heat Exchanger Tube Repair |
| Heat Exchanger | Fouling | JP-010 | Heat Exchanger Chemical Cleaning |
| Agitator | Seal Failure | JP-011 | Agitator Mechanical Seal Replacement |
| Agitator | Gearbox Failure | JP-012 | Agitator Gearbox Overhaul |
| Control Valve | Actuator Failure | JP-013 | Control Valve Actuator Repair |
| Control Valve | Trim Erosion | JP-014 | Control Valve Trim Replacement |
| Relief Valve | Set Pressure Drift | JP-015 | Pressure Relief Valve Test and Reset |
| Conveyor | Belt Damage | JP-016 | Conveyor Belt Section Replacement |
| Cooling Tower | Fill Replacement | JP-017 | Cooling Tower Fill Media Replacement |
| Compressor | Valve Failure | JP-018 | Reciprocating Compressor Valve Replacement |
| Instrument | Calibration Drift | JP-019 | Transmitter Calibration and Verification |
| Piping | Corrosion Leak | JP-020 | Pipe Section Cut and Replace |

### 7.3 Library Lifecycle

1. **Generation:** LIAM generates a draft plan using the Job Plan Generation Engine (Section 6), pulling from the best available historical WOs for that equipment class + failure mode
2. **Review:** A lead planner reviews the draft, corrects any steps, adjusts durations, and adds site-specific knowledge
3. **Approval:** Lead planner marks the plan as "Approved" — it becomes available for retrieval
4. **Usage:** Any planner can retrieve an approved plan by asking: "Give me the job plan for a centrifugal pump mechanical seal replacement"
5. **Update:** When new WOs provide better procedures or corrections, LIAM can propose updates to existing library plans. Updates require lead planner re-approval
6. **Versioning:** Superseded plans are retained with status "Superseded" for audit trail. Only the current approved version is returned to planners

### 7.4 Retrieval Interface

```
PLANNER: I need the standard plan for a pump seal job.

LIAM: I found JP-001: "Centrifugal Pump Mechanical Seal Replacement"
      Version 2, approved by J. Rodriguez on 2026-02-15.
      19 operations, 12 hours, 2-person crew.

      [View Plan]  [Use for WO]  [Request Update]

PLANNER: Use it for WO-400015234.

LIAM: Got it. I've attached JP-001 to WO-400015234 context.
      The plan references equipment class "Centrifugal Pump" —
      your WO is for P-102 (Flowserve ANSI 3x1.5-6).
      I've customized the plan with P-102-specific notes from
      recent WOs. Review and adjust as needed.

      [View Customized Plan]
```

---

## 8. Additional Skills

### 8.1 Work Order Intelligence Q&A

LIAM answers natural language questions about maintenance history by querying the SharePoint data hub. This is the simplest skill — direct data retrieval with AI-generated natural language responses.

**Example Queries and Responses:**

```
Q: "When did P-102 last fail and what did we do?"
A: P-102 last had a work order on November 15, 2025 (WO-400014567).
   Failure: Mechanical seal leak (failure code MC-SL-01).
   Work performed: Removed and replaced mechanical seal, inspected
   shaft and wear rings, laser aligned after reassembly.
   Duration: 12 hours, 2-person crew.

Q: "How many times has the mechanical seal failed on pumps in Unit 100
    this year?"
A: In 2026 (year to date), there have been 4 mechanical seal failures
   on pumps in Unit 100:
   • P-102 — Jan 8 (WO-400015001)
   • P-107 — Feb 22 (WO-400015089)
   • P-115 — Feb 28 (WO-400015112)
   • P-102 — Mar 1 (WO-400015134)
   Note: P-102 has had 2 seal failures in 2 months. Consider
   reviewing seal selection or operating conditions.

Q: "What is the average repair time for a Flowserve ANSI pump overhaul?"
A: Based on 8 completed overhauls of Flowserve ANSI pumps in the last
   3 years, the median repair time is 16 hours (range: 12–22 hours).
   Average crew size: 2 mechanics.
   Longest job: P-203 overhaul in May 2024 (22 hours — discovered
   cracked casing, required field weld repair).

Q: "Show me all open notifications for the Distillation area."
A: There are 7 open notifications for Distillation (Unit 200):
   1. NF-10045678 — V-201 level transmitter erratic (Priority 2)
   2. NF-10045690 — P-205 high vibration at outboard bearing (Priority 2)
   3. NF-10045701 — HX-202 outlet temp above design (Priority 3)
   [... remaining notifications ...]
```

### 8.2 Daily Maintenance Summary

An automated Power Automate flow generates a plain-English summary of maintenance activity and delivers it to Teams every morning.

**Trigger:** Scheduled — 6:00 AM CST, Monday through Friday

**Process:**
1. Read WorkOrders.xlsx for WOs with activity in the previous 24 hours
2. Read Notifications.xlsx for newly created notifications
3. AI generates a structured summary organized by work center

**Sample Output (posted to Teams channel):**

```
📋 DAILY MAINTENANCE SUMMARY — March 6, 2026

VN-PM (Mechanical):
• 3 WOs completed yesterday:
  - WO-400015134: P-102 mech seal replacement (12 hrs) ✅
  - WO-400015140: V-305 flange bolt-up (4 hrs) ✅
  - WO-400015098: HX-201 tube plugging (6 hrs) ✅
• 2 WOs in progress:
  - WO-400015145: P-310 full overhaul (Day 2 of 3)
  - WO-400015150: K-101 compressor valve replacement (started today)
• 1 new notification:
  - NF-10045720: P-108 abnormal noise at coupling end (Priority 2)

VN-EC (Electrical/Instrumentation):
• 1 WO completed:
  - WO-400015138: FT-201A flow transmitter calibration (2 hrs) ✅
• 2 WOs in progress:
  - WO-400015142: MOV-301 actuator repair (waiting for parts)
  - WO-400015149: Panel 4A breaker replacement (scheduled today)
• 0 new notifications

BACKLOG: 23 open WOs (VN-PM: 15, VN-EC: 8)
OVERDUE: 2 WOs past scheduled completion date
```

### 8.3 PM Recommendation Engine

A simple frequency-based analysis that compares actual failure rates against preventive maintenance intervals and flags mismatches.

**Trigger:** Weekly scheduled Power Automate flow (Sundays at 8:00 PM)

**Logic:**
1. For each equipment item with a PM plan in PreventiveMaintenance.xlsx:
   - Count corrective WOs (PM01 type) in the last 24 months
   - Calculate mean time between failures (MTBF)
   - Compare MTBF to PM interval
2. Flag if: MTBF < PM interval (equipment is failing more often than it's being maintained)

**Sample Alert (posted to Teams):**

```
⚠️ PM INTERVAL REVIEW RECOMMENDED

P-102 — Centrifugal Process Pump
• Current PM interval: 24 months
• Actual failures in last 24 months: 4 (MTBF = 6 months)
• Failure mode: Mechanical seal leak (3 of 4 occurrences)
• Recommendation: Consider reducing PM interval to 12 months
  or investigating root cause of recurring seal failures.

P-205 — Booster Pump
• Current PM interval: 12 months
• Actual failures in last 24 months: 3 (MTBF = 8 months)
• Failure mode: Bearing failure (2 of 3), Coupling failure (1)
• Recommendation: Consider reducing PM interval to 6 months
  or adding vibration monitoring to PM task list.
```

**Important constraint:** This is advisory only. LIAM does not modify PM plans, SAP master data, or any schedules. Recommendations are delivered to the planning team for their judgment.

---

## 9. Copilot Studio Architecture

### 9.1 Agent Configuration

| Setting | Value |
|---------|-------|
| Agent Name | LIAM — Maintenance Planning Assistant |
| Description | AI-powered maintenance planning assistant for Kuraray LaPorte. Generates job plans, answers maintenance history questions, and delivers operational reports. |
| Deployment | Microsoft Teams |
| Language | English |
| Authentication | Microsoft 365 SSO (integrated) |

### 9.2 System Prompt (Draft)

```
You are LIAM (LaPorte Intelligent Asset Management), an AI maintenance
planning assistant for the Kuraray LaPorte chemical plant.

Your primary users are maintenance planners who need help creating
structured job plans for corrective maintenance work orders.

CAPABILITIES:
- Generate job plans from historical work order data
- Answer questions about equipment maintenance history
- Retrieve pre-built job plans from the Job Plan Library
- Provide PM interval recommendations based on failure frequency

BEHAVIOR RULES:
- Always identify the equipment ID and failure mode before generating
  a job plan. Ask for clarification if the planner's request is ambiguous.
- When generating job plans, always include safety steps (LOTO, JSA,
  permits) — never omit safety procedures.
- When reporting durations, use median values from historical data,
  not averages. Explain this if asked.
- If you cannot find historical data for an equipment item, say so
  clearly. Do not fabricate work order numbers, dates, or procedures.
- Format job plans with operation numbers in increments of 10
  (0010, 0020, 0030...) to allow planners to insert steps.
- Always show the source work orders used to generate a plan so
  planners can trace the origin of each procedure.
- When asked about failure patterns, include both the count and a
  brief note if the pattern suggests a recurring problem.
- You do NOT have access to SAP directly. All data comes from
  SharePoint Excel files exported from SAP.
- You do NOT schedule work, predict failures, or modify PM plans.
  For scheduling, refer planners to P6 Intelligence.
  For failure prediction, refer to Pinnacle Reliability Newton.

DATA SOURCES:
- WorkOrders.xlsx — Historical completed work orders (3 years)
- TaskLists.xlsx — SAP standard task list operations
- Notifications.xlsx — Maintenance notifications and problem reports
- PreventiveMaintenance.xlsx — PM schedules and intervals
- EquipmentMaster.xlsx — Equipment hierarchy and master data
- FailureCodes.xlsx — Failure code catalog
- JobPlanLibrary.xlsx — Pre-built and LIAM-generated job plans

TONE:
- Professional, concise, technically accurate
- Use maintenance terminology that planners understand
- Do not over-explain — planners are subject matter experts
- When uncertain, say "Based on the available data..." rather than
  guessing
```

### 9.3 Topics / Skills

| Topic | Trigger Phrases | Action |
|-------|----------------|--------|
| Generate Job Plan | "Generate a job plan for...", "I need a plan for...", "Create a job plan...", "What's the procedure for..." | Invoke Job Plan Generation Engine (Section 6) |
| Ask Maintenance History | "When did ... last fail?", "How many times...", "What did we do on...", "Show me work orders for..." | Query WorkOrders.xlsx and Notifications.xlsx, generate natural language response |
| Get Library Plan | "Give me the standard plan for...", "Pull up the library plan for...", "Do we have a plan for..." | Search JobPlanLibrary.xlsx by equipment class + failure mode |
| Daily Summary | "What happened yesterday?", "Give me today's summary", "Morning report" | Trigger Daily Summary flow or retrieve most recent summary |
| PM Recommendation | "Should we adjust PM on...", "How often is ... failing?", "PM interval check for..." | Run PM frequency analysis for specified equipment |
| Help | "What can you do?", "Help", "How do I use this?" | Display capabilities list and example prompts |

### 9.4 Actions (Power Automate Integrations)

| Action | Type | Description |
|--------|------|-------------|
| SearchWorkOrders | Copilot Action | Query WorkOrders.xlsx with filters (equipment, date range, status, failure code) |
| SearchEquipment | Copilot Action | Look up equipment details from EquipmentMaster.xlsx |
| GetTaskList | Copilot Action | Retrieve task list operations for an equipment item |
| SearchNotifications | Copilot Action | Query open or historical notifications |
| GetLibraryPlan | Copilot Action | Retrieve a job plan from JobPlanLibrary.xlsx |
| SaveLibraryPlan | Copilot Action | Save a new job plan to JobPlanLibrary.xlsx (draft status) |
| GetPMSchedule | Copilot Action | Retrieve PM plan details for equipment |

### 9.5 Knowledge Sources

Copilot Studio indexes the following for grounded responses:
- All 7 SharePoint Excel files (structured data)
- LIAM-ARCHITECTURE.md (for self-description and capability questions)
- Plant-specific reference documents (OEM manuals, if uploaded to SharePoint)

### 9.6 Escalation Path

When LIAM cannot answer a question or generate a plan:

```
LIAM: I wasn't able to find historical work orders for equipment
      XYZ-999. This could mean:
      1. The equipment ID might be different in SAP — try checking
         IE03 in SAP for the correct number
      2. This equipment may not have had corrective maintenance in
         the last 3 years (our data window)
      3. The data export may not include this equipment's work center

      If you find the correct equipment ID, I can try again.
      For urgent planning needs, check SAP transaction IW47 directly
      with the functional location as a search filter.
```

---

## 10. Power Automate Flows

### 10.1 Daily Morning Summary

| Setting | Value |
|---------|-------|
| Flow Name | LIAM-DailyMorningSummary |
| Trigger | Schedule — Recurrence, 6:00 AM CST, Monday–Friday |
| Duration | ~2 minutes |

**Steps:**
1. **Get yesterday's date range** — Calculate previous business day (skip weekends for summary, but include weekend WOs on Monday)
2. **Read WorkOrders.xlsx** — Filter rows where Actual_Start or Actual_End falls within date range
3. **Read Notifications.xlsx** — Filter rows where Created_Date = yesterday
4. **Group by Work Center** — Separate VN-PM and VN-EC activity
5. **Compose summary prompt** — Build AI prompt with WO data, notification data, and backlog count
6. **AI Builder — Create Text** — Generate plain-English summary from the data
7. **Post to Teams** — Send Adaptive Card to the Maintenance Planning Teams channel
8. **Log execution** — Record success/failure timestamp

### 10.2 Job Plan Generation

| Setting | Value |
|---------|-------|
| Flow Name | LIAM-GenerateJobPlan |
| Trigger | Copilot Studio Action (on-demand) |
| Duration | ~30–60 seconds |

**Steps:**
1. **Receive input** — Equipment ID and failure description from Copilot
2. **Search WorkOrders.xlsx** — Execute cascading search strategy (Section 6.2)
3. **Search TaskLists.xlsx** — Find matching task list operations
4. **Lookup EquipmentMaster.xlsx** — Get equipment class and metadata
5. **Compose generation prompt** — Build AI prompt with source WO data, task list data, equipment metadata
6. **AI Builder — Create Text** — Generate normalized job plan
7. **Format output** — Structure into operations table with header and footer
8. **Return to Copilot** — Send formatted job plan back to the conversation

### 10.3 New WO Ingestion

| Setting | Value |
|---------|-------|
| Flow Name | LIAM-NewWOIngestion |
| Trigger | When a file is modified — SharePoint (WorkOrders.xlsx) |
| Duration | ~1 minute |

**Steps:**
1. **Detect file change** — SharePoint triggers when WorkOrders.xlsx is updated
2. **Read new rows** — Compare row count to last known count, identify new entries
3. **Validate data** — Check required fields are populated (WO_Number, Equipment_ID, Status)
4. **Log ingestion** — Record count of new WOs ingested and any validation errors
5. **Optional notification** — If a new WO matches a high-priority equipment item, send a Teams alert to the assigned planner

### 10.4 Weekly Backlog Report

| Setting | Value |
|---------|-------|
| Flow Name | LIAM-WeeklyBacklogReport |
| Trigger | Schedule — Recurrence, Monday 7:00 AM CST |
| Duration | ~3 minutes |

**Steps:**
1. **Read WorkOrders.xlsx** — Filter to Status = REL (released, not yet complete)
2. **Calculate aging** — Days since Planned_Start for each open WO
3. **Group by Work Center and Priority** — Create backlog breakdown
4. **Identify overdue** — Flag WOs past Planned_End date
5. **Compose report prompt** — Build AI prompt with backlog data
6. **AI Builder — Create Text** — Generate weekly backlog summary
7. **Post to Teams** — Send to Maintenance Planning channel
8. **Optional: Email** — Send to maintenance superintendent

### 10.5 PM Flag Alert

| Setting | Value |
|---------|-------|
| Flow Name | LIAM-PMFlagAlert |
| Trigger | Schedule — Recurrence, Sunday 8:00 PM CST |
| Duration | ~2 minutes |

**Steps:**
1. **Read PreventiveMaintenance.xlsx** — Get all active PM plans
2. **Read WorkOrders.xlsx** — Get corrective WOs (PM01) for last 24 months
3. **Calculate MTBF** — For each equipment with a PM plan, count corrective failures and calculate mean time between failures
4. **Compare to PM interval** — Flag equipment where MTBF < PM cycle duration
5. **Compose alert** — Build recommendation with failure history and suggested interval
6. **Post to Teams** — Send flagged items to Maintenance Planning channel (only if flags exist — no empty alerts)

---

## 11. Security & Access

### 11.1 Authentication

LIAM uses **Microsoft 365 integrated authentication**. There is no separate login, no API keys stored in the application, and no custom identity management. Users authenticate through their existing Microsoft 365 account, which is already managed by the organization's IT department with MFA, conditional access policies, and directory services.

### 11.2 Authorization Model

| Role | Access Level | Users |
|------|-------------|-------|
| Planner | Read all data, generate job plans, retrieve library plans, ask questions | All maintenance planners (VN-PM, VN-EC) |
| Lead Planner | All Planner permissions + approve/edit library plans | 2 lead planners |
| Supervisor | Read-only access to reports and summaries | Maintenance superintendent, area supervisors |
| Admin | Full access including Power Automate flow management | IT admin, LIAM system owner |

### 11.3 Data Classification

| Data Type | Classification | Notes |
|-----------|---------------|-------|
| Work order text | Internal/Operational | No PII — work descriptions, equipment data |
| Equipment master | Internal/Operational | Asset data, no financial information |
| Failure codes | Internal/Operational | Standardized catalog |
| Job plans | Internal/Operational | Procedures — no proprietary process chemistry |
| Planner names | Internal/Low Sensitivity | Only in Created_By fields, no SSNs or personal data |

### 11.4 Data Residency

All data resides within the organization's Microsoft 365 tenant:
- SharePoint Online — data files
- Copilot Studio — agent configuration and conversation logs
- Power Automate — flow definitions and run history
- Teams — message delivery and conversation history

No data is sent to external services, third-party APIs, or non-Microsoft infrastructure. The AI processing occurs within Microsoft's Copilot Studio and AI Builder services, governed by the organization's Microsoft 365 data processing agreement.

### 11.5 Teams Channel Permissions

Report delivery is controlled by Teams channel membership:
- **Maintenance Planning** channel: daily summaries, weekly backlogs, PM alerts
- Only members of the channel see the reports
- Channel membership managed by the maintenance department, not LIAM

---

## 12. Implementation Roadmap

### Phase 1 — Data Foundation (2 weeks)

**Objective:** Establish a clean, structured SharePoint data hub with 3 years of historical SAP PM data.

**Week 1:**
- Export work orders from SAP IW47: last 3 years, all equipment, TECO and CLSD status, work centers VN-PM and VN-EC
- Export task lists from SAP IA05/IA06: all active task lists for plant equipment
- Export notifications from SAP IW28: last 3 years, all notification types
- Export PM schedules from SAP IP19: all active maintenance plans
- Export equipment master from SAP IE05: all installed equipment
- Export failure code catalog from SAP OIS2: all active catalogs

**Week 2:**
- Clean and standardize field names across all exports (map SAP field names to LIAM schema)
- Remove test orders, training data, and cancelled WOs
- Standardize date formats (SAP date → ISO 8601)
- Normalize equipment IDs (strip leading zeros, standardize format)
- Upload all 7 files to SharePoint document library
- Validate data completeness: spot-check 20 known work orders across different equipment types
- Create JobPlanLibrary.xlsx template with empty table and correct schema
- Document data dictionary with field definitions, valid values, and data lineage

**Exit Criteria:** All 7 Excel files in SharePoint, validated against 20 known WOs, with documented data dictionary.

**Risk:** SAP export may not include all desired fields (especially Long_Text, which requires a separate download in some SAP configurations). Mitigation: test export process with a small sample first.

### Phase 2 — Copilot Agent Setup (1 week)

**Objective:** Working Copilot Studio agent with basic work order history Q&A.

**Tasks:**
- Create Copilot Studio agent with system prompt (Section 9.2)
- Connect SharePoint as a data source — configure Excel table references
- Build "AskMaintenance" topic with trigger phrases
- Build SearchWorkOrders and SearchEquipment actions in Power Automate
- Test with 5 representative queries:
  1. "When did P-102 last have maintenance?"
  2. "How many corrective WOs were there for Unit 100 this month?"
  3. "What failure codes have been used on heat exchangers?"
  4. "Show me open notifications for VN-PM"
  5. "What is the equipment class for P-205?"
- Deploy to Teams for 5 pilot planners
- Collect feedback on response quality, accuracy, and speed

**Exit Criteria:** Planners can ask equipment history questions through Teams and receive correct, natural language answers within 10 seconds.

### Phase 3 — Job Plan Generation Engine (3 weeks)

**Objective:** Core job plan generation skill — planner provides equipment + failure, LIAM returns a structured job plan.

**Week 1:**
- Build WO search & retrieval flow (cascading search strategy, Section 6.2)
- Build equipment class lookup for broadened search
- Test search coverage: try 10 equipment items and verify relevant WOs are found

**Week 2:**
- Build step extraction logic (Long_Text and Completion_Notes parsing)
- Build abbreviation expansion table
- Build step normalization (dedup, standardize language, logical ordering)
- Build safety step injection rules

**Week 3:**
- Build duration & crew estimation (median calculation, contingency buffer)
- Build output assembly (formatted job plan with header/footer)
- Generate 10 test job plans on known equipment with known good WO history
- Review all 10 with lead planner — iterate on quality, accuracy, and format
- Integrate as Copilot Studio skill (GenerateJobPlan topic)

**Exit Criteria:** Job plan generation produces job plans reviewed and approved as "usable" by lead planner for at least 8 of 10 test cases.

### Phase 4 — Automation & Reporting (1 week)

**Objective:** Automated daily summaries, weekly backlogs, and proactive PM alerts.

**Tasks:**
- Build and test Daily Morning Summary flow (Section 10.1)
- Build and test Weekly Backlog Report flow (Section 10.4)
- Build and test PM Flag Alert flow (Section 10.5)
- Build and test New WO Ingestion flow (Section 10.3)
- Configure Teams channel delivery for each flow
- Run all flows for 5 consecutive business days, verify output quality
- Collect feedback from planning team on report usefulness and format

**Exit Criteria:** All 4 Power Automate flows running reliably for 1 full business week with no errors and positive planner feedback.

### Phase 5 — Job Plan Library Buildout (ongoing)

**Objective:** Curated library of pre-built job plans for the most common failure modes at the plant.

**Initial Sprint (2 weeks):**
- Analyze WorkOrders.xlsx to identify top 20 failure modes by frequency
- Generate draft job plans for each using the Job Plan Generation Engine
- Lead planner reviews and approves each plan (target: 2–3 plans per day)
- Load approved plans into JobPlanLibrary.xlsx
- Build GetLibraryPlan topic in Copilot Studio
- Test retrieval with 5 planners

**Ongoing:**
- Continue building library beyond initial 20 (target: 50 plans in 90 days)
- Update existing plans when new WOs provide better procedures
- Track plan usage (Use_Count field) to prioritize updates for high-use plans
- Quarterly review of library with planning team

**Exit Criteria (initial):** 20 approved plans in library, retrievable through Teams, used by at least 3 planners.

---

## 13. Integration with BAITEKS Ecosystem

### 13.1 LIAM + P6 Intelligence

LIAM and P6 Intelligence are complementary tools covering different parts of the maintenance planning lifecycle:

| Capability | LIAM | P6 Intelligence |
|-----------|------|-----------------|
| **What to do** | ✅ Generates job plans with procedures, steps, durations | — |
| **When to do it** | — | ✅ Optimizes schedule, resolves resource conflicts |
| **History analysis** | ✅ Queries past WOs, finds patterns | ✅ Queries P6 schedule history |
| **Data source** | SAP PM (Excel export) | Primavera P6 (SQL Server) |
| **Interface** | Microsoft Teams (Copilot Studio) | Web app (FastAPI) |

**Planned Integration Flow:**
1. Planner asks LIAM to generate a job plan for a corrective work order
2. LIAM generates the plan with estimated duration and crew size
3. Planner approves the plan and enters it into SAP
4. SAP work order feeds into P6 scheduling
5. P6 Intelligence optimizes the schedule: resource leveling, critical path, conflict detection
6. Planner reviews the optimized schedule in P6 Intelligence

This is currently a **manual handoff** — the planner carries information between systems. Future integration could automate the transfer of duration and resource estimates from LIAM to P6.

### 13.2 Future: Marcus AI Voice Interface

For field crews and supervisors on the plant floor, LIAM's capabilities could be accessed through the Marcus voice interface:

- Supervisor walks up to equipment, asks: "Marcus, when did this pump last get overhauled?"
- Marcus routes the query to LIAM, which searches WO history and returns a spoken answer
- Use case: quick history checks without returning to a desk to open Teams

This integration requires:
- LIAM API endpoint (or Copilot Studio API access)
- Marcus voice → text → LIAM query → text → voice pipeline
- Network connectivity at the plant (may require mobile hotspot or plant WiFi)

**Status:** Concept — not in current build phases. Depends on Kuraray plant network and security approval.

### 13.3 Future: BAITEKS SAP Intelligence

If Kuraray moves to SAP API access (S/4HANA migration or gateway deployment), a future BAITEKS SAP Intelligence layer could:

- Replace batch Excel exports with real-time SAP queries
- Enable LIAM to read WO data in real time (no stale data)
- Potentially enable LIAM to write job plans directly into SAP task lists (with planner approval)
- Integrate with SAP notification workflow for automatic plan triggering

**Status:** Not planned. Depends on customer SAP roadmap.

---

## 14. Success Metrics

### 14.1 Primary Metrics

| Metric | Current State | Target | Measurement |
|--------|-------------|--------|-------------|
| Job plan creation time | 2–4 hours | <15 minutes (including review) | Planner self-report, before/after comparison |
| Job plan quality score | Inconsistent (no baseline) | >80% of plans used without modification | Track modifications made after LIAM generation |
| Planner adoption | 0/5 planners | 5/5 planners using within 30 days | Usage logs in Copilot Studio |
| Library size | 0 plans | 50 plans in 90 days | JobPlanLibrary.xlsx row count |
| Job plan reuse rate | Unknown | >60% of corrective WOs use a LIAM plan | Cross-reference WO plans with library |

### 14.2 Secondary Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Daily summary engagement | Opened by 3+ users daily | Teams message read receipts |
| Q&A query volume | >10 queries per week | Copilot Studio analytics |
| PM flag accuracy | >80% of flags actionable | Lead planner feedback on recommendations |
| Average generation time | <60 seconds | Power Automate flow run duration |
| Data freshness | WO data <7 days old | Last modified timestamp on WorkOrders.xlsx |

### 14.3 Business Impact (Estimated)

| Impact Area | Estimate | Basis |
|------------|----------|-------|
| Labor savings (planning time) | 500–1,000 hours/year | 250 job plans × 2–3 hours saved each |
| Labor cost savings | $50,000–120,000/year | Planning time savings × burdened rate |
| Execution efficiency | 10–15% reduction in job duration | Better-planned jobs execute faster |
| Knowledge retention | Qualitative — critical | Institutional knowledge captured in library |
| Consistency improvement | Qualitative — significant | Standardized procedures across all planners |

### 14.4 Measurement Plan

- **Week 1–2:** Baseline measurement — track current job plan creation time for 10 plans
- **Week 4 (post Phase 2):** Measure Q&A response accuracy and planner satisfaction
- **Week 7 (post Phase 3):** Measure job plan generation time, quality score, adoption rate
- **Week 8 (post Phase 4):** Measure daily summary engagement, PM flag accuracy
- **Week 12 (post Phase 5 initial):** Full metric review — library size, reuse rate, business impact estimate
- **Quarterly:** Ongoing metric review and target adjustment

---

## Appendix A — Glossary

| Term | Definition |
|------|-----------|
| BAPI | Business Application Programming Interface (SAP) |
| BOM | Bill of Materials |
| CLSD | Closed (SAP WO status — fully settled and closed) |
| CRTD | Created (SAP WO status — draft) |
| CSE | Confined Space Entry |
| EAM | Enterprise Asset Management |
| JSA | Job Safety Analysis |
| LOTO | Lockout / Tagout |
| MTBF | Mean Time Between Failures |
| NDE | Non-Destructive Examination |
| OEM | Original Equipment Manufacturer |
| PM | Preventive Maintenance |
| PM01 | SAP WO type — Corrective Maintenance |
| PM02 | SAP WO type — Preventive Maintenance |
| PM03 | SAP WO type — Emergency Maintenance |
| R&R | Remove and Replace |
| R&I | Remove and Install |
| REL | Released (SAP WO status — approved for execution) |
| SAP PM | SAP Plant Maintenance module |
| TECO | Technically Complete (SAP WO status — work finished) |
| VN-EC | Work center — Electrical / Instrumentation crew |
| VN-PM | Work center — Mechanical crew |
| WO | Work Order |

---

## Appendix B — Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | March 2026 | Mike Birklett | Initial architecture specification |

---

*LIAM — LaPorte Intelligent Asset Management*
*Built by BAITEKS — Business & AI Technology Solutions*
*https://baiteks.com*
