# Microsoft Sentinel PoC — Week 1 Progress Report

*Application Security Engineering — 10-Day Sentinel Case Study Project*

| | |
|---|---|
| **Project** | Microsoft Sentinel PoC (CloudScale) |
| **Reporting Period** | Day 1 – Day 5 (Week 1) |
| **Environment** | Azure Free Account, East US region |
| **Workspace** | LAW-Sentinel-CloudScale |
| **Resource Group** | RG-Sentinel-PoC |

---

## 1. Executive Summary

In Week 1 of this project, a functioning Microsoft Sentinel proof-of-concept environment was stood up end-to-end on Azure: from initial resource provisioning through to a working custom detection rule and incident management configuration. The environment now ingests subscription activity, evaluates it against both Microsoft's built-in detection logic and a purpose-built behavioral rule, and groups related alerts into manageable incidents rather than a flat, noisy alert stream.

Specifically, the following was accomplished:

- Provisioned the foundational Azure infrastructure: a dedicated resource group and a Log Analytics workspace configured for a 90-day retention window on Pay-as-you-go pricing.
- Enabled Microsoft Sentinel on that workspace and connected the Azure Activity Logs data connector, establishing a live telemetry feed of subscription-level control-plane events.
- Developed working proficiency in Kusto Query Language (KQL), progressing from basic filtering (`take`, `where`, `project`, `summarize`) to multi-stage analytical queries involving JSON parsing, time-based conditions, and entity extraction.
- Researched the MITRE ATT&CK framework and grounded the project's detection work in it, focusing on Initial Access, Privilege Escalation, and Defense Evasion as the tactics most relevant to an IAM-heavy cloud environment.
- Authored, saved, and tested a custom Analytics Rule ("Privileged Role Assignment Outside Business Hours") that detects Owner/Contributor role assignments occurring outside normal business hours — a behavioral indicator of potential privilege escalation.
- Enabled five built-in Analytics Rule templates covering brute-force detection, mass resource deletion anomalies, rare subscription-level operations, suspicious deployments, and AAD PowerShell misuse.
- Configured incident-level alert grouping (24-hour window, grouped by Account entity) so that related alerts consolidate into a single actionable incident rather than generating alert fatigue.

The environment is now in a state where it is actively collecting data and evaluating it against six analytics rules, with a documented, evidence-backed trail for every configuration decision made.

---

## 2. Architecture Overview

The PoC follows a simple, linear data flow: infrastructure to ingest → platform to analyze → detection layer → telemetry source. Each layer was built and validated before moving to the next.

![Microsoft Sentinel PoC architecture — Resource Group to Log Analytics Workspace to Microsoft Sentinel to Data Connectors](./assets/architecture.png)

- **Resource Group (RG-Sentinel-PoC)** — the logical container scoping all PoC resources to East US, isolating this work from any other subscription activity.
- **Log Analytics Workspace (LAW-Sentinel-CloudScale)** — the data platform underpinning Sentinel; stores all ingested logs for 90 days on a Pay-as-you-go pricing model.
- **Microsoft Sentinel** — the SIEM layer enabled directly on top of the workspace, providing Analytics, Incidents, and Logs (KQL) capabilities.
- **Data Connectors** — currently just Azure Activity Logs, feeding subscription-level management-plane events (resource creation, role assignments, deletions) into the workspace for analysis.

---

## 3. Deployed Components

| Component | Name / Value | Status |
|---|---|---|
| Resource Group | RG-Sentinel-PoC (East US) | ✅ Deployed |
| Log Analytics Workspace | LAW-Sentinel-CloudScale | ✅ Deployed — 90-day retention, Pay-as-you-go |
| Microsoft Sentinel | Enabled on LAW-Sentinel-CloudScale | ✅ Active |
| Data Connector | Azure Activity Logs | ✅ Connected |
| Analytics Rules (built-in) | 5 templates enabled | ✅ Active |
| Analytics Rules (custom) | Privileged Role Assignment Outside Business Hours | ✅ Saved & enabled |
| Incident Settings | Alert grouping — 24h window, grouped by Account entity | ✅ Configured |

