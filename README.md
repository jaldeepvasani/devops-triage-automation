# Automated DevOps Incident & PR Triage Loop

An event-driven workflow automation repository demonstrating two architectural approaches to eliminating manual GitHub incident triage and reducing Mean Time to Acknowledge (MTTA):

1. **v1.0 — Cloud SaaS Pipeline (Zapier + Google Workspace):** Rapid prototyping with managed SaaS orchestration, Google Sheets metrics logging, Gmail alerting, and automated GitHub labeling.
2. **v2.0 — Self-Hosted, 100% Local AI Pipeline (n8n + Docker + Ollama):** A zero-cloud, DSGVO/GDPR-compliant evolution where all incident logs and AI inference stay strictly on local hardware at €0 API cost.

---

## Problem Statement
In fast-moving DevOps and Infrastructure-as-Code (IaC) environments, incoming incident reports and CI/CD failure issues often sit untriaged in GitHub. Engineers waste time manually reading stack traces, classifying severity, notifying stakeholders, and updating tracking logs.

Furthermore, in European enterprise environments (under **DSGVO/GDPR**), sending raw production error logs, internal IP addresses, and infrastructure topologies to third-party cloud LLM APIs is often restricted.

---

## Architecture Comparison: v1.0 vs. v2.0

| Feature | v1.0 (Zapier SaaS) | v2.0 (Self-Hosted n8n + Ollama) |
| :--- | :--- | :--- |
| **Orchestration** | Zapier (Cloud) | n8n (Self-Hosted via Docker Compose) |
| **AI Inference** | AI by Zapier (Cloud LLM) | Ollama (`qwen2.5:3b`) running locally via Metal GPU |
| **Data Privacy** | Logs sent to external cloud APIs | **100% Local / DSGVO-Compliant** (zero external AI calls) |
| **Webhook Ingestion** | Managed polling/trigger | Real-time HTTPS via Cloudflare Quick Tunnel (`cloudflared`) |
| **Loop Prevention** | Trigger-level filtering | Dedicated n8n Filter node (`action == "opened"`) |
| **Cost** | SaaS subscription / task limits | **€0** (unlimited local executions) |

---

## v2.0: Self-Hosted Local AI Pipeline (n8n + Ollama)

### Solution Overview
This pipeline processes real-time GitHub webhooks locally using a **5-node n8n workflow**:
1. **GitHub Trigger:** Receives real-time `issues` webhook events via Cloudflare Quick Tunnel.
2. **Filter (`action == "opened"`):** Drops comment and label update events in ~230ms to prevent infinite webhook loops and wasted LLM compute.
3. **Basic LLM Chain + Ollama Chat Model (`qwen2.5:3b`):** Connects from the Docker container to the host machine (`http://host.docker.internal:11434`) to analyze the issue title and error log using a strict SRE JSON prompt.
4. **Code in JavaScript:** Strips any stray markdown fences, runs `JSON.parse()`, and merges the structured triage output with the originating `issue_number`.
5. **GitHub — Create Comment:** Posts a formatted Markdown triage report (Severity, Category, Suggested Label, Root Cause Summary, and Recommended Next Steps) directly onto the GitHub issue in ~10 seconds.

### v2.0 Architecture

```mermaid
flowchart LR
    A[GitHub Issue Opened] -->|HTTPS Webhook via Cloudflare Tunnel| B[n8n Trigger<br/>Docker :5678]
    B --> C{Filter Node<br/>action == opened?}
    C -->|No - Drop in ~230ms| X[Stop Execution]
    C -->|Yes| D[Basic LLM Chain]
    D <-->|host.docker.internal:11434| E[Local Ollama<br/>qwen2.5:3b]
    D -->|Raw JSON| F[Code Node<br/>JavaScript Parser]
    F -->|Formatted Markdown| G[GitHub API<br/>Post Triage Comment]