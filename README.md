# Enterprise AI Automation Workflows

![Workflows](https://img.shields.io/badge/Status-Active%20Portfolio-success?style=for-the-badge)
![n8n](https://img.shields.io/badge/Platform-n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![AI Agents](https://img.shields.io/badge/Architecture-Agentic%20RAG-7928CA?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

A curated collection of production-grade **AI automation workflows**, autonomous agent pipelines, and enterprise integrations designed for real-world business impact. Built with **n8n**, **Groq**, **Pinecone**, **Google Gemini**, and modern enterprise APIs.

---

## Project Catalog

| # | Project | Tech Stack | Description | Status |
| :-: | :--- | :--- | :--- | :-: |
| **01** | [**AI-Powered Customer Support Automation**](01-ai-powered-customer-support-automation/) | `n8n`, `Groq (Llama-3)`, `Pinecone RAG`, `Gemini`, `Gmail`, `Slack` | Autonomous inbound email triage, zero-shot intent routing, RAG-grounded customer support resolution, and real-time Slack escalation alerting. | ✅ Production Ready |
| **02** | [**AI Invoice Processing Pipeline**](02-ai-invoice-processing-pipeline/) | `n8n`, `Google Gemini (Flash)`, `Google Drive`, `Google Sheets`, `Gmail`, `JavaScript` | Automated enterprise invoice intake, multimodal PDF text extraction, structured Gemini Information Extractor, deterministic mathematical & date validation, Google Sheets ledger recording, and executive HTML email dispatch. | ✅ Production Ready |
| **03** | *Upcoming Workflow* | `n8n`, `LangChain`, `Webhooks` | *In development* | ⏳ Queued |

---

## 🛠️ Architecture & Engineering Standards

Every workflow in this repository is built following strict enterprise software and agentic design principles:

- **Strict RAG Grounding:** LLM agents are bounded to verified vector knowledge bases with zero-hallucination prompts and identical ingestion/retrieval embedding spaces.
- **Resilient Event Routing:** Intent classifiers maintain mutually exclusive domain boundaries with explicit catch-alls and no-ops to eliminate silent message drops.
- **Asynchronous & Non-Blocking:** Multi-channel alerting (Slack, email, CRM) is decoupled from core transactional paths to prevent system stalls.
- **Deterministic Data Contracts:** Normalized data nodes (e.g. `Edit Fields`) sit between triggers and reasoning models to facilitate modular testing with pinned mock payloads.
- **Zero Secrets in Code:** Credentials and tokens are managed via environment variables and instance secrets - never committed to version control.

---

## Getting Started

Each project folder is self-contained and includes:
1. **`README.md`**: Complete system architecture, live test proofs, engineering highlights, and step-by-step setup guide.
2. **`workflow.json`**: Ready-to-import, sanitized n8n workflow definition.
3. **`knowledge-base/`**: Sample source data, chunking files, or policy documents needed for RAG vector stores.
4. **`assets/`**: Canvas screenshots and visual verification evidence.

To explore a workflow, click on any project folder above or navigate to the directory directly.

---

## Author

**Ravi Rai**
- GitHub: [@theravirai](https://github.com/theravirai)
- Repository: [ai-automation-workflows](https://github.com/theravirai/ai-automation-workflows)
