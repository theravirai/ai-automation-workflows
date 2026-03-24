# Node Specifications & Parameter Reference

This document provides a detailed reference for every node in the **AI Assistant Orchestrator** and the **Weather Information Tool** sub-workflow, including type versions, parameters, dynamic expressions, and data contracts.

---

## 1. AI Assistant Orchestrator Canvas

### 1.1 `When chat message received`
- **Node Type:** `@n8n/n8n-nodes-langchain.chatTrigger` (v1.4)
- **Role:** Webhook trigger for inbound conversational chat messages.
- **Parameters:**
  ```json
  {
    "options": {}
  }
  ```
- **Output Payload:**
  - `$json.chatInput`: Raw text entered by the user.
  - `$json.sessionId`: Unique identifier for the conversation session.

---

### 1.2 `AI Agent`
- **Node Type:** `@n8n/n8n-nodes-langchain.agent` (v3.1)
- **Role:** Central reasoning and execution agent.
- **Parameters:**
  ```json
  {
    "promptType": "define",
    "text": "={{ $json.chatInput }}",
    "needsFallback": true,
    "options": {
      "systemMessage": "You are a versatile, reliable, and concise AI assistant.\n\nTool Routing & Usage Rules:\n1. Currency Exchange Rates: ALWAYS use this tool for currency conversion and exchange rate queries (e.g. USD to EUR, GBP to JPY). Pass 3-letter currency codes. Never use web search for exchange rates.\n2. Calculator: Use for arithmetic, numerical calculations, and multiplying exchange rates by amounts.\n3. Weather: ALWAYS use this tool for weather lookups in any location. Never use web search for weather.\n4. Search in Tavily: Use for current events, news, and general web information when no specialized tool exists.\n5. Think: Use for complex reasoning or multi-step planning before taking action.\n\nGeneral Directives:\n- Ground all factual answers strictly in tool outputs; never invent or extrapolate data.\n- If a tool fails or information is unavailable, explain clearly and concisely.\n- Keep responses direct, helpful, and concise."
    }
  }
  ```

---

### 1.3 `Groq Chat Model` (Primary LLM)
- **Node Type:** `@n8n/n8n-nodes-langchain.lmChatGroq` (v1.0)
- **Connection:** `ai_languageModel` (Index 0)
- **Parameters:**
  ```json
  {
    "model": "openai/gpt-oss-120b",
    "options": {}
  }
  ```
- **Credentials:** `groqApi` (Groq account)

---

### 1.4 `Mistral Cloud Chat Model` (Fallback LLM)
- **Node Type:** `@n8n/n8n-nodes-langchain.lmChatMistralCloud` (v1.0)
- **Connection:** `ai_languageModel` (Index 1)
- **Parameters:**
  ```json
  {
    "model": "mistral-medium-3.5",
    "options": {}
  }
  ```
- **Credentials:** `mistralCloudApi` (Mistral Cloud account)

---