---

## 4. Screenshot Compilation

The following evidence screenshots were captured across Week 1. Insert each corresponding image (e.g. under an `./assets/screenshots/` folder) beside its checklist item below:

- [ ] Day 1 — Resource Group creation confirmation
- [ ] Day 1 — Log Analytics Workspace deployment (retention + pricing tier)
- [ ] Day 1 — Microsoft Sentinel enablement confirmation
- [ ] Day 2 — Azure Activity Logs connector status (Connected)
- [ ] Day 2 — KQL query results (`take 10`, `summarize` by `OperationNameValue`)
- [ ] Day 3 — Custom privilege escalation query, saved and tested
- [ ] Day 4 — Each of the 5 enabled built-in Analytics rules
- [ ] Day 4 — Custom Analytics Rule full configuration (all tabs)
- [ ] Day 5 — Incident settings / alert grouping configuration

---

## 5. Challenges Faced and Resolutions

### 5.1 Sentinel's move to the unified Defender portal

Microsoft has been migrating Sentinel's UI from the standalone Azure portal into the unified Microsoft Defender portal (security.microsoft.com). This meant that documented navigation paths for reaching the Analytics page (Rule templates, in particular) were inconsistent with what rendered — the Defender portal's "Settings > Microsoft Sentinel" admin area looks visually similar to the operational "Microsoft Sentinel > Configuration > Analytics" working page but is a different section entirely.

**Resolution:** identified that Settings > Microsoft Sentinel is purely for workspace connection administration, while the actual working experience lives under the dedicated Microsoft Sentinel item in the main navigation rail. Where the unified portal's routing appeared to stick on a stale page, a hard refresh and a fresh (incognito) session resolved it. This is flagged as an ongoing risk for Week 2, since Microsoft's portal migration is still active and paths may continue to shift.

### 5.2 Understanding data flow without an "app" to connect

Initial assumption was that an application needed to be registered for logs to flow into Sentinel. Clarified that Azure Activity Logs are generated automatically by Azure Resource Manager for any subscription-level action (resource creation, deletions, role changes) — no app registration or agent installation is required. This meant that Day 1's own resource deployments were themselves sufficient to generate the first queryable log entries.

### 5.3 Scoping Defender for Cloud correctly

Needed to determine whether paid Defender for Cloud plans were in scope for this PoC. After comparing the free Foundational CSPM tier against the paid workload-protection plans, determined that paid tiers are unnecessary for a SIEM-focused PoC with no production workloads to protect, and documented this reasoning rather than defaulting to enabling (and being billed for) every available plan.

---

## 6. Questions for Week 2

- Once more data connectors are added (e.g. Azure AD Sign-in Logs), will the existing custom rule and grouping configuration need adjustment, or does it scale automatically across new data sources?
- What is the expected cadence for reviewing and tuning the 5 built-in analytics rules once real (non-PoC) traffic patterns are established?
- Should Week 2 formally track Sentinel's Defender-portal migration status, given Microsoft's stated retirement timeline for the classic Azure portal experience?
- At what point does it make sense to introduce automation (playbooks) versus continuing to review incidents manually?

---

## 7. Documentation & Backup

All screenshots and supporting documents for Week 1 are organized as follows:

```
SentinelPoC-Docs/
├── Day1-ResourceGroup-LAW-Sentinel/
├── Day2-DataConnector-KQLBasics/
├── Day3-AdvancedKQL-MITRE-CustomQuery/
├── Day4-AnalyticsRules/
└── Day5-Incidents-WeeklyReport/
```

> **Recommendation:** maintain a synced copy of this folder structure in cloud storage in addition to the local copy, so no evidence is lost between sessions.
