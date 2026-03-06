# LIAM — Functional Requirements & Traceability

> **Status:** Stub — to be expanded during Phase 1

---

## FR-001: Job Plan Generation

| Field | Value |
|-------|-------|
| Priority | P1 — Core |
| Description | Given an equipment ID and failure description, LIAM shall search historical work orders, extract repair steps, normalize them, estimate duration/crew, and return a structured job plan |
| Input | Equipment ID, failure description (natural language) |
| Output | Structured job plan (operations, work center, duration, crew size, safety requirements) |
| Data Sources | WorkOrders.xlsx, TaskLists.xlsx, Notifications.xlsx, EquipmentMaster.xlsx |
| Acceptance Criteria | Job plan generated in <60 seconds, contains minimum 5 operations, includes safety header |

---

## FR-002: Work Order Intelligence Q&A

| Field | Value |
|-------|-------|
| Priority | P1 — Core |
| Description | LIAM shall answer natural language questions about maintenance history |
| Input | Natural language question via Teams |
| Output | Plain-English answer with supporting data |
| Data Sources | WorkOrders.xlsx, Notifications.xlsx, EquipmentMaster.xlsx |
| Acceptance Criteria | Correct answers to equipment history, failure frequency, repair duration queries |

---

## FR-003: Proactive Job Plan Library

| Field | Value |
|-------|-------|
| Priority | P2 — Enhancement |
| Description | LIAM shall maintain a library of pre-built job plans for common failure modes, retrievable on demand |
| Input | Equipment class + failure mode query |
| Output | Pre-built job plan from JobPlanLibrary.xlsx |
| Data Sources | JobPlanLibrary.xlsx |
| Acceptance Criteria | Library contains top 20 failure modes within 90 days of launch |

---

## FR-004: Daily Maintenance Summary

| Field | Value |
|-------|-------|
| Priority | P2 — Enhancement |
| Description | LIAM shall generate and deliver a daily plain-English maintenance summary to Teams |
| Input | Scheduled trigger (6:00 AM daily) |
| Output | Summary posted to Teams channel |
| Data Sources | WorkOrders.xlsx, Notifications.xlsx |
| Acceptance Criteria | Delivered by 6:15 AM, covers previous day activity, readable by non-technical leadership |

---

## FR-005: PM Interval Recommendation

| Field | Value |
|-------|-------|
| Priority | P3 — Advisory |
| Description | LIAM shall flag equipment where failure frequency exceeds PM interval and recommend adjustment |
| Input | Scheduled analysis (weekly) |
| Output | Advisory alert to Teams |
| Data Sources | WorkOrders.xlsx, PreventiveMaintenance.xlsx |
| Acceptance Criteria | Correctly identifies equipment with failure rate > PM frequency |

---

## Traceability Matrix

| Requirement | Phase | Copilot Skill | Power Automate Flow | Data Files |
|------------|-------|--------------|--------------------|-----------|
| FR-001 | Phase 3 | GenerateJobPlan | Job Plan Generation | WO, TL, NF, EM |
| FR-002 | Phase 2 | AskMaintenance | — | WO, NF, EM |
| FR-003 | Phase 5 | GetLibraryPlan | — | JPL |
| FR-004 | Phase 4 | — | Daily Morning Summary | WO, NF |
| FR-005 | Phase 4 | — | PM Flag Alert | WO, PM |

---

*To be expanded with detailed test cases, edge cases, and validation criteria during Phase 1.*
