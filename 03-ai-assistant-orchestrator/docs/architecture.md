# System Architecture & Design Principles

This document provides a comprehensive architectural breakdown of the **AI Assistant Orchestrator** and its companion **Weather Information Tool** sub-workflow.

---

## 1. Architectural Overview

The system is structured as an enterprise-grade agentic workflow that pairs autonomous multi-tool reasoning with deterministic external integrations:

- **Decoupled Architecture:** The primary conversational reasoning engine (LangChain ReAct agent) is completely decoupled from external tool implementations.
- **Sub-Workflow Delegation:** Complex external integrations (like OpenWeatherMap) are encapsulated within dedicated sub-workflows, ensuring that parameter validation, upstream error capture, and data normalization occur independently of the main canvas.
- **Zero Hallucination Grounding:** System directives strictly mandate that factual figures (weather metrics, foreign exchange rates, mathematical totals) must be acquired via tools rather than fabricated by LLM weights.
- **High-Availability Failover:** The orchestrator features a dual-model configuration where a sub-second primary model (`Groq openai/gpt-oss-120b`) is backed by a resilient enterprise fallback model (`Mistral Cloud mistral-medium-3.5`).

---

## 2. End-to-End System Architecture

```mermaid
flowchart TD
    subgraph Intake["1. Conversational Intake"]
        A[Chat Trigger: Webhook Endpoint] --> B[AI Agent: LangChain ReAct Engine]
    end

    subgraph Memory["2. Contextual State & Memory"]
        C[(Simple Memory: 10-Turn Buffer Window)] <-->|Context Retrieval & State Update| B
    end

    subgraph LLM["3. High-Availability Reasoning Layer"]
        D[Groq Chat Model: Primary LPU Inference] -->|Primary Model| B
        E[Mistral Cloud Chat Model: Failover Engine] -.->|Fallback Model| B
    end

    subgraph Tools["4. Multi-Tool Execution Ecosystem"]
        B -->|Foreign Exchange Query| F[Currency Exchange Rates: Frankfurter API]
        B -->|Arithmetic & Multi-Step Math| G[Calculator: Numerical Computation]
        B -->|Reasoning & Planning Scratchpad| H[Think: Chain-of-Thought Scratchpad]
        B -->|Live Web & News Intelligence| I[Search in Tavily: Web Knowledge]
        B -->|Meteorological Query| J[Weather: Tool Workflow Bridge]
    end

    subgraph SubWorkflow["5. Deterministic Weather Sub-Workflow"]
        J -.->|Execute Sub-Workflow| K[Execute Workflow Trigger: Typed Inputs]
        K --> L{"Validate Input: City Not Empty?"}
        L -->|Valid City| M[OpenWeatherMap: Real-Time API]
        L -->|Missing City| N[Missing City Error: Error Contract Set Node]
        M --> O[Normalize Weather Response: Code Node]
        O -.->|Structured Weather Payload| J
        N -.->|Structured Error Payload| J
    end

    classDef intake fill:#e1f5fe,stroke:#0288d1,stroke-width:1px;
    classDef agent fill:#ede7f6,stroke:#512da8,stroke-width:1px;
    classDef memory fill:#e0f2f1,stroke:#00796b,stroke-width:1px;
    classDef llm fill:#fff3e0,stroke:#f57c00,stroke-width:1px;
    classDef tools fill:#e8f5e9,stroke:#388e3c,stroke-width:1px;
    classDef subflow fill:#fce4ec,stroke:#c2185b,stroke-width:1px;

    class A intake;
    class B agent;
    class C memory;
    class D,E llm;
    class F,G,H,I,J tools;
    class K,L,M,N,O subflow;
```

---

## 3. ReAct Agent Execution Loop

The **AI Agent** operates on a **Reasoning + Acting (ReAct)** paradigm:

```mermaid
sequenceDiagram
    autonumber
    actor User as User Chat
    participant Agent as AI Agent (LangChain)
    participant Memory as Simple Memory
    participant LLM as Groq Chat Model
    participant Tool as Specialized Tool

    User->>Agent: Inbound Message
    Agent->>Memory: Load Chat History (Last 10 turns)
    Memory-->>Agent: Contextual Turns
    Agent->>LLM: Formulate Prompt with Available Tools
    LLM-->>Agent: Action Plan (Tool Call: name, args)
    Agent->>Tool: Execute Tool (Input Parameters)
    Tool-->>Agent: Observation (JSON Payload)
    Agent->>LLM: Synthesize Response with Observation
    LLM-->>Agent: Final Direct Markdown Response
    Agent->>Memory: Save User Turn & Agent Reply
    Agent-->>User: Outbound Chat Message
```

---

## 4. Multi-Model Resilience & Failover Strategy

To eliminate single points of failure (SPOF):

1. **Primary Model (Groq - `openai/gpt-oss-120b`):**
   - Engineered for sub-second inference latency (~400–600 ms).
   - Handles multi-turn tool selection and JSON argument extraction.
2. **Fallback Model (Mistral Cloud - `mistral-medium-3.5`):**
   - Wired via `ai_languageModel` index 1 with `needsFallback: true`.
   - Automatically takes over reasoning if Groq encounters:
     - Rate-limit thresholds (HTTP 429).
     - Transient cloud outages (HTTP 500/503).
     - Extended context window saturation.

---

## 5. Conversational Memory Management

- **Node Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- **Window Size:** 10 conversational turns (`contextWindowLength: 10`).
- **Session Isolation:** Keyed automatically by n8n's incoming `sessionId`, ensuring that distinct client sessions maintain independent histories without cross-talk or state leakage.

---

## 6. Sub-Workflow Encapsulation Pattern

Rather than cluttering the primary agent canvas with API formatting nodes, the weather integration is encapsulated in an autonomous sub-workflow:
1. **Contract Independence:** The sub-workflow exposes typed inputs (`city`, `country_code`, `units`) and always returns a consistent JSON schema (`{ success, city, country, temperature, feels_like, humidity, condition, wind_speed, units }`).
2. **Elimination of Intermediate LLMs:** The sub-workflow utilizes deterministic JavaScript (`Code` node) to parse and normalize the weather payload, reducing latency and operational API costs.
3. **Isolated Error Boundaries:** Upstream OpenWeatherMap errors (such as 404 City Not Found) are converted into structured `{ success: false, error: "..." }` responses, preventing orchestrator canvas crashes.
