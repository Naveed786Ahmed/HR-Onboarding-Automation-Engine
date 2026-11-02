# Enterprise-Grade Autonomous HR Onboarding Engine

> **A Stateless, Event-Driven & Fault-Tolerant ATS/Onboarding Pipeline Built with Database-Driven State Machines**

[![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io/)
[![PostgreSQL](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?style=flat-square&logo=supabase)](https://supabase.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-gpt--4o--mini-412991?style=flat-square&logo=openai)](https://openai.com/)
[![DocuSign](https://img.shields.io/badge/DocuSign-E--Sign%20API-FF0033?style=flat-square&logo=docusign)](https://www.docusign.com/)
[![Slack](https://img.shields.io/badge/Slack-Interactive%20Block%20Kit-4A154B?style=flat-square&logo=slack)](https://slack.com/)

---

<!-- Demo Video Section -->
<div align="center">

[![Watch Enterprise HR Engine Demo Video](https://img.youtube.com/vi/gcYUnXp-SOg/maxresdefault.jpg)](https://www.youtube.com/watch?v=gcYUnXp-SOg)

> 🎥 **[Click Here to Watch Full 1080p HD System Walkthrough Video](https://www.youtube.com/watch?v=gcYUnXp-SOg)**
</div>

---

## 🎯 The Business Problem & Engineering Context

Mid-to-large-scale enterprises processing high-volume candidate onboarding face critical operational and backend challenges:

1. **High Latency Resume Screening:** Manual processing of unstructured resumes leads to hiring delays and inconsistent candidate evaluations.
2. **Contract Lifecycle Friction:** Manual NDA and contract generation creates unnecessary administrative overhead.
3. **The Long-Running Async State Problem (Core Backend Challenge):** Standard automation engines rely on volatile in-memory timers or long-polling execution. If a candidate takes 3–5 days to sign an agreement, server reboots or memory clears result in lost execution state.
4. **Lack of Human-in-the-Loop (HITL) Security:** Fully automated pipelines often lack secure manager intervention gates before provisioning enterprise assets.
5. **Auditability & Compliance:** Regulatory standards require immutable, timestamped audit logs for all onboarding state transitions.

---

## 💡 The Architectural Solution

This system solves backend state loss by adopting a **Database-Driven State Machine Pattern** using **n8n**, **Supabase (PostgreSQL)**, **DocuSign Connect**, and **Slack Block Kit**.

Instead of holding execution memory open during multi-day signing delays, all state transitions are persisted transactionally in PostgreSQL. Decoupled webhooks listen for external state triggers, ensuring **zero-memory dependency**, **high fault tolerance**, and **end-to-end auditability**.

---

## 🏛️ System Architecture

![Architecture Diagram](docs/architecture-diagram.svg)

## ⚙️ How It Works (The Complete Flow)

### 1. Candidate Intake & AI Screening (Phase 1)

- The candidate uploads their resume PDF through a secure web form.
- n8n extracts the raw text from the binary file and sends it to OpenAI (gpt-4o-mini) with a strict JSON schema.
- The AI parses the resume, evaluates experience, and returns a structured qualification score.

### 2. Database Persistence & Contract Dispatch (Phase 2)

- The parsed candidate data and AI score are stored inside Supabase PostgreSQL.
- The system immediately triggers the DocuSign API to dispatch an employment agreement.
- The candidate's record state in the database is updated to `Pending_Signature`, and the initial workflow closes safely, consuming zero server resources while waiting.

### 3. Asynchronous Webhook & State Transition (Phase 3)

- Days later, once the candidate signs the agreement, DocuSign fires a secure webhook back to our system.
- The workflow catches the event, updates the candidate status to `Signed` in Supabase, and posts an interactive card to Slack using Block Kit.

### 4. Human-in-the-Loop (HITL) & IT Provisioning (Phase 4)

The hiring manager reviews the candidate details directly in Slack and clicks **Approve** or **Reject**.

Based on the button click, a conditional switch routes the logic:

**If Approved:**
- Status updates to `Approved`.
- A mock IT provisioning HTTP request runs.
- An audit log is written to Supabase.
- The Slack message updates in-place to show approval.

**If Rejected:**
- Status updates to `Rejected`.
- The Slack message updates accordingly.

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Orchestration | n8n (Self-Hosted) |
| Database | Supabase (PostgreSQL) |
| AI Engine | OpenAI API (gpt-4o-mini) |
| E-Sign | DocuSign REST API & Connect Webhooks |
| Notifications / UI | Slack API (Block Kit) |

---

## 📂 Repository Layout

```text
hr-onboarding-automation/
│
├── workflows/
│   ├── phase1_ingestion_ai.json
│   └── phase2_docusign_slack_approval.json
│
├── docs/
│   ├── architecture-diagram.png
│   └── database-schema.sql
│
├── .env.example
├── .gitignore
└── README.md
```

---

## 🚀 Getting Started

### 1. Setup Database

Run the SQL queries provided in `docs/database-schema.sql` inside your Supabase SQL Editor to create the `candidates` and `audit_logs` tables.

### 2. Import Workflows

- Open your n8n dashboard.
- Import both JSON files located inside the `workflows/` directory.

### 3. Environment Variables

Create a local `.env` file using `.env.example` as a template and provide your API credentials for OpenAI, Supabase, DocuSign, and Slack.

---

## 👨‍💻 Author

**Naveed Ahmed**

AI & Automation Developer

- GitHub: [@Naveed786Ahmed](https://github.com/Naveed786Ahmed)
- LinkedIn: Naveed Ahmed