### 1.5 `Simple Memory`
- **Node Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow` (v1.4)
- **Connection:** `ai_memory` (Index 0)
- **Parameters:**
  ```json
  {
    "contextWindowLength": 10
  }
  ```

---

### 1.6 `Currency Exchange Rates`
- **Node Type:** `n8n-nodes-base.httpRequestTool` (v4.5)
- **Connection:** `ai_tool`
- **Parameters:**
  ```json
  {
    "description": "Get current currency exchange rates and convert between currencies (e.g. USD, EUR, GBP, JPY). Always use this tool for exchange rate queries and currency conversions.",
    "url": "={{ `https://api.frankfurter.dev/v2/rate/${$fromAI('from_currency', 'Three-letter currency code to convert from, e.g. USD, EUR, GBP', 'string').toUpperCase().trim()}/${$fromAI('to_currency', 'Three-letter currency code to convert to, e.g. EUR, USD, JPY', 'string').toUpperCase().trim()}` }}",
    "options": {}
  }
  ```

---

### 1.7 `Weather`
- **Node Type:** `@n8n/n8n-nodes-langchain.toolWorkflow` (v2.2)
- **Connection:** `ai_tool`
- **Parameters:**
  ```json
  {
    "description": "Get current weather information for a specific city or location.",
    "workflowId": {
      "__rl": true,
      "value": "6fKOAXSrJoRHtx50",
      "mode": "list",
      "cachedResultName": "Weather Information Tool",
      "cachedResultUrl": "/workflow/6fKOAXSrJoRHtx50"
    },
    "workflowInputs": {
      "mappingMode": "defineBelow",
      "value": {
        "city": "={{ $fromAI('city', 'The name of the city, e.g. London or Tokyo', 'string') }}",
        "country_code": "={{ $fromAI('country_code', 'Optional two-letter ISO country code, e.g. US, GB, FR', 'string') }}",
        "units": "={{ $fromAI('units', 'Units of measurement: metric or imperial (default: metric)', 'string') }}"
      }
    }
  }
  ```

---

### 1.8 `Calculator`
- **Node Type:** `@n8n/n8n-nodes-langchain.toolCalculator` (v1.0)
- **Connection:** `ai_tool`
- **Parameters:**
  ```json
  {
    "description": "Perform arithmetic and numerical calculations."
  }
  ```

---

### 1.9 `Think`
- **Node Type:** `@n8n/n8n-nodes-langchain.toolThink` (v1.1)
- **Connection:** `ai_tool`
- **Parameters:**
  ```json
  {
    "description": "Use for complex reasoning, multi-step planning, or problem breakdown before acting."
  }
  ```

---

### 1.10 `Search in Tavily`
- **Node Type:** `@tavily/n8n-nodes-tavily.tavilyTool` (v1.0)
- **Connection:** `ai_tool`
- **Parameters:**
  ```json
  {
    "resource": "search",
    "operation": "query",
    "descriptionType": "manual",
    "toolDescription": "Search the web for current events, news, or general web information. Do NOT use for weather or currency exchange rates, as dedicated tools exist for those.",
    "query": "={{ $fromAI('Query', ``, 'string') }}",
    "options": {}
  }
  ```
- **Credentials:** `tavilyApi` (Tavily account)

---

## 2. Weather Information Tool Sub-Workflow

### 2.1 `When Executed by Another Workflow`
- **Node Type:** `n8n-nodes-base.executeWorkflowTrigger` (v1.2)
- **Parameters:**
  ```json
  {
    "inputSource": "workflowInputs",
    "workflowInputs": {
      "values": [
        { "name": "city" },
        { "name": "country_code" },
        { "name": "units" }
      ]
    }
  }
  ```

---

### 2.2 `Validate Input`
- **Node Type:** `n8n-nodes-base.if` (v2.2)
- **Parameters:**
  ```json
  {
    "conditions": {
      "combinator": "and",
      "conditions": [
        {
          "id": "city-not-empty",
          "leftValue": "={{ $json.city }}",
          "operator": {
            "type": "string",
            "operation": "notEmpty"
          },
          "rightValue": ""
        }
      ],
      "options": {
        "caseSensitive": true,
        "leftValue": "",
        "typeValidation": "strict",
        "version": 2
      }
    }
  }
  ```

---

### 2.3 `OpenWeatherMap`
- **Node Type:** `n8n-nodes-base.openWeatherMap` (v1.0)
- **Parameters:**
  ```json
  {
    "operation": "currentWeather",
    "locationSelection": "cityName",
    "cityName": "={{ $json.country_code ? `${$json.city.trim()},${$json.country_code.trim()}` : $json.city.trim() }}",
    "format": "={{ $json.units === 'imperial' ? 'imperial' : 'metric' }}"
  }
  ```
- **Node Settings:** `onError: "continueRegularOutput"`
- **Credentials:** `openWeatherMapApi` (OpenWeatherMap account)

---

### 2.4 `Missing City Error`
- **Node Type:** `n8n-nodes-base.set` (v3.5)
- **Parameters:**
  ```json
  {
    "assignments": {
      "assignments": [
        {
          "id": "err-success",
          "name": "success",
          "type": "boolean",
          "value": false
        },
        {
          "id": "err-msg",
          "name": "error",
          "type": "string",
          "value": "City name is required and cannot be empty."
        }
      ]
    }
  }
  ```

---

### 2.5 `Normalize Weather Response`
- **Node Type:** `n8n-nodes-base.code` (v2.0)
- **Language:** JavaScript
- **Script:**
  ```javascript
  const item = $input.first().json;
  const triggerData = $('When Executed by Another Workflow').first().json;
  const units = triggerData.units === 'imperial' ? 'imperial' : 'metric';

  if (item.error || item.message || !item.main) {
    return [{
      json: {
        success: false,
        error: item.message || item.error || 'Failed to retrieve weather data for the specified location.'
      }
    }];
  }

  return [{
    json: {
      success: true,
      city: item.name,
      country: item.sys?.country || null,
      temperature: item.main.temp,
      feels_like: item.main.feels_like,
      humidity: item.main.humidity,
      condition: item.weather?.[0]?.description || item.weather?.[0]?.main || 'Unknown',
      wind_speed: item.wind?.speed ?? 0,
      units: units
    }
  }];
  ```
