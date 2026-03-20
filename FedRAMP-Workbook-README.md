# 🛡️ FedRAMP Compliance & Continuous Monitoring Workbook for Microsoft Sentinel

A comprehensive Azure Monitor Workbook for Microsoft Sentinel that provides evidence artifacts, continuous monitoring dashboards, and auditor-ready export tables aligned to the **NIST SP 800-53 Rev 5** control framework for **FedRAMP High and Moderate** baselines.

---

## 📋 Overview

FedRAMP authorization requires ongoing evidence that security controls are implemented, monitored, and improving over time. This workbook centralizes that evidence directly from your Sentinel workspace — eliminating manual log pulls, spreadsheet assembly, and last-minute audit scrambles.

It is designed for three audiences:

- **Security Operations teams** doing monthly ConMon reporting
- **Compliance and GRC teams** building evidence packages for 3PAOs and Authorizing Officials
- **Auditors and assessors** who need exportable, time-scoped artifact tables mapped to specific NIST 800-53 controls

---

## ✨ Features

- **10 collapsible sections** covering 8 NIST 800-53 control families
- **47 KQL queries** producing tables, charts, pie charts, and time-series visualizations
- **Dynamic time range selector** — scope evidence to any assessment window (30 / 90 / 365 days)
- **Export-to-Excel** on every evidence table for audit package assembly
- **MITRE ATT&CK tactic coverage** reporting
- **MTTD / MTTR trend analysis** for continuous improvement demonstration (CA-7)
- **Proof of Artifacts section** — five dedicated export tables organized by control family
- No additional licensing required beyond Microsoft Sentinel

---

## 🗂️ Workbook Sections

| # | Section | NIST 800-53 Controls | Key Evidence Provided |
|---|---------|---------------------|----------------------|
| 1 | Executive Dashboard | All families | Incident KPIs, severity trends, MITRE tactic coverage, data source health |
| 2 | Audit & Accountability (AU) | AU-2, AU-3, AU-6, AU-9, AU-11, AU-12 | Log source coverage, event volume, 90-day retention proof, critical event categories |
| 3 | Access Control (AC) | AC-2, AC-3, AC-6, AC-7, AC-17 | Failed logins, privileged role changes, Conditional Access failures, guest access |
| 4 | Identification & Authentication (IA) | IA-2, IA-5, IA-8 | MFA trends, password spray detection, account lockouts, service principal auth |
| 5 | Incident Response (IR) | IR-4, IR-5, IR-6, IR-8 | MTTD, MTTR, incident log, owner assignment, playbook executions, classification |
| 6 | Configuration Management (CM) | CM-2, CM-3, CM-6, CM-8 | Azure resource changes, policy compliance, new resource deployments |
| 7 | System & Information Integrity (SI) | SI-3, SI-4, SI-7 | Threat intelligence hits, Defender recommendations, malware detections, suspicious processes |
| 8 | System & Communications Protection (SC) | SC-5, SC-7, SC-28 | Firewall/NSG denies, DoS-pattern traffic, data exfiltration alerts |
| 9 | Continuous Monitoring (CA) | CA-2, CA-7, CA-9 | MTTD/MTTR week-over-week trends, incident volume trend, ATT&CK coverage |
| 10 | Proof of Artifacts | All families | Five exportable evidence tables scoped to your assessment period |

---

## 📦 Proof of Artifacts — Export Package

Section 10 is purpose-built for audit engagements. It provides five fully exportable tables:

| Export Table | Controls Addressed | Typical Use |
|-------------|-------------------|-------------|
| Complete Incident Log | IR-4, IR-5, IR-6, AU-6 | Primary ConMon artifact; ATO renewal evidence |
| Complete Alert Log | SI-4, AU-2, AU-3 | Detection coverage proof; 3PAO review |
| Sign-in Audit Log | AC-7, IA-2, AU-3 | Authentication evidence; MFA enforcement proof |
| Privileged Access & Role Changes | AC-2, AC-6 | Least privilege evidence; account management |
| Azure Configuration Changes | CM-3, CM-8 | Change control evidence; inventory management |

Set the **Time Range** selector to your assessment period before exporting (e.g., Last 90 days for quarterly reviews, Last Year for annual ATO renewals).

---

## 🔧 Prerequisites

| Requirement | Details |
|-------------|---------|
| Microsoft Sentinel | Any tier; workspace must be active |
| Azure RBAC | Microsoft Sentinel Reader (minimum) to view; Contributor to save |
| Data Connectors | More connectors = more coverage. See table below. |
| Log Retention | Recommend 90+ days to fully populate AU-11 retention evidence |

### Recommended Data Connectors

The workbook queries the following Sentinel tables. Sections gracefully return empty results if a connector is not enabled — you do not need all of these for the workbook to be useful.

| Table | Connector | Controls |
|-------|-----------|---------|
| `SecurityIncident` | Built-in (Sentinel) | IR, CA |
| `SecurityAlert` | Built-in (Sentinel) | SI, AU |
| `SigninLogs` | Microsoft Entra ID | AC, IA |
| `AuditLogs` | Microsoft Entra ID | AC, AU |
| `AzureActivity` | Azure Activity | CM, AU |
| `SecurityEvent` | Windows Security Events | AU, SI, IA |
| `CommonSecurityLog` | Firewall / CEF sources | SC |
| `ThreatIntelligenceIndicator` | Threat Intelligence | SI |
| `SecurityRecommendation` | Microsoft Defender for Cloud | SI |
| `AADServicePrincipalSignInLogs` | Microsoft Entra ID | IA |
| `OfficeActivity` | Microsoft 365 | AU |
| `DeviceEvents` | Microsoft Defender for Endpoint | SI |

