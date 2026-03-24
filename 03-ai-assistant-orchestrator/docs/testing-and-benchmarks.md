# Testing, Verification & Performance Benchmarks

This document records the empirical testing methodology, execution traces, and performance benchmarks for the **AI Assistant Orchestrator** and **Weather Information Tool**.

---

## 1. Testing Methodology

In compliance with repository standards (`AGENTS.md: Testing & Verification Standards`):
- **Atomic Sub-Workflow Testing:** The Weather sub-workflow was validated via `prepare_workflow_pin_data` and `test_workflow` with simulated inputs prior to live canvas testing.
- **End-to-End Orchestrator Testing:** Multi-turn live chat sessions were tested via `execute_workflow` in manual mode to inspect full token metrics and tool execution traces.

---

## 2. Test Execution Traces

### Scenario 1: Direct Currency Exchange Lookup
- **Inbound Query:** *"What is the exchange rate from USD to EUR right now?"*
- **Execution ID:** `#260`
- **Tool Invoked:** `Currency Exchange Rates`
- **Tool Input:** `{"from_currency": "USD", "to_currency": "EUR"}`
- **Tool Output:**
  ```json
  {
    "date": "2026-09-21",
    "base": "USD",
    "quote": "EUR",
    "rate": 0.87069
  }
  ```
- **Agent Final Output:**
  > *"The current exchange rate is 1 USD ≈ 0.87069 EUR."*
- **Latency:** **1.26 seconds** (Tavily search was completely bypassed).

---

### Scenario 2: Chained Conversion & Arithmetic
- **Inbound Query:** *"Convert 250 EUR to USD"*
- **Execution ID:** `#261`
- **Step 1 (Tool Call):** `Currency Exchange Rates`
  - Input: `{"from_currency": "EUR", "to_currency": "USD"}`
  - Output: `{"date": "2026-09-21", "base": "EUR", "quote": "USD", "rate": 1.1485}`
- **Step 2 (Tool Call):** `Calculator`
  - Input: `{"input": "250*1.1485"}`
  - Output: `{"response": "287.125"}`
- **Agent Final Output:**
  > *"250 EUR is equivalent to ≈ 287.13 USD (using the rate 1 EUR = 1.1485 USD)."*
- **Latency:** **1.36 seconds** across 2 sequential tool invocations.

---

### Scenario 3: Real-Time Meteorological Retrieval
- **Inbound Query:** *"What is the current temperature, wind speed, and humidity in Paderborn?"*
- **Tool Invoked:** `Weather` (`toolWorkflow`)
- **Sub-Workflow Trace:**
  1. `When Executed by Another Workflow` passed `{ city: "Paderborn", units: "metric" }`.
  2. `Validate Input` evaluated `notEmpty` ➔ `true`.
  3. `OpenWeatherMap` queried Paderborn.
  4. `Normalize Weather Response` emitted:
     ```json
     {
       "success": true,
       "city": "Paderborn",
       "country": "DE",
       "temperature": 15.32,
       "feels_like": 14.71,
       "humidity": 69,
       "condition": "few clouds",
       "wind_speed": 2.57,
       "units": "metric"
     }
     ```
- **Agent Final Output:**
  > *"In Paderborn, Germany, the current weather is:*  
  > *- **Temperature:** 15.3 °C (feels like 14.7 °C)*  
  > *- **Wind Speed:** 2.6 m/s*  
  > *- **Humidity:** 69%*  
  > *- **Condition:** Few clouds"*

---

### Scenario 4: General Web Intelligence Search
- **Inbound Query:** *"Who won the 2026 FIFA World Cup, and what was the final score?"*
- **Tool Invoked:** `Search in Tavily`
- **Tool Input:** `{"query": "2026 FIFA World Cup winner final score"}`
- **Observation:** Real-time web retrieval fetched tournament results and match summaries.
- **Routing Verification:** Weather and Currency tools remained untouched.

---

### Scenario 5: Sub-Workflow Fault Tolerance
- **Case A: Missing City Parameter (Execution #247)**
  - Input: `{ city: "", country_code: "", units: "metric" }`
  - Route: Diverted immediately to `Missing City Error`.
  - Output:
    ```json
    {
      "success": false,
      "error": "City name is required and cannot be empty."
    }
    ```
- **Case B: API 404 City Not Found (Execution #248)**
  - Input: `{ city: "AtlantisUnknownCity" }`
  - Route: `OpenWeatherMap` (with `continueRegularOutput`) ➔ `Normalize Weather Response`.
  - Output:
    ```json
    {
      "success": false,
      "error": "city not found"
    }
    ```

---

## 3. Performance & Token Benchmarks

| Metric | Direct Tool Execution | Chained Execution (2 Tools) | Sub-Workflow Delegation |
| :--- | :---: | :---: | :---: |
| **Total Latency** | 1.26 s | 1.36 s | 4.30 s |
| **LLM Reasoning Turn 1** | ~500 ms | ~400 ms | ~540 ms |
| **Tool Execution** | ~80 ms | ~75 ms + ~2 ms | ~1,160 ms (Remote API) |
| **LLM Synthesis Turn 2** | ~480 ms | ~340 ms | ~2,600 ms |
| **Prompt Tokens** | 601 | 595 | 436 |
| **Completion Tokens** | 42 | 63 | 89 |
| **Total Tokens** | 735 | 818 | 633 |
