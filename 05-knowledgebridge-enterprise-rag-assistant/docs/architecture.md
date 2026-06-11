# System Architecture: KnowledgeBridge Enterprise RAG Assistant

## 1. Architectural Philosophy & Design Principles

**KnowledgeBridge** is designed around a clean separation of concerns between **Continuous Knowledge Ingestion** and **Autonomous Context-Grounded Reasoning**. In enterprise environments, document lifecycles are asynchronous and decoupled from real-time user query traffic.

To ensure production resilience, scalability, and strict compliance, KnowledgeBridge adheres to four architectural pillars:

```mermaid
flowchart TD
    subgraph Ingestion_Flow ["Phase 1: Ingestion Pipeline"]
        F["Document Ingestion Form Trigger<br/><i>(Multipart File Intake)</i>"] --> L["Enterprise Document Parser & Chunker<br/><i>(Recursive Text Splitter: 900c / 150o)</i>"]
        L --> P1["Pinecone Vector Ingestion Store<br/><i>(Batch Upsert: 200 items / index: rag-n8n)</i>"]
        E1["Gemini Embedding Generator (Ingestion)<br/><i>(Dense 3072-dim Vector Space)</i>"] -.->|ai_embedding| P1
    end

    subgraph Retrieval_Flow ["Phase 2: Conversational Retrieval & Reasoning Agent"]
        C["Enterprise Chat Trigger<br/><i>(User Dialogue Stream)</i>"] --> A["KnowledgeBridge RAG Agent<br/><i>(LangChain ReAct Agent v3.1)</i>"]
        M["Groq LLM Engine (Qwen 27B)<br/><i>(High-Throughput Reasoning)</i>"] -.->|ai_languageModel| A
        W["Conversation Window Buffer Memory<br/><i>(10-Turn Rolling Context)</i>"] -.->|ai_memory| A
        T["Enterprise Knowledge Base Retriever<br/><i>(Top-5 Chunk Vector Tool)</i>"] -.->|ai_tool| A
        E2["Gemini Embedding Generator (Retrieval)<br/><i>(Dense 3072-dim Vector Space)</i>"] -.->|ai_embedding| T
    end

    classDef ing fill:#e8f5e9,stroke:#2e7d32,stroke-width:1.5px;
    classDef ret fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px;
    class F,L,P1,E1 ing;
    class C,A,M,W,T,E2 ret;
```

---

## 2. Ingestion Pipeline Mechanics

The ingestion pipeline transforms raw unstructured documents (PDFs, TXT, DOCX, Markdown) into dense mathematical vectors enriched with organizational metadata.

### Step 1: Document Intake (`Document Ingestion Form Trigger`)
- **Node Type:** `n8n-nodes-base.formTrigger`
- **Payload:** Accepts binary file uploads (`Upload File`) through a web form interface or automated curl/API POST endpoints.
- **Contract:** Passes binary stream metadata (`$binary.Upload_File`) containing `fileName`, `fileSize`, and `mimeType`.

### Step 2: Parsing & Semantic Chunking (`Enterprise Document Parser & Chunker`)
- **Node Type:** `@n8n/n8n-nodes-langchain.documentDefaultDataLoader`
- **Chunk Geometry:**
  - **Chunk Size:** ~900 characters (~180-220 tokens). This size provides sufficient semantic context for a single policy clause or technical specification without diluting similarity search vectors.
  - **Chunk Overlap:** ~150 characters (~30-40 tokens). Prevents semantic boundary truncation by repeating trailing sentences across adjacent chunks.
- **Metadata Preservation:**
  ```javascript
  {
    "title": "={{ $binary.Upload_File.fileName || 'Untitled Document' }}",
    "source": "={{ $binary.Upload_File.fileName || 'Direct Upload' }}",
    "category": "enterprise_docs",
    "uploaded_at": "={{ $now.toISO() }}"
  }
  ```

### Step 3: High-Dimensional Dense Embedding (`Gemini Embedding Generator`)
- **Node Type:** `@n8n/n8n-nodes-langchain.embeddingsGoogleGemini`
- **Embedding Model:** `models/gemini-embedding-001`
- **Output Dimensionality:** **3072 dimensions**.
- **Vector Space Consistency:** Guarantees that vectors generated during ingestion reside in the exact mathematical subspace as queries formulated during conversational retrieval.

### Step 4: Vector Indexing (`Pinecone Vector Ingestion Store`)
- **Node Type:** `@n8n/n8n-nodes-langchain.vectorStorePinecone` (Mode: `insert`)
- **Target Index:** `rag-n8n`
- **Batch Processing:** Processes up to 200 chunks per transaction (`embeddingBatchSize: 200`) to respect API throughput limits while maximizing ingestion speed.

---

## 3. Conversational RAG Query Agent

The retrieval agent operates as an autonomous ReAct reasoning loop bounded by strict grounding directives.

```mermaid
sequenceDiagram
    autonumber
    actor User as Enterprise User
    participant Chat as Enterprise Chat Trigger
    participant Agent as KnowledgeBridge RAG Agent
    participant Memory as Window Buffer Memory (10 Turns)
    participant LLM as Groq Qwen 27B
    participant Tool as Knowledge Base Retriever
    participant Pinecone as Pinecone Vector Store (3072-dim)

    User->>Chat: Asks inquiry ("What is the parental leave policy?")
    Chat->>Agent: Dispatches prompt & session state
    Agent->>Memory: Reads recent 10 dialogue turns
    Memory-->>Agent: Returns conversation context
    Agent->>LLM: Evaluates intent with system prompt
    LLM-->>Agent: Issues tool call: Enterprise Knowledge Base Retriever(query)
    Agent->>Tool: Executes retrieval tool
    Tool->>Pinecone: Semantic cosine query (Top 5 chunks)
    Pinecone-->>Tool: Returns 5 chunks + metadata (title, source, uploaded_at)
    Tool-->>Agent: Delivers contextual excerpts
    Agent->>LLM: Evaluates grounded synthesis & citation requirement
    LLM-->>Agent: Generates concise factual answer citing [Source: title]
    Agent->>Memory: Records user prompt & model answer
    Agent-->>User: Streams/returns verified grounded response
```

---

## 4. Anti-Hallucination & Grounding Guardrails

Enterprise RAG applications require deterministic boundaries against fabricated facts. KnowledgeBridge enforces this through five discrete layers:

1. **System Prompt Bounding:** The agent is instructed to consider *only* retrieved passages as ground truth. Speculative completion is explicitly prohibited.
2. **Missing Context Fallback:** If retrieval similarity scores fall below threshold or return empty sets, the agent must output:
   > *"I could not find relevant information regarding your request in the enterprise knowledge base."*
3. **Mandatory Metadata Citations:** Whenever statements are made from retrieved text, the agent must cite the source document title using the format `[Source: <title>]`.
4. **Context Window Isolation:** A 10-turn memory buffer prevents conversation drift and hallucination compounding across extended sessions.
5. **Observability Tracing:** Every execution automatically captures structured tracing metadata (`question`, `model`, `timestamp`, `workflow`) for downstream auditing and evaluation.
