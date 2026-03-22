# 📊 Microsoft Sentinel — Table Coverage & ROI Explorer Workbook

A Microsoft Sentinel workbook that maps every analytics rule to the log tables it queries, measures ingestion cost against detection coverage, and scores rule effectiveness — giving security teams a clear picture of where their Sentinel investment is working and where it isn't.

---

## 🧭 Overview

Most Sentinel environments accumulate analytics rules and data connectors over time without a clear picture of the relationship between them. This workbook answers four questions that are otherwise very hard to answer:

1. **Which analytics rules query a given log table?** — and are they enabled, tuned, and effective?
2. **Which tables have no detection rules at all?** — ingesting data you're paying for but not acting on
3. **What is each table costing you relative to the detections it powers?** — your actual ROI per data source
4. **Which rules are noisy or silent?** — high alert volume with no incident conversion, or enabled rules that never fire

Everything is pulled live from Azure Resource Manager and your Log Analytics workspace — no hardcoded rule lists, no manual maintenance, always current.

---

## ✨ Features

- **Live ARM introspection** — reads analytics rule definitions directly from Azure Resource Graph, not from a static list
- **Four-tab interface** — Table Detail, All Tables Coverage, Cost & ROI, Rule Effectiveness
- **Cross-service joins** — bridges ARM metadata with Log Analytics billing and alert data in a single query
- **Ingestion cost proxy** — uses the built-in `Usage` table to calculate GB/day per table without requiring billing API access
- **Noise identification** — surfaces rules with high alert volume and zero incident conversion
- **Silent rule detection** — finds enabled rules that haven't fired in the selected time window
- **MITRE ATT&CK coverage view** — shows which tactics are covered by rules on any given table
- **Alert → Incident conversion scoring** — measures signal quality per rule with TP/FP/Benign breakdown
- **No additional licensing required** — uses only built-in Sentinel and Log Analytics capabilities

---

## 🗂️ Workbook Tabs

### 🔍 Tab 1 — Table Detail
Select any Sentinel table from the dropdown to see every analytics rule that references it, discovered by parsing the KQL `query` property of each `microsoft.securityinsights/alertRules` ARM resource. Shows enabled/disabled status, severity distribution, MITRE tactic coverage, actual alert firing history for the lookback window, and alert-to-incident conversion rates per rule.

### 📊 Tab 2 — All Tables Coverage
A full coverage matrix across all known Sentinel security tables. Each table is classified as Good Coverage, Partial Coverage, All Disabled, or No Rules — making it immediately obvious which data sources have zero detection logic attached to them. Tables with no rules are the highest-priority ROI gaps.

### 💰 Tab 3 — Cost & ROI
Maps ingestion volume (in GB, from the `Usage` table) against analytics rule coverage for every billable table. Classifies each table into an ROI status:

| Status | Meaning |
|--------|---------|
| 🚨 High Cost, No Coverage | Top-quartile spend, zero detection rules — urgent gap |
| ⚠️ High Cost, Partial Coverage | High spend, rules exist but no High severity coverage |
| ✅ High Cost, Good Coverage | High spend, well-covered — maintain |
| 🟡 Low Cost, No Coverage | Low ingestion, no rules — low priority |
| ✅ Covered | Enabled rules present |

### 🎯 Tab 4 — Rule Effectiveness
Scores every rule that fired in the lookback window by alert-to-incident conversion rate, false positive rate, and confirmed true positives. Includes two dedicated views:
- **Noise Candidates** — rules with 5+ alerts and zero incident conversion (tune, suppress, or retire)
- **Silent Rules** — enabled rules in ARM that produced no alerts in the selected window (broken pipeline, over-tuned threshold, or no matching conditions)

---

## ⚙️ How It Pulls Data — Three Mechanisms

Understanding the data sources helps you know what permissions and connectors are required.

