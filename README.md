<div align="center">

# 🏥 MedDocs 360

### Medical Document Tracking & Automated Multi-Channel Reporting System

[![Power Automate](https://img.shields.io/badge/Real--Time%20Tracking-Power%20Automate-0066FF?style=for-the-badge&logo=microsoftpowerautomate&logoColor=white)](https://powerautomate.microsoft.com/)
[![n8n](https://img.shields.io/badge/Reporting%20Pipeline-n8n-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io/)
[![Microsoft 365](https://img.shields.io/badge/Ecosystem-Microsoft%20365-D83B01?style=for-the-badge&logo=microsoft&logoColor=white)](https://www.microsoft.com/en-us/microsoft-365)
[![Box](https://img.shields.io/badge/Cloud%20Archive-Box-0061D5?style=for-the-badge&logo=box&logoColor=white)](https://www.box.com/)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue?style=for-the-badge&logo=semver&logoColor=white)](https://semver.org/)

</div>

> **MedDocs 360** is a dual-layer automation system built for [Chambers Law Firm](https://www.chamberslaw.com/) that tracks medical questionnaire documents through their full lifecycle — **Blank → Completed → Reviewed** — in real time, and auto-generates weekly executive reports delivered as **PDF** and **Excel** to cloud storage and email.
> - **Real-Time Tracking:** Two Power Automate cloud flows instantly log document events from OneDrive into a centralized Excel database with full deduplication.
> - **Weekly Reporting:** An n8n scheduled pipeline filters, compiles, and exports a premium HTML/PDF report and an Excel digest — then emails a stat-card summary via Outlook.
> - **Zero Manual Effort:** From file upload to executive inbox, the entire pipeline runs autonomously with no human intervention required.
>
> *Track every document. Report every week. Automatically.*

---

## 📑 Table of Contents

- [Why MedDocs 360?](#-why-meddocs-360)
- [Database & CRM Engine](#-database--crm-engine)
- [Architecture & Workflows](#-architecture--workflows)
  - [Layer 1 — Real-Time Tracking (Power Automate)](#layer-1--real-time-tracking-power-automate)
  - [Layer 2 — Weekly Reporting Pipeline (n8n)](#layer-2--weekly-reporting-pipeline-n8n)
- [Sample Reports & Notifications](#-sample-reports--notifications)
- [Technical Stack](#️-technical-stack)

- [License](#-license)
- [Author](#-author)

---

## 🎯 Why MedDocs 360?

| Problem | The MedDocs 360 Solution |
|---|---|
| **Manual document logging** | Power Automate flows trigger instantly on OneDrive file events and log records automatically — no data entry required. |
| **Duplicate records in the database** | Deduplication using the unique `file_id` (OneDrive Graph API) prevents the same document from being logged twice. |
| **No visibility into weekly activity** | A scheduled n8n pipeline filters the last 7 days of records per sheet and compiles a full metrics report every week. |
| **Reports locked in one format** | The pipeline outputs both a **PDF** (via Api2Pdf) and an **Excel** `.xlsx` file, archived to Box and delivered by email. |
| **Inconsistent client name formatting** | Custom JavaScript parsing normalizes names (`Lastname, Firstname`) and extracts doctor names from nested folder paths automatically. |

---

## 📊 Database & CRM Engine

At the core of the system is `n8n database` — a centralized Excel file with three structured tracking tables that serve as the single source of truth for the entire pipeline.

| Column | Description |
|---|---|
| `file_id` | Unique OneDrive / Graph API file identifier — used for deduplication |
| `matter_number` | Legal matter case number |
| `client_name` | Client name, normalized as `Lastname, Firstname` |
| `doctor_name` | Doctor / provider first name, extracted from the folder name |
| `created_at` / `uploaded_at` | Date of upload, formatted as `MM/dd/yyyy` |
| `created_by` / `uploaded_by` | Uploader's display name |
| `filename` | Original document filename |

**Sheets:**
- `blank docs`
- `completed docs`
- `reviewed docs`

![Database — Blank Docs](images/Database%20-%20blank%20docs.png)
![Database — Completed Docs](images/Database%20-%20completed%20docs.png)
![Database — Reviewed Docs](images/Database%20-%20reviewed%20docs.png)

---

## ⚡ Architecture & Workflows

The system is composed of two independent automation layers.

### Layer 1 — Real-Time Tracking (Power Automate)

Two cloud flows respond instantly to file creation events on OneDrive for Business.

**Flow 1: Track Blank Documents**
Triggered when a new file appears in the "CLF Medical" OneDrive folder. Extracts `File ID`, `Matter Number`, `Client Name` (from folder, normalized as `Lastname, Firstname`), `Doctor Name` (first name, split by `-`), `Uploaded At` (`MM/dd/yyyy`), `Uploaded By`, and `Filename`. Performs deduplication via `file_id` before writing to the **Blank Docs** table.

![Track Blank Documents Workflow](images/MedDocs%20360%20-%20Track%20Blank%20Documents%20workflow%20-%20Microsoft%20Power%20Automate.png)

---

**Flow 2: Track Completed & Reviewed Documents**
Triggered on new file creation in the same OneDrive folder. Uses **dynamic path detection** to differentiate between files in `/Completed Client Questionnaires/` vs. nested subfolders (e.g., `/COMPLETED=by Angela or Abigail/`). Extracts `Client Name` from the filename (`Lastname Firstname`, split by `-`) and routes each record to either the **Completed Docs** or **Reviewed Docs** table based on the resolved path. Deduplication enforced via `file_id`.

![Track Completed and Reviewed Documents Workflow](images/MedDocs%20360%20-%20Track%20Completed%20and%20Reviewed%20Documents%20workflow%20-%20Microsoft%20Power%20Automate.png)

---

### Layer 2 — Weekly Reporting Pipeline (n8n)

A scheduled n8n workflow runs every week to compile, export, and deliver the executive report.

- **7-Day Dynamic Filtering:** Three independent filter code nodes handle date-range filtering per sheet — supporting both ISO date strings (`2026-07-21T02:06:11Z`) and Excel serial numbers (`46224`). Each tags records with a `_source` field (`blank`, `completed`, `reviewed`).
- **HTML Report Generation:** A compile node generates a premium HTML report and outputs structured metrics (`totalFiles`, `totalBlank`, `totalCompleted`, `totalReviewed`, `periodStart`, `periodEnd`) for downstream use.
- **PDF Export:** Converted from HTML via Api2Pdf and uploaded to Box.
- **Excel Export:** Mapped via a dedicated Code node, converted to `.xlsx`, and uploaded to Box.
- **Outlook Email Digest:** An HTML email with 4 color-coded metric stat cards, a document breakdown table, and a direct Box folder link — sent automatically after both uploads complete.

![Report Generation Workflow](images/MedDocs%20360%20-%20Report%20Generation%20workflow%20-%20n8n.png)

---

## 📬 Sample Reports & Notifications

Sample output files from the automated weekly pipeline:

- [`CLF-Medical-Weekly-Report-07-23-2026.pdf`](sample-reports/CLF-Medical-Weekly-Report-07-23-2026.pdf)
- [`CLF-Medical-Weekly-Report-07-23-2026.xlsx`](sample-reports/CLF-Medical-Weekly-Report-07-23-2026.xlsx)

The Outlook email digest includes 4 color-coded metric stat cards and a direct Box folder link:

![Email Notification](images/Email%20Notification.png)

---

## 🛠️ Technical Stack

| Component | Technology | Role |
|:---|:---|:---|
| **Real-Time Tracking** | [Microsoft Power Automate](https://powerautomate.microsoft.com/) | Instant event-driven flows triggered on OneDrive file creation |
| **Reporting Pipeline** | [n8n](https://n8n.io/) | Weekly scheduled batch pipeline for filtering, compiling, and exporting reports |
| **Source File Storage** | [Microsoft OneDrive for Business](https://www.microsoft.com/en-us/microsoft-365/onedrive/onedrive-for-business) | Watched folder where documents are uploaded by staff |
| **Central Database** | [Microsoft Excel Online](https://www.microsoft.com/en-us/microsoft-365/excel) | Three-sheet tracking database (`n8n database`) |
| **Cloud Archive** | [Box](https://www.box.com/) | Destination for generated PDF and Excel weekly reports |
| **PDF Engine** | [Api2Pdf](https://www.api2pdf.com/) | Converts the HTML report template to a polished PDF |
| **Email Notifications** | [Microsoft Outlook](https://outlook.microsoft.com/) | Delivers the HTML stat-card email digest to stakeholders |
| **Logic & Scripting** | JavaScript (Node.js ES6+) | n8n Code nodes for date filtering, name parsing, and report mapping |

---

## 🔒 Data Integrity & Compliance

- **Deduplication by File ID:** Every record is keyed on the unique OneDrive Graph API `file_id`, ensuring no document is ever logged twice regardless of filename or timing.
- **Dual-Format Archiving:** Reports are stored in both PDF and Excel formats on Box, creating an auditable, version-controlled record of weekly activity.
- **Proprietary & Confidential:** This system is the exclusive property of [Chambers Law Firm, P.A.](https://chamberslaw.com) All rights reserved.

---

---

## 📄 License

This project is **Proprietary and Confidential** — see the [LICENSE](LICENSE) file for details.

- **Owner:** [Chambers Law Firm, P.A.](https://chamberslaw.com) © 2026. All rights reserved.
- **Prohibited:** Dissemination, reproduction, or use of this material without prior written permission from Chambers Law Firm is strictly forbidden.

---

## 👤 Author

<div align="center">

**Md. Rifat Aknda**

*AI/ML & Automation Engineer · AI/ML & IoT Researcher*

---

[![GitHub](https://img.shields.io/badge/GitHub-rifatmilon-181717?style=for-the-badge&logo=github)](https://github.com/rifatmilon)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Md.%20Rifat%20Aknda-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/rifatmilon/)
[![Fiverr](https://img.shields.io/badge/Fiverr-rifatmilon-1DBF73?style=for-the-badge&logo=fiverr&logoColor=white)](https://www.fiverr.com/rifatmilon)
[![Google Scholar](https://img.shields.io/badge/Google%20Scholar-Md.%20Rifat%20Aknda-4285F4?style=for-the-badge&logo=googlescholar)](https://scholar.google.com/citations?user=qPC0U2gAAAAJ)

</div>