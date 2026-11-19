# Automated DevOps Incident & PR Triage Loop

An event-driven workflow automation pipeline that bridges **GitHub Issues**, **Generative AI (Structured Outputs)**, and **Google Workspace** to eliminate manual incident triage, reduce Mean Time to Acknowledge (MTTA), and maintain an automated engineering metrics log.

---

## Problem Statement
In fast-moving DevOps and Infrastructure-as-Code (IaC) environments, incoming incident reports and CI/CD failure issues often sit untriaged in GitHub. Engineers waste time manually reading stack traces, classifying severity, notifying stakeholders, and updating tracking sheets.

## Solution
This project implements a **5-step closed-loop automation pipeline** using Zapier:
1. **Trigger (GitHub):** Monitors the repository for newly created issues using a standardized `.github/ISSUE_TEMPLATE/incident_report.md`.
2. **AI Triage (AI by Zapier):** Analyzes the issue title and error log using an SRE-focused prompt with **strict structured outputs** (`severity`, `github_label`, `summary`, `checklist`).
3. **Metrics Logging (Google Sheets):** Appends every triaged incident to `DevOps_Triage_Dashboard` (`Triage_Log`) as a lightweight DORA/operational metrics log.
4. **Stakeholder Alerting (Gmail):** Dispatches a formatted email alert with severity classification, plain-language summary, verification checklist, and direct GitHub link.
5. **Closed-Loop Labeling (GitHub):** Automatically applies the corresponding severity label (`severity:high`, `severity:medium`, `severity:low`) back onto the originating GitHub Issue.

---

## Architecture

```mermaid
flowchart LR
    A[GitHub Issue Created] -->|Polling / Trigger| B[Zapier Workflow]
    B --> C[AI by Zapier<br/>Structured Output Triage]
    C -->|Severity, Summary, Checklist| D[Google Sheets<br/>DevOps_Triage_Dashboard]
    C -->|Formatted Alert| E[Gmail<br/>Engineering Inbox]
    C -->|Exact Label Mapping| F[GitHub API<br/>Auto-Apply Severity Label]