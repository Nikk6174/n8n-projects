# Social\_Media

* **auto‑dm\_new\_twitter\_followers.json** – Uses Cohere Embeddings, OpenAI Chat, Pinecone

---

## Workflow Purpose

Automatically sends personalized direct messages to new Twitter followers. This workflow captures each new follower’s publicly available profile data (bio, recent tweets, interests), transforms it into vector embeddings for context retrieval, and leverages OpenAI to craft a warm, relevant welcome DM—then dispatches it via the Twitter API.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/auto-dm-new-twitter-followers`
   * **Purpose:** Listens for “follow” events from Twitter’s Account Activity API. When a user follows your account, Twitter sends a POST payload to this endpoint, which kicks off the workflow.

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter (`@n8n/n8n-nodes-langchain.textSplitterCharacterTextSplitter`)
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Breaks lengthy follower bios or aggregated recent tweets into overlapping chunks, ensuring semantic embedding captures context without truncation.

2. **Embeddings (Cohere)**

   * **Model:** `embed-english-v3.0`
   * **Credentials:** Cohere API key (`COHERE_API`)
   * **Role:** Transforms each text chunk (bio, tweets, location, etc.) into high‑dimensional vector embeddings for efficient similarity search.

3. **Pinecone Insert**

   * **Vector Store:** Pinecone
   * **Index Name:** `auto-dm_new_twitter_followers`
   * **Role:** Inserts the follower’s embedding vectors into the Pinecone index, building a repository of profile contexts.

4. **Pinecone Query**

   * **Index Name:** `auto-dm_new_twitter_followers`
   * **Role:** At DM‑generation time, retrieves the most semantically relevant chunks (e.g., key interests, hobbies) to inform the tone and content of the message.

5. **Vector Tool**

   * **Name/Description:**

     * Name: “Pinecone”
     * Description: “Vector context of new follower”
   * **Role:** Exposes the Pinecone query results as a callable tool to the RAG Agent, supplying targeted context for personalization.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Temporarily holds event‑specific data—such as follower handle, timestamp, and retrieval outputs—so the agent can reference them during the LLM call.

7. **Chat Model (OpenAI Chat)**

   * **Model:** GPT‑3.5‑Turbo or GPT‑4 (configurable)
   * **Credentials:** OpenAI API key (`OPENAI_API`)
   * **Role:** Given system and user prompts (plus retrieved context), generates a concise, engaging direct message tailored to the new follower.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an assistant for Auto‑DM New Twitter Followers. Craft a friendly, personalized welcome message based on the follower’s profile context.”
   * **Connected Tools:**

     1. Pinecone Vector Tool (for context retrieval)
     2. Window Memory (for event details)
     3. OpenAI Chat Model (for generation)
   * **Role:** Coordinates retrieval of relevant profile embeddings and uses the LLM to produce a topically tailored DM.

9. **Send Twitter DM**

   * **Type:** HTTP Request / Twitter API node
   * **Endpoint:** `POST https://api.twitter.com/1.1/direct_messages/events/new.json`
   * **Payload:**

     ```json
     {
       "event": {
         "type": "message_create",
         "message_create": {
           "target": { "recipient_id": "<follower_user_id>" },
           "message_data": { "text": "{{ $json[\"RAG Agent\"].text }}" }
         }
       }
     }
     ```
   * **Credentials:** Twitter API OAuth token
   * **Role:** Delivers the generated message directly into the follower’s inbox.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append row to the “Log” sheet
   * **Columns Mapped:**

     * **Handle:** The follower’s Twitter handle
     * **MessageSnippet:** First 100 characters of the DM
     * **Status:** “Sent” or error code
   * **Credentials:** Google Sheets OAuth2 (`SHEETS_API`)
   * **Role:** Maintains an audit trail of every DM sent, facilitating analytics and troubleshooting.

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Auto‑DM New Twitter Followers error: {$json.error.message}`
   * **Credentials:** Slack API token (`SLACK_API`)
   * **Role:** Immediately flags any failures—whether in embedding, retrieval, DM generation, or Twitter API call—so you can intervene quickly.

---

## Credentials Required

* **COHERE\_API** – Cohere embeddings
* **PINECONE\_API** – Vector storage/query
* **OPENAI\_API** – LLM chat generation
* **TWITTER\_API** – Twitter OAuth for webhooks & DM sending
* **SHEETS\_API** – Google Sheets OAuth2
* **SLACK\_API** – Slack bot token for alerts

---

## Execution Order

* **v1** execution mode ensures all dependencies (chunking → embeddings → indexing → retrieval → generation → DM send → logging) run in topological order.

---

## Usage & Setup

1. **Import Workflow**

   * In n8n, upload `auto-dm_new_twitter_followers.json`.

2. **Configure Twitter Webhook**

   * Register your `/auto-dm-new-twitter-followers` endpoint via Twitter’s Account Activity API dashboard.
   * Subscribe to “follow” events.

3. **Add Credentials**

   * Input Cohere, Pinecone, OpenAI, Twitter, Google Sheets, and Slack credentials under n8n’s Credentials settings.

4. **Activate Workflow**

   * Turn on the workflow in n8n.
   * Verify that Twitter’s webhook calls are reaching the endpoint (check n8n’s webhook logs).

5. **Monitor & Iterate**

   * View Google Sheets “Log” for records of sent DMs.
   * Watch the Slack `#alerts` channel for any errors.
   * Tweak system or user prompts in the RAG Agent to refine DM tone.

---

## Key Benefits

* **Hyper‑Personalization:** Leverages semantic embeddings to tailor each DM to the follower’s unique profile.
* **Real‑Time Engagement:** Hooks directly into Twitter’s webhook system for instant outreach.
* **Scalable & Reliable:** Pinecone indexing and windowed memory ensure fast, context‑rich generation even at high volume.
* **End‑to‑End Visibility:** Google Sheets logging and Slack alerts give you full transparency on performance and failures.
* **Flexible & Extensible:** Easily swap models (e.g., GPT‑4), adjust embeddings, or add sentiment analysis to refine messaging over time.
