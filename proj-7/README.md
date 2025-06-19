
# 📅 My Project 7 – AI-Powered Google Calendar Assistant

This n8n workflow integrates an AI agent with Google Gemini, Google Calendar, and memory capabilities to act on a **schedule trigger** or **chat-based input**. It can be used for smart daily assistant tasks like summarizing schedules, responding to user queries, and managing calendar events.

---

## 🔧 Workflow Overview

### 📌 Trigger Nodes

* **Schedule Trigger**: Fires daily at **5 AM**.
* **When Chat Message Received**: Triggers the workflow when a new chat message is received through the connected chat interface.

### 🧠 Core AI Logic

* **AI Agent (@n8n/n8n-nodes-langchain.agent)**:

  * Acts as the central logic unit for the workflow.
  * Connected with:

    * **Google Gemini Chat Model** (`models/gemini-1.5-flash`)
    * **Memory Buffer Window** for maintaining short-term memory
    * **Google Calendar Tool** for event-based actions and responses

### 🤖 AI Tools

* **Google Gemini Chat Model**:

  * Powers natural language understanding and generation.
  * Requires `Google Gemini (PaLM)` API credentials.

* **Simple Memory**:

  * Maintains conversational context using memory buffer (windowed context).

* **Google Calendar Tool**:

  * Connects to the user's Google Calendar to list events and potentially perform calendar actions.
  * Requires OAuth2 authentication.

---

## 🔄 Workflow Connections

```plaintext
[SCHEDULE TRIGGER] OR [CHAT MESSAGE TRIGGER]
         ↓
       [AI AGENT]
        ↙   ↓   ↘
[GEMINI] [MEMORY] [GOOGLE CALENDAR]
```

* The AI Agent utilizes:

  * **Google Gemini Chat Model** → for intelligent responses.
  * **Memory Buffer** → to retain conversational context.
  * **Google Calendar** → to fetch or interact with events.

---

## 🔐 Credentials Used

* **Google Gemini API**: For language model capabilities.
* **Google Calendar OAuth2**: For accessing calendar data.

---

## 🛠️ Setup Instructions

1. Import the workflow JSON into your n8n instance.
2. Connect and configure the following credentials:

   * `Google Gemini (PaLM) API`
   * `Google Calendar OAuth2 API`
3. Activate the workflow.
4. Trigger via:

   * Daily at 5 AM
   * Webhook: `When chat message received`

---

## 💡 Example Use Cases

* Daily schedule summary at 5 AM
* Chat-based queries like "What’s on my calendar today?"
* AI assistant integration in chat interfaces
* Smart reminder generation and event logging

---

## 📎 Notes

* Ensure your n8n instance is accessible for incoming webhook events if used externally.
* You can customize the AI Agent's prompt handling or memory size in the respective nodes.
* Extend functionality by adding more AI tools (e.g., email, Notion, Google Sheets).

---

## 📬 Webhook Info

* **Webhook ID**: `c53df3e9-7a1e-4950-9b05-0903de53c6b5`
* Exposed endpoint will be similar to:
  `https://<your-n8n-instance>/webhook/c53df3e9-7a1e-4950-9b05-0903de53c6b5`

---

## 📁 File Info

* **Workflow ID**: `3rvvuWPVfojQZ6HX`
* **Version**: 1.2+
* **Execution Order**: `v1`