### 1. Azure Resource Graph (`queryType: 1`)
Queries the Azure Resource Manager control plane directly — no log data involved. Reads every `microsoft.securityinsights/alertrules` resource in the subscription and extracts the rule's KQL query text, enabled state, severity, tactics, and last modified time. This is what makes rule discovery live and always current.

### 2. Log Analytics Tables (`queryType: 0`)
Standard KQL queries against the Sentinel workspace:

| Table | Purpose |
|-------|---------|
| `SecurityAlert` | Alert firing history, volume trends, severity |
| `SecurityIncident` | Incident records, alert ID linkage, TP/FP classification |
| `Usage` | Billable ingestion volume in MB/GB by table — the cost proxy |

### 3. Cross-Service Joins via `arg("")`
Several queries use `arg("")` inside a Log Analytics KQL expression to pull ARM Resource Graph data inline and join it against workspace data in a single query. This is what powers the ROI matrix — joining live rule coverage from ARM against ingestion cost from the `Usage` table without leaving the workbook query engine.

---

## 🔧 Prerequisites

| Requirement | Details |
|-------------|---------|
| Microsoft Sentinel | Active workspace |
| Azure RBAC | **Reader** on the subscription (for Resource Graph) + **Microsoft Sentinel Reader** on the workspace |
| Log Analytics Contributor | Required to save the workbook |
| Data retention | 30+ days recommended for meaningful cost and effectiveness data |
| No extra connectors needed | Works against any active Sentinel workspace |

> **Note on permissions:** Azure Resource Graph queries require at minimum **Reader** role at the subscription scope. Users with workspace-only access will see empty results on any tab that uses Resource Graph (Tab 1 rules grid, Tab 2 coverage matrix, Tab 4 silent rules).

---

## 🚀 Installation

### Option 1 — Import via Advanced Editor

1. In the Azure Portal, navigate to **Microsoft Sentinel → Workbooks**
2. Click **+ Add workbook**
3. Click **Edit** (pencil icon) in the top toolbar
4. Click the **`</>`** Advanced Editor icon
5. Delete all existing content and paste the full contents of `sentinel-roi-workbook.json`
6. Click **Apply**, then **Save**
7. Name the workbook (e.g., `Table Coverage & ROI Explorer`) and choose your Resource Group

### Option 2 — Deploy via Azure CLI

```bash
az deployment group create \
  --resource-group <your-sentinel-rg> \
  --template-file arm-deploy.json \
  --parameters workspaceName=<your-workspace-name>
```

> ARM template wrapper available in the `/deploy` directory.

---

## 🖥️ Usage Guide

### Finding ROI Gaps

1. Open the workbook and go to the **💰 Cost & ROI** tab
2. Set **Alert Lookback** to **Last 30 days**
3. Look for rows flagged **🚨 High Cost, No Coverage** — these are tables where you are paying for ingestion with zero detection rules attached
4. Note the table names, then switch to the **📊 All Tables Coverage** tab and confirm no rules reference them
5. Use the **Content Hub** in Sentinel to install the relevant solution, or write custom analytics rules for the identified tables

### Identifying Noisy Rules

1. Go to the **🎯 Rule Effectiveness** tab
2. Scroll to the **Noise Candidates** section
3. Any rule with 5+ alerts and 0% conversion over 30 days is a tuning candidate
4. Filter by Severity to prioritize — a High severity rule generating 50 alerts with no incidents is more urgent than an Informational rule

### Table-Level Deep Dive

1. Go to the **🔍 Table Detail** tab
2. Select a table from the **🗂️ Select Table** dropdown
3. Review the rules grid — check which rules are enabled, their MITRE coverage, and their query frequency
4. Review the **Alert Activity** timechart to see when rules fired
5. Review the **Incident Conversion** table to identify which rules on this table are producing actionable incidents vs. noise

---

## 📐 Architecture

