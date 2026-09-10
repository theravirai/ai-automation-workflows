# Enterprise AI Automation Workflows & Autonomous Agent Systems

[![Infrastructure](https://img.shields.io/badge/Infrastructure-Google%20Cloud%20Platform-4285F4?style=flat-square&logo=googlecloud&logoColor=white)](06-slackops-conversational-rag-copilot/docs/gcp-deployment-guide.md)
[![Security](https://img.shields.io/badge/Security-Cloudflare%20Zero%20Trust-F38020?style=flat-square&logo=cloudflare&logoColor=white)](06-slackops-conversational-rag-copilot/docs/gcp-deployment-guide.md)
[![Containers](https://img.shields.io/badge/Containers-Docker%20%7C%20Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](06-slackops-conversational-rag-copilot/docs/gcp-deployment-guide.md)
[![Orchestrator](https://img.shields.io/badge/Orchestrator-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io)
[![Standard](https://img.shields.io/badge/Standard-Model%20Context%20Protocol%20(MCP)-7928CA?style=flat-square)](04-agentbridge-ai-tool-calling-platform-using-mcp/)
[![Framework](https://img.shields.io/badge/Framework-LangChain-1C3C3C?style=flat-square&logo=langchain&logoColor=white)](03-ai-assistant-orchestrator/)
[![Vector Store](https://img.shields.io/badge/Vector%20Store-Pinecone%20Serverless-000000?style=flat-square&logo=pinecone&logoColor=white)](05-knowledgebridge-enterprise-rag-assistant/)
[![LLM Inference](https://img.shields.io/badge/LLM%20Inference-Groq%20LPU-F55036?style=flat-square&logo=fastapi&logoColor=white)](06-slackops-conversational-rag-copilot/)
[![Embeddings](https://img.shields.io/badge/Embeddings-Google%20Gemini%20(3072--dim)-4285F4?style=flat-square&logo=google&logoColor=white)](05-knowledgebridge-enterprise-rag-assistant/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)

A curated collection of six production-grade AI automation pipelines, autonomous agent orchestrators, and Model Context Protocol (MCP) integrations deployed on Google Cloud Platform (GCP). The systems in this repository address real-world business challenges: automated customer support triage, deterministic financial ledger processing, multi-tool assistant routing, enterprise Model Context Protocol platforms, and collaborative ChatOps knowledge retrieval.

---

## Architectural Blueprint

The following diagram illustrates the multi-tier architecture spanning cloud ingress, self-hosted orchestration, redundant inference models, and vector knowledge stores:

```mermaid
flowchart TB
    subgraph Ingress ["1. Multi-Channel Event Ingress"]
        Slack["Slack API Webhooks<br/>(Mentions, Files, DMs)"]
        Gmail["Gmail Triggers<br/>(Customer Emails & Invoices)"]
        Drive["Google Drive<br/>(Document Intake)"]
        MCPClient["MCP Clients<br/>(Cursor, Claude Desktop, IDEs)"]
    end

    subgraph Infrastructure ["2. Production Cloud Infrastructure (GCP us-central1-a)"]
        CF["Cloudflare Zero Trust Tunnel<br/>(n8n.ravirai.dev / End-to-End TLS 1.3)"]
        Docker["Docker Compose Stack<br/>(n8n Orchestrator / Port 5678)"]
        Systemd["systemd Daemon<br/>(Auto-Restart Service)"]
        Backup["Automated Hot Backups<br/>(Cron to Google Cloud Storage)"]
        
        CF <-->|"Internal HTTP 5678"| Docker
        Systemd -.-> Docker
        Docker -.-> Backup
    end

    subgraph ReasoningMesh ["3. Resilient Multi-LLM Reasoning Mesh"]
        Groq["Groq LPU<br/>(GPT-OSS 120B / LLaMA 3.3 / Qwen 27B)"]
        Gemini["Google Gemini 2.5 Flash Lite<br/>(Zero-Downtime Failover LLM)"]
        Mistral["Mistral Cloud<br/>(Specialized Fallback LLM)"]
        
        Groq -.->|"Automatic Failover"| Gemini
        Groq -.->|"Dynamic Routing"| Mistral
    end

    subgraph Storage ["4. Knowledge Bases & Systems of Record"]
        Pinecone[("Pinecone Serverless<br/>3072-dim Gemini Embeddings")]
        Sheets[("Google Sheets Ledger<br/>Verified Financial Records")]
        Memory[("Window Buffer Memory<br/>10-Turn Thread-Scoped State")]
    end

    Slack -->|"Encrypted Ingress"| CF
    Gmail -->|"Encrypted Ingress"| CF
    Drive -->|"Encrypted Ingress"| CF
    MCPClient -->|"Encrypted Ingress"| CF
    Docker --> ReasoningMesh
    ReasoningMesh <--> Pinecone
    ReasoningMesh <--> Sheets
    ReasoningMesh <--> Memory
```

---

## Project Catalog

| # | Project | Tech Stack | Description | Status |
| :-: | :--- | :--- | :--- | :-: |
| **01** | [**AI-Powered Customer Support Automation**](01-ai-powered-customer-support-automation/) | `n8n`, `Groq (Llama-3)`, `Pinecone RAG`, `Gemini`, `Gmail`, `Slack` | Autonomous inbound email triage, zero-shot intent routing, RAG-grounded customer support resolution, and real-time Slack escalation alerting. | ✅ Production Ready |
| **02** | [**AI Invoice Processing Pipeline**](02-ai-invoice-processing-pipeline/) | `n8n`, `Google Gemini (Flash)`, `Google Drive`, `Google Sheets`, `Gmail`, `JavaScript` | Automated enterprise invoice intake, multimodal PDF text extraction, structured Gemini Information Extractor, deterministic mathematical & date validation, Google Sheets ledger recording, and executive HTML email dispatch. | ✅ Production Ready |
| **03** | [**AI Assistant Orchestrator & Tool Ecosystem**](03-ai-assistant-orchestrator/) | `n8n`, `LangChain`, `Groq (120B)`, `Mistral`, `OpenWeatherMap`, `Frankfurter API`, `Tavily` | Conversational ReAct AI assistant with dual-model failover, multi-turn memory, real-time currency conversion, deterministic sub-workflow weather intelligence, and arithmetic calculations. | ✅ Production Ready |
| **04** | [**AgentBridge: AI Tool Calling Platform using MCP**](04-agentbridge-ai-tool-calling-platform-using-mcp/) | `n8n`, `Model Context Protocol (MCP)`, `Groq`, `Google Gemini`, `Slack`, `Calendar` | Enterprise AI platform decoupling high-level agent reasoning from execution via Model Context Protocol (MCP), featuring dual-model failover, 10-turn memory window, and 8 standardized tools. | ✅ Production Ready |
| **05** | [**KnowledgeBridge: Enterprise RAG Assistant**](05-knowledgebridge-enterprise-rag-assistant/) | `n8n`, `Groq (Qwen 27B)`, `Pinecone RAG (3072-dim)`, `Gemini Embeddings`, `LangChain` | Autonomous enterprise knowledge retrieval assistant featuring document ingestion with 3072-dim embeddings, 10-turn window memory, and strictly grounded conversational retrieval. | ✅ Production Ready |
| **06** | [**SlackOps Conversational RAG Copilot**](06-slackops-conversational-rag-copilot/) | `n8n (GCP Cloud)`, `Slack API`, `Groq (GPT-OSS 120B)`, `Gemini (Flash Fallback)`, `Pinecone RAG` | Enterprise Slack knowledge copilot featuring autonomous in-Slack document vectorization, bot-loop guardrails, 10-turn thread-scoped memory, and in-thread citations. | ✅ Production Ready |

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
