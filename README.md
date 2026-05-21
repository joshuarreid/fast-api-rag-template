# FastAPI RAG Template

A reusable FastAPI starter for retrieval-augmented generation with Qdrant, OpenAI, and sentence-transformers.

## Features

- FastAPI service with `/health`, `/ingest`, and `/rag` endpoints
- Environment-based configuration for managed Qdrant deployments
- Embeddings via sentence-transformers or OpenAI
- Automatic Qdrant collection creation on startup
- Dockerfile ready for DigitalOcean App Platform

## Project Structure

```text
.
├── app/
│   ├── __init__.py
│   ├── db_client.py
│   ├── ingest.py
│   ├── main.py
│   └── rag.py
├── .env.example
├── .gitignore
├── Dockerfile
├── README.md
└── requirements.txt
```

## Configuration

Copy `.env.example` to `.env` and set the values for your environment.

```env
VECTORDB_URL=https://your-qdrant-endpoint
VECTORDB_API_KEY=your-qdrant-api-key
COLLECTION_NAME=documents
EMBEDDINGS_PROVIDER=sentence_transformers
EMBEDDINGS_MODEL=all-MiniLM-L6-v2
OPENAI_API_KEY=
OPENAI_CHAT_MODEL=gpt-4o-mini
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
QDRANT_VECTOR_SIZE=384
QDRANT_DISTANCE=cosine
RAG_TOP_K=3
```

Notes:

- Use `EMBEDDINGS_PROVIDER=openai` to generate embeddings with OpenAI instead of sentence-transformers.
- For `sentence_transformers`, keep `QDRANT_VECTOR_SIZE` aligned with the model dimension. `all-MiniLM-L6-v2` uses `384`.
- For `openai`, `text-embedding-3-small` uses `1536` and `text-embedding-3-large` uses `3072`.

## Local Development

1. Create a virtual environment and install dependencies.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

2. Copy the example environment file.

```bash
cp .env.example .env
```

3. Start the API.

```bash
uvicorn app.main:app --reload --port 8080
```

## API Usage

Ingest a document:

```bash
curl -X POST http://localhost:8080/ingest \
  -H "Content-Type: application/json" \
  -d '{
    "doc_id": "doc-1",
    "text": "FastAPI is a modern Python web framework for building APIs.",
    "metadata": {"source": "example"}
  }'
```

Query the RAG endpoint:

```bash
curl "http://localhost:8080/rag?query=What%20is%20FastAPI%3F"
```

## Docker

Build and run locally:

```bash
docker build -t fast-api-rag-template .
docker run --env-file .env -p 8080:8080 fast-api-rag-template
```

## DigitalOcean App Platform

1. Push the repository to GitHub.
2. Create an App Platform app using Docker as the build type.
3. Set environment variables in App Platform instead of committing secrets.
4. Deploy the app. The container listens on port `8080` by default and also respects the `PORT` environment variable.

## Implementation Notes

- The app creates the configured Qdrant collection on startup if it does not exist.
- If `OPENAI_API_KEY` is not set, `/rag` returns retrieved context with a placeholder answer instead of calling an LLM.
- Qdrant payloads store the original text under `text` and arbitrary metadata under `metadata`.
# fast-api-rag-template