```
Azure Subscription (ARM)
│
└── microsoft.securityinsights/alertRules
        ↓ Azure Resource Graph (queryType: 1)
        ↓ arg("") cross-join inside KQL
        ↓
Log Analytics Workspace
│
├── SecurityAlert       → Alert volume, severity, firing history
├── SecurityIncident    → Conversion rates, TP/FP, classification
└── Usage               → Billable GB per table (cost proxy)
        ↓
        ↓ KQL joins
        ↓
┌──────────────────────────────────────────┐
│     Sentinel ROI Workbook                │
│  ┌──────────┬──────────┬───────────────┐ │
│  │  Table   │Coverage  │ Cost & ROI    │ │
│  │  Detail  │ Matrix   │   Matrix      │ │
│  └──────────┴──────────┴───────────────┘ │
│  ┌─────────────────────────────────────┐ │
│  │      Rule Effectiveness Scorecard   │ │
│  │  Noise Candidates │ Silent Rules    │ │
│  └─────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

---

## 🔄 Supported Tables

The workbook's dropdown and coverage matrix include the following tables out of the box. Add additional tables by editing the `jsonData` array in the `SelectedTable` parameter and the `KnownTables` datatable in the coverage query.

| Category | Tables |
|----------|--------|
| Identity & Access | `SigninLogs`, `AuditLogs`, `AADNonInteractiveUserSignInLogs`, `AADServicePrincipalSignInLogs`, `AADManagedIdentitySignInLogs` |
| Windows / Endpoint | `SecurityEvent`, `WindowsEvent`, `Syslog`, `DeviceProcessEvents`, `DeviceNetworkEvents`, `DeviceFileEvents`, `DeviceLogonEvents`, `DeviceRegistryEvents`, `DeviceAlertEvents` |
| Network | `CommonSecurityLog`, `DnsEvents`, `NetworkAccessTraffic` |
| Microsoft 365 | `OfficeActivity`, `EmailEvents`, `EmailAttachmentInfo` |
| Azure | `AzureActivity`, `AzureDiagnostics` |
| Sentinel Native | `SecurityAlert`, `SecurityIncident`, `BehaviorAnalytics`, `ThreatIntelligenceIndicator`, `ThreatIntelIndicators` |
| Identity Protection | `IdentityLogonEvents`, `IdentityQueryEvents`, `IdentityDirectoryEvents` |
| Cloud Apps | `CloudAppEvents`, `McasShadowItReporting` |
| Multi-Cloud | `AWSCloudTrail`, `GCPAuditLogs` |

---

## 🤝 Contributing

To add support for additional tables or new analysis views:

1. Fork the repository
2. Add new table names to the `SelectedTable` parameter `jsonData` array and to the `KnownTables` datatable in the coverage query
3. For new analysis tabs, add a `conditionalVisibility` group item and a corresponding entry in the `selectedTab` parameter `jsonData`
4. Test in a Sentinel workspace with the Advanced Editor import method
5. Submit a pull request with a description of what was added and why

---

## 🐛 Known Limitations

- **Resource Graph scope** — by default, ARG queries run against the subscription associated with the workbook's `fallbackResourceIds`. In multi-subscription environments, you may need to adjust the ARG scope or use cross-tenant ARG queries.
- **Rule name matching** — alert-to-rule joins use `AlertName == RuleName` string matching. If a rule's display name has been changed after alerts were generated, historical alerts won't join correctly.
- **Usage table accuracy** — the `Usage` table records ingestion in MB and may have up to a 2-hour delay. GB figures are approximate and should be used for relative comparison, not billing reconciliation.
- **Fusion & NRT rules** — Fusion and Near-Real-Time rules may not appear in the ARM `alertrules` resource type depending on workspace configuration. Scheduled rules have full coverage.

---

## 📄 License

MIT License — free to use, modify, and distribute. Attribution appreciated but not required.

---

## 📚 References

- [Azure Resource Graph documentation](https://learn.microsoft.com/en-us/azure/governance/resource-graph/overview)
- [Microsoft Sentinel Analytics Rules](https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-built-in)
- [Log Analytics Usage table](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/usage)
- [Azure Monitor Workbooks](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
- [arg() function in KQL](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/arg-function)
