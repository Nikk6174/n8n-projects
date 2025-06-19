# Email\_Automation

* **forward\_attachments.json** – Uses OpenAI Chat, OpenAI Embeddings, Pinecone

---

## Workflow Purpose

Automatically processes incoming email attachments: it ingests file contents, indexes them in Pinecone for RAG retrieval, and drafts context‑aware summaries or forwards via OpenAI Chat.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/forward-attachments`
   * **Purpose:** Activated when an email arrives with attachments (configured via an external mail‑to‑webhook service).

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Chops attachment text (extracted via OCR or text‑extraction pre–workflow) into manageable chunks for embedding.

2. **Embeddings (OpenAI)**

   * **Model:** `text-embedding-3-small`
   * **Credentials:** OpenAI API key
   * **Role:** Transforms each chunk into vector embeddings for semantic indexing.

3. **Pinecone Insert**

   * **Vector Store:** Pinecone
   * **Index Name:** `forward_attachments`
   * **Role:** Inserts new embeddings for attachment content into the Pinecone index.

4. **Pinecone Query**

   * **Index Name:** `forward_attachments`
   * **Role:** Retrieves the most relevant content chunks when composing the summary or forward message.

5. **Vector Tool**

   * **Name/Description:** “Pinecone” / “Vector context”
   * **Role:** Exposes retrieved vectors as a tool to the RAG Agent, supplying semantic context.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Holds any conversational context or intermediate data during the single run.

7. **Chat Model (OpenAI Chat)**

   * **Model:** GPT‑3.5 or GPT‑4 (configurable)
   * **Credentials:** OpenAI API key
   * **Role:** Drafts the forward or summary message, using the retrieved context and a system prompt.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an assistant for Forward Attachments”
   * **Connected Tools:**

     * **Pinecone Vector Tool** (for context)
     * **Window Memory** (for temporary state)
     * **OpenAI Chat Model** (for generation)
   * **Role:** Orchestrates retrieval and LLM generation to produce the final output.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append row to the “Log” sheet
   * **Columns:** “Status” (e.g., “Success”), “Summary” or “Forwarded To”
   * **Credentials:** Google Sheets OAuth2
   * **Role:** Logs each processed attachment for auditing.

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Forward Attachments error: {$json.error.message}`
   * **Credentials:** Slack API token
   * **Role:** Sends an immediate alert if any step fails.

---

## Credentials Required

* **OPENAI\_API** (for embeddings & chat)
* **PINECONE\_API** (for vector storage/query)
* **SHEETS\_API** (Google Sheets OAuth2)
* **SLACK\_API** (for error alerts)

---

## Execution Order

Set to version **v1**, ensuring nodes respect the defined sequence.

---

## Usage & Setup

1. **Import Workflow**

   * Load `forward_attachments.json` into your n8n instance.
2. **Configure Credentials**

   * Provide OpenAI, Pinecone, Google Sheets, and Slack credentials.
3. **Deploy & Activate**

   * Enable the workflow.
   * Ensure your mail service forwards attachments to the `/forward-attachments` webhook.
4. **Invoke & Monitor**

   * Verify Google Sheets for logs.
   * Watch Slack for any error alerts.

---

## Key Benefits

* **Seamless RAG Pipeline:** Indexes and retrieves attachment content for rich summaries.
* **Automated Forwarding:** Reduces manual handling of email attachments.
* **Observability:** Logging and alerting provide full visibility into processing status.
* **Extensible Design:** Swap in different LLMs or vector stores as needed.
