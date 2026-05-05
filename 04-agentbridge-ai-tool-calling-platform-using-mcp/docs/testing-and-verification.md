# Testing, Verification & Benchmarks

This document details the test matrix, benchmark metrics, and execution proof logs validating the **AgentBridge MCP Platform**.

---

## 1. Verified Live Execution Proofs

### Test Case 1: Mathematical Expression Evaluation (Execution `#272`)
- **Execution Mode:** Manual / Chat Trigger
- **Execution Status:** ✅ `success`
- **Total Latency:** 1.40s
- **Token Usage:** 2,276 total tokens (2,258 prompt, 18 completion)

#### Trace Log:
1. **User Inbound:** `"What is 45 * 12 + 100?"`
2. **Groq Primary Model:** Emits tool call `MCP_Tool_Bridge_Client_Calculator_Tool` with `{"input": "45 * 12 + 100"}`.
3. **MCP Tool Bridge Client:** Transmits JSON-RPC payload to MCP Server endpoint.
4. **Calculator Tool (Server):** Computes arithmetic output `640`.
5. **Agent Final Synthesis:** Returns `"45 × 12 + 100 = **640**"`.

---

### Test Case 2: Multi-Parameter Currency Conversion (Execution `#280`)
- **Execution Mode:** Manual / Chat Trigger
- **Execution Status:** ✅ `success`
- **Total Latency:** 1.46s
- **Token Usage:** 2,340 total tokens (2,310 prompt, 30 completion)

#### Trace Log:
1. **User Inbound:** `"Convert 50 USD to EUR"`
2. **Groq Primary Model:** Emits tool call `MCP_Tool_Bridge_Client_Currency_Exchange_Rates_Tool` with `{"amount": 50, "from_currency": "USD", "to_currency": "EUR"}`.
3. **Currency Exchange Rates Tool (Server):** Queries Frankfurter API endpoint `https://api.frankfurter.dev/v1/latest?amount=50&base=USD&symbols=EUR`.
4. **API Response:** `[{"amount": 50, "base": "USD", "date": "2026-09-24", "rates": {"EUR": 43.987}}]`.
5. **Agent Final Synthesis:** Returns `"50 USD is approximately **43.99 EUR** (rate as of 2026-09-24)."`.

---

### Test Case 3: Interactive Multi-Turn Chat & Live Failover (Demo Session)
- **Execution Source:** Interactive Canvas Chat Demo (`assets/agentbridge-interactive-chat-demo.gif`)
- **Execution Status:** ✅ `success`
- **Memory Retention:** Multi-turn session state preserved across all turns via `Conversation Window Memory`.

#### Turn-by-Turn Trace:
1. **Turn 1 (Direct Answer):**
   - **User:** `"Hello! What operations can you help me perform?"`
   - **Model Decision:** Direct answering protocol invoked. No tool calls emitted.
   - **Output:** Structured overview of supported capabilities (math, currency exchange, timezone/datetime, weather, Wikipedia lookup, Slack alerts, calendar scheduling, data table storage).
2. **Turn 2 (Timezone Resolution via MCP):**
   - **User:** `"What is the current time in Tokyo?"`
   - **Model Decision:** Emits tool call to `Date & Time Utility Tool` with timezone `Asia/Tokyo`.
   - **MCP Tool Output:** Returns current ISO datetime formatted for Japan Standard Time (JST).
   - **Agent Output:** Synthesizes clear, readable current local time for Tokyo.
3. **Turn 3 (Currency Conversion + Live Model Failover):**
   - **User:** `"How much is 120 EUR in GBP?"`
   - **Tool Invoked:** `Currency Exchange Rates Tool` via MCP with `from_currency: "EUR"`, `to_currency: "GBP"`, `amount: 120`.
   - **Primary Model Rate Limit:** Primary model (Groq) hit an input token rate limit (HTTP 429).
   - **Automated Failover:** Orchestrator automatically intercepted the error and routed the context and tool response to **Google Gemini**.
   - **Response Synthesis:** Gemini seamlessly picked up the execution context and synthesized the final converted amount (`120 EUR in GBP`) directly to the user without breaking conversational flow or throwing a workflow error.

---

## 2. High-Availability Dual-Model Failover Verification

During burst load testing, the Groq on-demand tier encountered a standard rate limit of 7,000 input tokens per minute (`ITPM`). The orchestrator demonstrated the following resilience behavior:

1. **Detection:** When Groq returns HTTP `429 (rate_limit_exceeded)`, the LangChain agent intercepts the error without crashing the workflow.
2. **Failover Execution:** If configured with fallback enabled (`needsFallback: true`), the agent immediately forwards the conversation history and pending tool call payload to `Gemini Fallback Model (Fallback Model)`.
3. **Session Continuity:** The user's active session state in `Conversation Window Memory` remains unbroken.

---

## 3. Comprehensive Manual Verification Matrix

Use these test prompts in the n8n Chat canvas to verify each tool:

| Capability | Test Input Prompt | Expected Tool Invoked | Success Criteria |
| :--- | :--- | :--- | :--- |
| **Arithmetic** | `"What is 15% of 850 plus 42?"` | `Calculator Tool` | Exact numerical calculation (`169.5`). |
| **Forex** | `"Convert 150 GBP to JPY"` | `Currency Exchange Rates Tool` | Live exchange rate and converted total. |
| **Date & Time** | `"What time is it right now in Tokyo?"` | `Date & Time Utility Tool` | Correct ISO datetime formatted in JST timezone. |
| **Weather** | `"What's the weather in Paris, France right now?"` | `Weather Service Tool` | Current temperature, condition, and humidity. |
| **Wikipedia** | `"Tell me briefly about Alan Turing's contribution to computing."` | `Wikipedia Knowledge Tool` | Concise encyclopedic summary with factual grounding. |
| **Slack** | `"Send a message to Slack saying 'Deploy pipeline completed successfully'"` | `Slack Messenger Tool` | Confirmed message post to `#ai-support-automation`. |
| **Calendar** | `"Schedule an appointment titled 'Project Sync' for tomorrow at 2 PM to 3 PM"` | `Google Calendar Scheduler Tool` | Event created on Google Calendar with title and timestamps. |
| **Direct Answer** | `"Can you explain the difference between REST and GraphQL?"` | *None (Direct Answer)* | Answers conversationally without invoking any MCP tool. |
