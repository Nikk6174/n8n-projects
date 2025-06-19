# Social\_Media

* **alert\_on\_instagram\_competitor\_story.json** – Uses OpenAI Chat, OpenAI Embeddings, Supabase Vector

---

## Workflow Purpose

Continuously monitors an Instagram competitor’s Story content, indexes text for semantic search, and sends an alert when specified keywords or topics appear.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/alert-on-instagram-competitor-story`
   * **Purpose:** Receives payloads containing the latest Story text or metadata (pushed from an external scraper or Instagram API integration).

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Breaks each incoming Story text block into chunks for better embedding coverage.

2. **Embeddings (OpenAI)**

   * **Model:** `text-embedding-3-small`
   * **Credentials:** OpenAI API key
   * **Role:** Converts each text chunk into a high‑dimensional vector representation.

3. **Supabase Insert**

   * **Vector Store:** Supabase Vector
   * **Index Name:** `alert_on_instagram_competitor_story`
   * **Role:** Stores new embeddings of the competitor’s Story text to maintain an up‑to‑date context index.

4. **Supabase Query**

   * **Index Name:** `alert_on_instagram_competitor_story`
   * **Role:** Retrieves the top-k most similar chunks based on pre‑configured alert keywords or a seed prompt.

5. **Vector Tool**

   * **Name/Description:** “Supabase” / “Vector context”
   * **Role:** Supplies retrieved chunks as a tool to the RAG Agent for contextual decision‑making.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Maintains transient context for the single run (e.g., which Story was last evaluated).

7. **Chat Model (OpenAI Chat)**

   * **Model:** GPT‑3.5 or GPT‑4 (configurable)
   * **Credentials:** OpenAI API key
   * **Role:** Reviews retrieved context and determines whether an alert should be triggered, crafting a human‑readable notification message.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an assistant for Alert on Instagram Competitor Story.”
   * **Connected Tools:**

     * **Supabase Vector Tool** (for context retrieval)
     * **Window Memory** (for run state)
     * **OpenAI Chat Model** (for analysis and notification content)
   * **Role:** Coordinates context retrieval with LLM reasoning to decide if an alert condition is met and to generate the alert payload.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append a row to the “Log” sheet
   * **Columns:** “Timestamp,” “KeywordsDetected,” “Status”
   * **Credentials:** Google Sheets OAuth2
   * **Role:** Records each evaluation and its outcome for auditing and trend analysis.

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Alert on Instagram Competitor Story: {$json.keywordsDetected}`
   * **Credentials:** Slack API token
   * **Role:** Immediately notifies the team when target keywords or topics are found.

---

## Credentials Required

* **OPENAI\_API** (for embeddings & chat)
* **SUPABASE\_API** (for vector storage/query)
* **SHEETS\_API** (Google Sheets OAuth2)
* **SLACK\_API** (for sending alerts)

---

## Execution Order

Configured as **v1**, ensuring strict node sequencing as defined.

---

## Usage & Setup

1. **Import Workflow**

   * Load `alert_on_instagram_competitor_story.json` into your n8n instance.
2. **Configure Source**

   * Set up an external process or API that posts new Story content to the `/alert-on-instagram-competitor-story` webhook.
3. **Configure Credentials**

   * Add your OpenAI, Supabase, Google Sheets, and Slack credentials.
4. **Deploy & Activate**

   * Enable the workflow.
   * Ensure external scraper or API subscriptions push Story updates to the webhook.
5. **Monitor & Iterate**

   * Review Google Sheets logs for detected keywords.
   * Tweak system prompts or vector‑query parameters to refine alerts.

---

## Key Benefits

* **Proactive Monitoring:** Automatically flags competitor activity in real time.
* **Context‑Aware Alerts:** Uses vector similarity to detect nuanced mentions or topics.
* **Transparent Logging:** Maintains a clear audit trail of detections.
* **Flexible Customization:** Easily adjust keywords, models, or storage backend as your monitoring needs evolve.
