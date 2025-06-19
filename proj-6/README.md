# Email\_Automation

* **follow-up\_emails.json** – Uses Anthropic Claude, OpenAI Embeddings

---

## Workflow Purpose

Automates the generation and logging of personalized follow‑up emails. When triggered (via webhook), it ingests incoming context (previous conversation or contact list), retrieves relevant history from a vector store, and uses Anthropic Claude to draft an appropriate follow‑up. Finally, it logs the outcome to Google Sheets and notifies on error via Slack.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/follow-up-emails`
   * **Purpose:** Entry point for new follow‑up requests. Expect JSON body containing at least the contact ID or conversation snippet.

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Breaks long conversation histories into manageable chunks for embedding.

2. **Embeddings (OpenAI)**

   * **Model:** `text-embedding-3-small`
   * **Credentials:** OpenAI API key
   * **Role:** Converts each text chunk into a vector representation.

3. **Weaviate Insert**

   * **Vector Store:** Weaviate
   * **Index Name:** `follow-up_emails`
   * **Role:** Stores new embeddings (e.g., past email content or conversation snippets) for later retrieval.

4. **Weaviate Query**

   * **Index Name:** `follow-up_emails`
   * **Role:** Retrieves the most relevant past chunks based on similarity to the current context.

5. **Vector Tool**

   * **Name/Description:** “Weaviate” / “Vector context”
   * **Role:** Exposes the queried vectors as a “tool” to the RAG Agent, so it can include context in its reasoning.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Maintains short‑term conversational context across this single webhook run.

7. **Chat Model (Anthropic Claude)**

   * **Credentials:** Anthropic API key
   * **Role:** The core LLM used for drafting follow‑up emails, guided by retrieved context and system prompt.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an assistant for Follow‑up Emails”
   * **Connected Tools:**

     * **Weaviate Vector Tool** (for context)
     * **Window Memory** (for conversation)
     * **Anthropic Claude** (for generation)
   * **Role:** Coordinates context retrieval + LLM generation to produce the final email text.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append row to the “Log” sheet
   * **Columns:** at minimum captures “Status” (e.g., “Success” + snippet or generated subject)
   * **Credentials:** Google Sheets OAuth2

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Follow-up Emails error: {$json.error.message}`
   * **Credentials:** Slack API token
   * **Role:** Catches any error in the RAG Agent branch and notifies the team.

---

## Credentials Required

* **OPENAI\_API** (for embeddings)
* **ANTHROPIC\_API** (for Claude chat)
* **WEAVIATE\_API** (for vector storage)
* **SHEETS\_API** (Google Sheets OAuth2)
* **SLACK\_API** (for error reporting)

---

## Execution Order

Set to version **v1**, ensuring nodes run in the precise sequence defined by connections.

---

## Usage & Setup

1. **Import Workflow**

   * Load `follow-up_emails.json` into your n8n instance.

2. **Configure Credentials**

   * Create or link credentials for OpenAI, Anthropic, Weaviate, Google Sheets, and Slack.

3. **Deploy & Activate**

   * Enable the workflow.
   * Expose your n8n endpoint (if external triggers are required).

4. **Invoke**

   * Send a POST to `https://<your-n8n>/webhook/follow-up-emails` with the payload:

     ```json
     {
       "contactId": "12345",
       "conversation": "Hi, just checking in about our proposal..."
     }
     ```

5. **Monitor & Iterate**

   * Check Google Sheets for logs.
   * Watch Slack for any error alerts.

---

## Key Benefits

* **Scalable RAG Pipeline:** Embedding + vector store ensures context‑aware drafting.
* **Modular Design:** Easy to swap out LLMs or vector stores.
* **Full Observability:** Logging and alerting layers for both success and failure.
* **Rapid Deployment:** Webhook trigger allows integration into any system or form.
