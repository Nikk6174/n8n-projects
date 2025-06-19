# Social\_Media

* **auto‑dm\_new\_twitter\_followers.json** – Uses Cohere Embeddings, OpenAI Chat, Pinecone

---

## Workflow Purpose

Automatically sends personalized direct messages to new Twitter followers. When a user follows your account, this workflow captures their profile data, indexes it for context, and uses OpenAI to craft and send a tailored welcome DM.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/auto-dm-new-twitter-followers`
   * **Purpose:** Receives a payload whenever a new follower event is detected (set up via Twitter API → webhook subscription).

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Divides long bio or profile descriptions into chunks for embedding.

2. **Embeddings (Cohere)**

   * **Model:** `embed-english-v3.0`
   * **Credentials:** Cohere API key
   * **Role:** Converts each text chunk (e.g., bio, tweets) into vector embeddings.

3. **Pinecone Insert**

   * **Vector Store:** Pinecone
   * **Index Name:** `auto-dm_new_twitter_followers`
   * **Role:** Stores embeddings of follower data (bio, recent tweets, profile info) for similarity searches.

4. **Pinecone Query**

   * **Index Name:** `auto-dm_new_twitter_followers`
   * **Role:** Retrieves relevant context (e.g., interests or keywords) to inform the DM’s tone and content.

5. **Vector Tool**

   * **Name/Description:** “Pinecone” / “Vector context”
   * **Role:** Supplies retrieved embeddings as a tool to the RAG Agent, enriching its understanding of the new follower.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Temporarily holds the current event’s context (e.g., follower handle, timestamp) during processing.

7. **Chat Model (OpenAI Chat)**

   * **Model:** GPT‑3.5 or GPT‑4 (configurable)
   * **Credentials:** OpenAI API key
   * **Role:** Generates the direct message content, guided by system prompts and retrieved context.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an assistant for Auto‑DM New Twitter Followers.”
   * **Connected Tools:**

     * **Pinecone Vector Tool** (for follower context)
     * **Window Memory** (for event specifics)
     * **OpenAI Chat Model** (for DM generation)
   * **Role:** Coordinates context retrieval and LLM generation to produce a personalized welcome message.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append a row to the “Log” sheet
   * **Columns:** “Handle,” “MessageSnippet,” “Status”
   * **Credentials:** Google Sheets OAuth2
   * **Role:** Records each DM sent, enabling audit and follow‑up analysis.

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Auto‑DM New Twitter Followers error: {$json.error.message}`
   * **Credentials:** Slack API token
   * **Role:** Immediately flags any failures in context retrieval or message sending.

---

## Credentials Required

* **COHERE\_API** (for embeddings)
* **PINECONE\_API** (for vector storage/query)
* **OPENAI\_API** (for chat generation)
* **SHEETS\_API** (Google Sheets OAuth2)
* **SLACK\_API** (for error alerts)

---

## Execution Order

Set to **v1**, ensuring each node runs in the sequence defined by the connections.

---

## Usage & Setup

1. **Import Workflow**

   * Load `auto-dm_new_twitter_followers.json` into n8n.
2. **Configure Webhook**

   * Register your webhook URL with Twitter’s API for new‑follower events.
3. **Configure Credentials**

   * Provide Cohere, Pinecone, OpenAI, Google Sheets, and Slack credentials in n8n.
4. **Deploy & Activate**

   * Enable the workflow.
   * Ensure Twitter events are routed to the `/auto-dm-new-twitter-followers` webhook.
5. **Monitor**

   * Check Google Sheets for logs of sent messages.
   * Watch Slack for any error notifications.

---

## Key Benefits

* **Personalization at Scale:** Uses vector context to tailor each DM to the follower’s interests.
* **Seamless Integration:** Hooks directly into Twitter’s webhook system for real‑time engagement.
* **Visibility:** Logs and alerts keep you informed of successes and any issues.
* **Extensibility:** Easily adapt prompts or swap models/tools as your messaging strategy evolves.
