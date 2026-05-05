# MCP Tools Specification & Parameter Reference

This document provides the formal contract specification for all 8 tools exposed by the **MCP Server** (`Workflow 1`) and consumed by the **AgentBridge Client** (`Workflow 2`).

---

## 1. Calculator Tool

- **Node Type:** `@n8n/n8n-nodes-langchain.toolCalculator` (v1)
- **Node Name:** `Calculator Tool`
- **Purpose:** High-precision mathematical calculations and arithmetic expression evaluation.
- **Description Exposing to LLM:** 
  > *"Perform mathematical calculations and evaluate arithmetic expressions. Supports basic arithmetic (+, -, *, /), exponents (^), modulo (%), and functions like sqrt(), sin(), cos(), log(), or percentage calculations (e.g., '15% of 250', '(14 * 12) / 3'). Always pass a clean mathematical expression as input."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `input` | `string` | **Yes** | — | Clean mathematical expression to evaluate (e.g., `"45 * 12 + 100"`, `"sqrt(144) + 10"`). |

### Output Example
```json
[
  {
    "type": "text",
    "text": "640"
  }
]
```

---

## 2. Wikipedia Knowledge Tool

- **Node Type:** `@n8n/n8n-nodes-langchain.toolWikipedia` (v1)
- **Node Name:** `Wikipedia Knowledge Tool`
- **Purpose:** Encyclopedic knowledge retrieval, historical entity background, and scientific concepts.
- **Description Exposing to LLM:** 
  > *"Search Wikipedia for factual background information, historical events, scientific concepts, and entity biographies. Input must be a specific topic or entity query (e.g., 'Quantum computing', 'Alan Turing'). Returns concise contextual summaries."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `input` | `string` | **Yes** | — | Concise query string or entity name (e.g., `"Model Context Protocol"`, `"Nikola Tesla"`). |

---

## 3. Date & Time Utility Tool

- **Node Type:** `n8n-nodes-base.dateTimeTool` (v2)
- **Node Name:** `Date & Time Utility Tool`
- **Operation:** `getCurrentDate`
- **Purpose:** Resolution of relative dates ("today", "next Monday") and timezone-aware timestamps.
- **Description Exposing to LLM:** 
  > *"Retrieve the current date, time, and day of the week in a specified timezone. Useful for resolving relative dates like 'today', 'tomorrow', or 'next Monday'."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `timezone` | `string` | No | `"UTC"` | IANA timezone string (e.g., `"America/New_York"`, `"Europe/London"`, `"Asia/Tokyo"`). |

### Output Contract
```json
[
  {
    "currentDate": "2026-09-25T00:04:22.000Z"
  }
]
```

---

## 4. Weather Service Tool

- **Node Type:** `n8n-nodes-base.openWeatherMapTool` (v1)
- **Node Name:** `Weather Service Tool`
- **Operation:** `currentWeather`
- **Purpose:** Real-time global meteorological telemetry.
- **Description Exposing to LLM:** 
  > *"Retrieve real-time weather conditions (temperature, weather summary, humidity, wind speed) for any city worldwide. Specify the city name and an optional 2-letter country code."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `city` | `string` | **Yes** | — | Name of the city to query (e.g., `"London"`, `"Berlin"`, `"San Francisco"`). |
| `country_code` | `string` | No | `""` | Two-letter ISO country code for disambiguation (e.g., `"GB"`, `"DE"`, `"US"`). |
| `language` | `string` | No | `"en"` | Two-letter language code for weather descriptions (e.g., `"en"`, `"de"`, `"es"`). |

---

## 5. Currency Exchange Rates Tool

- **Node Type:** `n8n-nodes-base.httpRequestTool` (v4.5)
- **Node Name:** `Currency Exchange Rates Tool`
- **Method:** `GET`
- **Endpoint:** `https://api.frankfurter.dev/v1/latest?amount={{amount}}&base={{from_currency}}&symbols={{to_currency}}`
- **Purpose:** Real-time foreign currency conversions and spot exchange rates.
- **Description Exposing to LLM:** 
  > *"Convert an amount from one fiat currency to another using real-time foreign exchange rates via Frankfurter. Requires from_currency (3-letter code), to_currency (3-letter code), and optional amount (defaults to 1)."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `from_currency` | `string` | **Yes** | — | 3-letter ISO currency code to convert from (e.g., `"USD"`, `"EUR"`, `"GBP"`). |
| `to_currency` | `string` | **Yes** | — | 3-letter ISO currency code to convert to (e.g., `"EUR"`, `"USD"`, `"JPY"`). |
| `amount` | `number` | No | `1` | Numeric quantity of the base currency to convert. Defaults to `1`. |

### Output Example
```json
[
  {
    "amount": 50,
    "base": "USD",
    "date": "2026-09-24",
    "rates": {
      "EUR": 43.987
    }
  }
]
```

---

## 6. Slack Messenger Tool

- **Node Type:** `n8n-nodes-base.slackTool` (v2.7)
- **Node Name:** `Slack Messenger Tool`
- **Resource / Operation:** `message` / `post`
- **Purpose:** Automated team notifications and alerts directly into designated Slack channels.
- **Description Exposing to LLM:** 
  > *"Post a notification or message to a designated Slack channel. Supports standard Slack markdown formatting."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `channel` | `string` | No | `"C0C2D0HHQ8L"` | Target channel ID or name (defaults to `#ai-support-automation`). |
| `message` | `string` | **Yes** | — | Formatted text content supporting Slack markdown (bold, links, code blocks). |

---

## 7. Google Calendar Scheduler Tool

- **Node Type:** `n8n-nodes-base.googleCalendarTool` (v1.3)
- **Node Name:** `Google Calendar Scheduler Tool`
- **Resource / Operation:** `event` / `create`
- **Purpose:** Autonomous appointment scheduling and calendar event creation.
- **Description Exposing to LLM:** 
  > *"Create a new event in Google Calendar with title, start time, end time, optional description, and timezone."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `title` | `string` | **Yes** | — | Event headline / summary (e.g., `"Q3 Financial Review"`). |
| `start` | `string` | **Yes** | — | Start time in ISO 8601 format (e.g., `"2026-10-01T10:00:00Z"`). |
| `end` | `string` | **Yes** | — | End time in ISO 8601 format (e.g., `"2026-10-01T11:00:00Z"`). |
| `description` | `string` | No | `""` | Detailed agenda, meeting links, or notes for the calendar event. |
| `timezone` | `string` | No | `"UTC"` | Optional IANA timezone identifier. |
| `calendar_id` | `string` | No | `"primary"` | Target calendar ID (defaults to primary user calendar). |

---

## 8. Data Table Persistence Tool

- **Node Type:** `n8n-nodes-base.dataTableTool` (v1.1)
- **Node Name:** `Data Table Persistence Tool`
- **Resource / Operation:** `row` / `insert`
- **Purpose:** Structured data recording and audit logging into n8n internal database tables.
- **Description Exposing to LLM:** 
  > *"Insert a structured record into an n8n Data Table with title, category, data, and metadata fields."*

### Parameter Schema
| Parameter | Type | Required | Default | Description & Example |
| :--- | :--- | :--- | :--- | :--- |
| `table_id` | `string` | **Yes** | — | Target n8n Data Table ID or name. |
| `title` | `string` | No | `""` | Record title or entry headline. |
| `category` | `string` | No | `""` | Classification category (e.g., `"support_ticket"`, `"audit_log"`, `"lead"`). |
| `data` | `string` | No | `""` | Primary payload body, serialized JSON, or text content. |
| `metadata` | `string` | No | `""` | Supplemental key/value context or execution metadata. |
