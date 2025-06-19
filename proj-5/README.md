# AI\_ML

* **daily\_content\_ideas.json** – Uses Cohere Embeddings, OpenAI Chat, Supabase Vector

---

## Workflow Purpose

Automates the generation of fresh social‑media content ideas on a daily cadence. When triggered via webhook, it ingests recent content (e.g., posts, articles), indexes it in Supabase Vector, retrieves the most relevant context, and uses OpenAI to propose new, engaging ideas.

---

## Trigger

1. **Webhook Trigger**

   * **Type:** HTTP POST
   * **Path:** `/daily-content-ideas`
   * **Purpose:** Entry point for a daily run. Can be called by an external scheduler or n8n’s built‑in cron/webhook trigger to kick off content‑idea generation.

---

## Main Nodes & Flow

1. **Text Splitter**

   * **Type:** Character‑based splitter
   * **Settings:**

     * Chunk size: 400 characters
     * Overlap: 40 characters
   * **Role:** Breaks incoming text (e.g., blog posts, tweets, newsletter snippets) into chunks for embedding.

2. **Embeddings (Cohere)**

   * **Model:** `embed-english-v3.0`
   * **Credentials:** Cohere API key
   * **Role:** Converts each text chunk into vector embeddings suitable for similarity search.

3. **Supabase Insert**

   * **Vector Store:** Supabase Vector
   * **Index Name:** `daily_content_ideas`
   * **Role:** Inserts new embeddings—keeping an up‐to‐date index of source content.

4. **Supabase Query**

   * **Index Name:** `daily_content_ideas`
   * **Role:** Retrieves top‑k similar content chunks based on a fresh input prompt or seed keywords.

5. **Vector Tool**

   * **Name/Description:** “Supabase” / “Vector context”
   * **Role:** Exposes retrieved vectors as a tool to the RAG Agent for context‑aware generation.

6. **Window Memory**

   * **Type:** Memory buffer (windowed)
   * **Role:** Holds conversation or run‑specific context—useful if the webhook payload includes prior run metadata or feedback.

7. **Chat Model (OpenAI)**

   * **Model:** GPT‑3.5 or GPT‑4 (configurable)
   * **Credentials:** OpenAI API key
   * **Role:** Generates a list of novel content ideas, guided by retrieved context and a system prompt.

8. **RAG Agent**

   * **Prompt Type:** `define`
   * **System Message:** “You are an AI assistant for Daily Content Ideas”
   * **Connected Tools:**

     * **Supabase Vector Tool** (for context snippets)
     * **Window Memory** (for any prior state)
     * **OpenAI Chat Model** (for idea generation)
   * **Role:** Orchestrates context retrieval and LLM generation to output a structured set of content ideas.

---

## Post‑Processing & Notifications

1. **Append Sheet (Google Sheets)**

   * **Operation:** Append new row to the “Log” sheet
   * **Columns:** “Status” (e.g., “Success”), “Ideas” (the generated list)
   * **Credentials:** Google Sheets OAuth2
   * **Role:** Persists each day’s ideas for audit and review.

2. **Slack Alert**

   * **Channel:** `#alerts`
   * **Message:** `Daily Content Ideas error: {$json.error.message}`
   * **Credentials:** Slack API token
   * **Role:** Notifies the team immediately on any failures in the RAG Agent or downstream nodes.

---

## Credentials Required

* **COHERE\_API** (for embeddings)
* **SUPABASE\_API** (for vector storage/query)
* **OPENAI\_API** (for chat generation)
* **SHEETS\_API** (Google Sheets OAuth2)
* **SLACK\_API** (for error alerts)

---

## Execution Order

Configured as **v1**, ensuring nodes fire in sequence according to the defined connections.

---

## Usage & Setup

1. **Import Workflow**

   * Upload `daily_content_ideas.json` into n8n.
2. **Configure Credentials**

   * Add your Cohere, Supabase, OpenAI, Google Sheets, and Slack credentials in n8n.
3. **Deploy & Activate**

   * Turn the workflow on.
   * Schedule an HTTP POST daily (5 AM) via external cron or n8n’s own scheduler pointing to `/daily-content-ideas`.
4. **Invoke & Monitor**

   * Check Google Sheets for the logged idea lists.
   * Watch Slack for error notifications.

---

## Key Benefits

* **Automated RAG Pipeline:** Keeps vector index fresh with the latest source content.
* **Daily Freshness:** Ensures new ideas every day without manual intervention.
* **Modular & Extensible:** Easily swap embedding models or vector stores, or add new content sources.
* **Observability:** Logging and alerts guarantee you see results and catch failures fast.
