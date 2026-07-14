# Slack App Manifest, Event Routing & Security Architecture

This document details the Slack application configuration, OAuth permission scopes, event subscription lifecycle, and security guardrails governing the **SlackOps Conversational RAG Copilot**.

---

## 1. Production Slack App Manifest (JSON)

You can copy and paste this manifest directly into the [Slack App Management Portal](https://api.slack.com/apps) (**Create New App → From an app manifest**):

```json
{
  "display_information": {
    "name": "SlackOps Conversational RAG Copilot",
    "description": "Enterprise autonomous knowledge retrieval and in-channel document indexing assistant.",
    "background_color": "#1A365D"
  },
  "features": {
    "bot_user": {
      "display_name": "SlackOps Copilot",
      "always_online": true
    }
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "app_mentions:read",
        "channels:history",
        "chat:write",
        "chat:write.customize",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "event_subscriptions": {
      "request_url": "https://n8n.ravirai.dev/webhook/ad621192-6932-4cda-a446-c8cdbeb5abed/webhook",
      "bot_events": [
        "app_mention",
        "file_share"
      ]
    },
    "org_deploy_enabled": false,
    "socket_mode_enabled": false,
    "token_rotation_enabled": false
  }
}
```

---

## 2. OAuth Scope Justifications

Every permission granted to the bot adheres to the principle of least privilege:

| Scope | Permission Level | Operational Justification |
| :--- | :---: | :--- |
| **`app_mentions:read`** | Read | Allows the bot to intercept incoming user questions when explicitly tagged (e.g. `@n8n what is our SLA?`). |
| **`channels:history`** | Read | Enables the bot to inspect preceding message context in public channels to resolve references like *"explain step 2"*. |
| **`chat:write`** | Write | Grants permission to post markdown answers, bullet points, and source citations back into the channel. |
| **`chat:write.customize`** | Write | Permits setting custom profile icons and status badges dynamically. |
| **`files:read`** | Read | Enables accessing uploaded PDF, TXT, and DOCX metadata (`url_private_download`) for document vectorization. |
| **`files:write`** | Write | Allows the bot to post file confirmation badges and snippet attachments into the conversation thread. |
