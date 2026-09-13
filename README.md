<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/33036dfb-7131-462c-a5ac-d4cdf98cd207" />
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/2137ef71-9ac8-4020-9749-33a64edd0f28" />
# Enterprise RAG AI Agent System (n8n, Supabase pgvector & Gemini)

A production-grade, end-to-end Retrieval-Augmented Generation (RAG) system built with **n8n**, **Supabase (`pgvector`)**, and **Google Gemini**. The system automates document ingestion from Google Drive, performs dynamic text chunking, stores high-dimensional embeddings, and provides context-aware answers through an interactive AI Chat Agent.

---

## 🌟 Key Features

* **Automated Data Ingestion**: Real-time triggering and download of documents uploaded to Google Drive.
* **Smart Text Chunking**: Integrates `Recursive Character Text Splitter` to optimize context length and retrieval precision.
* **3072-Dimensional Vector Store**: Configured with Supabase (`pgvector`) and custom SQL match functions (`match_documents`) to support Google Gemini Embeddings.
* **Metadata Context Filtering**: Uses dynamic key filtering (`metadata->>file_id`) to isolate search contexts between different project files and prevent data bleed.
* **Persistent Chat Memory**: Stores multi-turn conversation history using `Postgres Chat Memory` (`n8n_chat_histories`).
* **Structured AI Responses**: Enforces structured Markdown formatting (headers, key metrics, bullet points, and tables) for professional client-facing outputs.

---

## 🏗️ System Architecture

The project consists of two core workflows linked via a shared Supabase database:

1. **Ingestion Pipeline**: `Google Drive Trigger` ➔ `File Extraction` ➔ `Recursive Text Splitter` ➔ `Gemini Embeddings` ➔ `Supabase Vector Store`.
2. **RAG Chat Agent**: `Chat Trigger` ➔ `AI Agent` ➔ `Supabase Vector Search (with file_id filter)` ➔ `Gemini Model` ➔ `Postgres Memory`.

---

## 🛠️ Tech Stack

* **Workflow Orchestration**: n8n
* **Vector Database**: Supabase (PostgreSQL with `pgvector` extension)
* **Embedding Model**: Google Gemini Text Embeddings (3072 dimensions)
* **LLM & Reasoning**: Google Gemini / OpenRouter AI
* **Document Ingestion**: Google Drive API
* **Deployment Environment**: Docker Desktop (Windows) with `ngrok` Webhook Tunneling

---

## ⚙️ Database Setup (Supabase SQL)

To support 3072-dimensional embeddings, run the following SQL script in Supabase:

```sql
-- 1. Update embedding column dimension
ALTER TABLE documents ALTER COLUMN embedding TYPE vector(3072);

-- 2. Update vector search function with metadata filtering
CREATE OR REPLACE FUNCTION match_documents (
  query_embedding vector(3072),
  match_count int DEFAULT null,
  filter jsonb DEFAULT '{}'
) RETURNS TABLE (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
LANGUAGE plpgsql
AS $$ BEGIN   RETURN QUERY   SELECT     documents.id,     documents.content,     documents.metadata,     1 - (documents.embedding <=> query_embedding) AS similarity   FROM documents   WHERE documents.metadata @> filter   ORDER BY documents.embedding <=> query_embedding   LIMIT match_count; END; $$;
