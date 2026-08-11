# Production Assets & Visual Demonstrations

This directory contains the visual artifacts, live recordings, and production screenshots validating the **SlackOps Conversational RAG Copilot** running live on Google Cloud Platform (GCP) and integrated with the Slack workspace (`#slack-enterprise-rag-assistant`).

---

## 📸 Asset Inventory

| File Name | Format | Dimensions | Description |
| :--- | :---: | :---: | :--- |
| **`slack-interactive-chat-demo.gif`** | GIF | 1920×954 | Live recording demonstrating multi-turn thread-aware retrieval, structured markdown answers, citations, and hallucination boundary refusal. |
| **`slack-document-ingestion-demo.gif`** | GIF | 1920×954 | Live recording demonstrating drag-and-drop document upload into Slack, automated API download, vectorization, and in-thread confirmation. |
| **`workflow-canvas-overview.png`** | PNG | 2608×1312 | Complete 17-node production canvas on GCP n8n organized into 3 modular operational node groups. |
| **`slack-in-thread-ingestion-screenshot.png`** | PNG | 1920×954 | Full-channel screenshot capturing the upload of `01-incident-response-and-oncall-playbook.pdf` and the instant in-thread confirmation. |
| **`slack-rag-conversation-screenshot.png`** | PNG | 1920×954 | High-resolution capture of the multi-turn SRE query, war room roles, and out-of-scope travel policy refusal. |

---

## 🎙️ Verbatim Production Dialogue Transcripts

### 1. Document Ingestion Flow (`slack-in-thread-ingestion-screenshot.png`)
- **User Action:** Ravi Rai uploads `01-incident-response-and-oncall-playbook.pdf` into `#slack-enterprise-rag-assistant`
- **User Prompt:**
  ```text
  @n8n please index this playbook into the knowledge base
  ```
- **Bot Response (In-Thread):**
  ```text
  ✅ Document Indexed Successfully
  File: 01-incident-response-and-oncall-playbook.pdf
  Status: Ingested and embedded into the enterprise vector store (rag-n8n).
  ```

---

### 2. Multi-Turn RAG Retrieval (`slack-rag-conversation-screenshot.png`)

#### Query 1: Incident SLA & Communication Protocol
- **User Prompt:**
  ```text
  @n8n Assistant what is our response SLA and communication protocol for a Sev-1 incident?
  ```
- **Bot Response:**
  ```text
  - **Response SLA:** Initiate response within **< 15 minutes** of detection.
  - **Communication cadence:** Post a **status update every 30 minutes** in the dedicated incident channel **#incident-war-r** (incident war-room).

  Source: 01-incident-response-and-oncall-playbook.pdf
  ```

#### Query 2: War Room Roles & Pager Escalation
- **User Prompt:**
  ```text
  @n8n What roles are required in the war room and how do we page the on-call engineer?
  ```
- **Bot Response:**
  ```text
  **War-Room Roles**
  - **Incident Commander (IC)** – Owns the incident workflow, assigns tasks, coordinates communication, and shields responders from distractions.
  - **Technical Lead (TL)** – Drives root-cause investigation, log analysis, telemetry review, and executes rollbacks or hot-fixes.
  - **Communications Lead (CL)** – Crafts public status-page updates and executive summaries for leadership channels.

  **Paging the On-Call Engineer**
  - Trigger the on-call responder with **@oncall-sre** in the incident channel.
  - Alternatively, run the Slack shortcut **/pager duty trigger sev-1** to page the appropriate on-call engineer directly.

  Source: 01-incident-response-and-oncall-playbook.pdf
  ```

#### Query 3: Hallucination Guardrail & Out-of-Scope Refusal
- **User Prompt:**
  ```text
  @n8n What is our policy on international business class flights?
  ```
- **Bot Response:**
  ```text
  I could not find verified documentation regarding your request in the enterprise knowledge base. You can upload relevant policy or tech specs directly into this channel to index them.
  ```
