# Sample Enterprise Knowledge Documents

This folder contains pre-compiled, production-style enterprise policy and architecture PDFs ready for ingestion into the **KnowledgeBridge** RAG vector repository.

---

## Document Catalog

| File | Document Title | Domain / Category | Key Test Topics Covered |
| :--- | :--- | :--- | :--- |
| **`01-enterprise-ai-governance-and-acceptable-use-policy.pdf`** | Enterprise AI Governance & Acceptable Use Policy | AI Compliance & Governance | • Approved model registry (Groq, Gemini, Claude)<br/>• PII masking standards (PCI-DSS, HIPAA)<br/>• RAG grounding & anti-hallucination rules<br/>• Human escalation thresholds (>$500 refunds)<br/>• SOC-2 log retention (365 days) |
| **`02-cloud-infrastructure-sla-and-security-standards.pdf`** | Cloud Infrastructure & Security Standards | Infrastructure & DevOps | • 99.99% monthly uptime SLA<br/>• Pinecone serverless 3072-dimensional vector indexing<br/>• Data encryption at rest (AES-256) & transit (TLS 1.3)<br/>• Disaster recovery (RPO < 15m, RTO < 60m)<br/>• DDoS rate limits (120 req/min/IP) |
| **`03-remote-work-security-and-employee-benefits-guide.pdf`** | Remote Work & Employee Benefits Guide | HR, People & Operations | • $1,500 USD home office equipment stipend<br/>• 36-month MacBook Pro / Dell hardware refresh<br/>• 100% employer-covered health & dental<br/>• 16 weeks fully paid parental leave<br/>• $2,500 annual professional development stipend<br/>• Zero-Trust YubiKey / MFA compliance |

---

## How to Ingest

1. Open your **KnowledgeBridge** workflow in n8n.
2. Select the **Document Ingestion Form Trigger** node.
3. Click **Test step** (or open the Form URL in your browser).
4. Upload any of the PDFs above.
5. Execute the pipeline — the **Enterprise Document Parser & Chunker** will extract the text, slice it into ~900 character chunks with 150 character overlap, attach document metadata (`title`, `source`, `category`, `uploaded_at`), generate **3072-dimensional** Gemini embeddings, and upsert them into Pinecone index `rag-n8n`.

---

## Recommended Verification Queries (Chat Trigger)

After uploading the documents, test the **KnowledgeBridge RAG Agent** via the **Enterprise Chat Trigger**:

```text
1. "What is our policy on approved AI models for production?"
   -> Expected: Cites Groq Llama-3.3, Qwen 2.5, Gemini 1.5, Claude 3.5 from Document 01.

2. "What are our high-availability targets and Pinecone vector configuration?"
   -> Expected: Cites 99.99% uptime SLA, 3072-dimensional Pinecone index, and AES-256 encryption from Document 02.

3. "How much is the home office stipend and what is the parental leave policy?"
   -> Expected: Cites $1,500 home office setup stipend and 16 weeks fully paid parental leave from Document 03.

4. "What is our policy on pet travel expenses?"
   -> Expected Grounding Test: "I could not find relevant information regarding your request in the enterprise knowledge base." (Strict refusal without hallucination).
```