---

## 🚀 Installation

### Option 1 — Import via Advanced Editor (Recommended)

1. Navigate to **Azure Portal → Microsoft Sentinel → Workbooks**
2. Click **+ Add workbook**
3. Click the **Edit** button (pencil icon) in the top toolbar
4. Click the **`</>`** (Advanced Editor) icon
5. Select all existing content and delete it
6. Paste the entire contents of `FedRAMP-Sentinel-Workbook.json`
7. Click **Apply**
8. Click **Save**, give the workbook a name (e.g., `FedRAMP Compliance - ConMon`), and choose your Resource Group
9. Click **Save** to confirm

### Option 2 — Deploy via ARM Template

You can wrap the workbook JSON in an ARM template for automated deployment via Azure DevOps, GitHub Actions, or Bicep:

```bash
az deployment group create \
  --resource-group <your-rg> \
  --template-file arm-deploy.json \
  --parameters workspaceName=<your-sentinel-workspace>
```

> See the `/deploy` folder for a ready-to-use ARM template wrapper (coming soon).

---

## 🖥️ Usage

### For Monthly ConMon Reporting

1. Open the workbook and set **Time Range → Last 30 days**
2. Review Section 1 (Executive Dashboard) for leadership summary
3. Check Section 9 (Continuous Monitoring) for MTTD/MTTR trend charts
4. Screenshot or export relevant tables for your monthly ConMon report

### For Annual ATO Renewal / Assessment

1. Set **Time Range → Last Year** (or custom range matching your assessment period)
2. Navigate to **Section 10 — Proof of Artifacts**
3. Export each table to Excel using the ⬇️ icon
4. Organize exports into an evidence folder by control family:
   ```
   /evidence/
     AU/  → audit-log-coverage.xlsx, event-volume.xlsx
     AC/  → failed-logins.xlsx, privileged-access.xlsx
     IA/  → mfa-trends.xlsx, lockouts.xlsx
     IR/  → incident-log.xlsx, mttd-mttr.xlsx
     CM/  → config-changes.xlsx, resource-inventory.xlsx
   ```
5. Attach trend screenshots from Section 9 as CA-7 continuous improvement evidence

### For 3PAO Assessors

Provide assessors with read-only access to the workbook (Microsoft Sentinel Reader role). Direct them to:
- Section 2 (AU) for log completeness verification
- Section 5 (IR) for incident handling process evidence
- Section 10 (Artifacts) for time-scoped evidence export

---

## 📐 Architecture

```
Microsoft Sentinel Workspace (Log Analytics)
│
├── SecurityIncident ──────────────────────► IR, CA sections
├── SecurityAlert ──────────────────────────► SI, AU sections
├── SigninLogs / AuditLogs ─────────────────► AC, IA sections
├── AzureActivity ──────────────────────────► CM, AU sections
├── SecurityEvent ──────────────────────────► AU, IA, SI sections
├── CommonSecurityLog ──────────────────────► SC section
├── ThreatIntelligenceIndicator ────────────► SI section
└── SecurityRecommendation ─────────────────► SI section
                    │
                    ▼
        FedRAMP Compliance Workbook
        ┌─────────────────────────────┐
        │ 10 Sections · 47 KQL Queries│
        │ Dynamic Time Range Filter   │
        │ Export-to-Excel on all tables│
        └─────────────────────────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
   Monthly ConMon        Audit Evidence
   Report Views          Export Package
```

---

## 🗓️ Recommended Review Schedule

| Frequency | Activity | Sections to Use |
|-----------|----------|-----------------|
| Daily | Incident triage | Section 1 (Dashboard), Section 5 (IR) |
| Weekly | Threat review | Sections 7 (SI), 8 (SC) |
| Monthly | ConMon report | Sections 1, 9 (CA trends) |
| Quarterly | Access review | Sections 3 (AC), 4 (IA) + export |
| Annually | ATO evidence package | All sections — Section 10 export |

---

## 🤝 Contributing

Contributions welcome. To add support for additional control families or data connectors:

1. Fork the repository
2. Add your KQL query section to `workbook_gen.py` following the existing pattern
3. Regenerate the workbook JSON: `python3 workbook_gen.py`
4. Test the import in a Sentinel workspace
5. Submit a pull request with the updated JSON and a description of the controls addressed

---

## 📄 License

MIT License — free to use, modify, and distribute. Attribution appreciated but not required.

---

## 🙋 Support & Feedback

Open an issue in this repository for:
- KQL query bugs or table schema changes
- Requests for additional control family coverage
- Questions about FedRAMP evidence requirements
- Connector compatibility issues

---

## 📚 References

- [NIST SP 800-53 Rev 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [FedRAMP Continuous Monitoring Strategy Guide](https://www.fedramp.gov/assets/resources/documents/CSP_Continuous_Monitoring_Strategy_Guide.pdf)
- [Microsoft Sentinel Documentation](https://docs.microsoft.com/en-us/azure/sentinel/)
- [Azure Monitor Workbooks](https://docs.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
- [FedRAMP Automation](https://automate.fedramp.gov/)
