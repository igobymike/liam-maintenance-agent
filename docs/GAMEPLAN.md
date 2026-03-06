# LIAM — Build Gameplan

> **Status:** Stub — to be detailed as each phase begins

---

## Phase 1 — Data Foundation (2 weeks)

**Goal:** Clean, structured SharePoint data hub with 3 years of historical SAP PM data.

- [ ] Export work orders from SAP IW47 (last 3 years, all equipment, TECO status)
- [ ] Export task lists from SAP IA05/IA06
- [ ] Export notifications from SAP IW28
- [ ] Export PM schedules from SAP IP19
- [ ] Export equipment master from SAP IE05
- [ ] Export failure code catalog from SAP OIS2
- [ ] Standardize column names across all files
- [ ] Clean data: remove test orders, standardize date formats, normalize equipment IDs
- [ ] Upload all files to SharePoint document library
- [ ] Validate data completeness: spot-check 20 known work orders
- [ ] Create JobPlanLibrary.xlsx template with schema
- [ ] Document data dictionary for each file

**Exit Criteria:** All 7 Excel files in SharePoint, validated, with documented schema.

---

## Phase 2 — Copilot Agent Setup (1 week)

**Goal:** Working Copilot Studio agent with basic WO history Q&A.

- [ ] Create Copilot Studio agent (LIAM)
- [ ] Connect SharePoint data source
- [ ] Build "Ask Maintenance" skill — basic Q&A on WO history
- [ ] Test with 5 representative queries
- [ ] Deploy to Teams for 5 pilot planners
- [ ] Collect feedback on response quality and speed

**Exit Criteria:** Planners can ask equipment history questions and get correct answers.

---

## Phase 3 — Job Plan Generation Engine (3 weeks)

**Goal:** Core job plan generation skill — input equipment + failure, output structured job plan.

- [ ] Build WO search & retrieval (same equipment, then same class)
- [ ] Build step extraction from WO long text and completion notes
- [ ] Build step normalization (dedup, standardize language, logical ordering)
- [ ] Build safety step injection (LOTO, CSE, permits)
- [ ] Build duration & crew estimation (median, contingency buffer)
- [ ] Build output assembly (formatted job plan with header/footer)
- [ ] Generate 10 test job plans on known equipment
- [ ] Review with lead planner — iterate on quality
- [ ] Integrate as Copilot Studio skill

**Exit Criteria:** Job plan generation produces usable output reviewed by lead planner.

---

## Phase 4 — Automation & Reporting (1 week)

**Goal:** Automated daily summaries and proactive alerts.

- [ ] Build Daily Morning Summary flow (6 AM trigger)
- [ ] Build Weekly Backlog Report flow (Monday 7 AM)
- [ ] Build PM Flag Alert flow (weekly)
- [ ] Build New WO Ingestion flow (file arrival trigger)
- [ ] Test all flows for 5 consecutive days
- [ ] Configure Teams channel delivery

**Exit Criteria:** All 4 Power Automate flows running reliably for 1 week.

---

## Phase 5 — Job Plan Library Buildout (ongoing)

**Goal:** Curated library of pre-built job plans for top failure modes.

- [ ] Identify top 20 failure modes by frequency (from WO data)
- [ ] Generate draft job plans for each using the engine
- [ ] Lead planner review and approval for each plan
- [ ] Load approved plans into JobPlanLibrary.xlsx
- [ ] Build "Get Library Plan" skill in Copilot Studio
- [ ] Target: 50 plans in library within 90 days

**Exit Criteria:** 20 approved plans in library, retrievable via Teams.

---

*Each phase will be detailed with specific dates, owners, and dependencies as work begins.*
