# Mini RAG App

A minimal Retrieval-Augmented Generation (RAG) backend for document question answering. The app lets users upload PDF or text files, process them into chunks, store metadata in MongoDB, index embeddings in Qdrant, and generate answers using an LLM.

## Features

- Upload PDF and TXT documents
- Extract and chunk document text
- Store projects, files, and chunks in MongoDB
- Generate embeddings with Cohere or OpenAI-compatible providers
- Store and search vectors with Qdrant
- Generate RAG answers with OpenAI API or local Ollama models
- FastAPI endpoints for upload, processing, search, and answer generation
- Docker Compose setup for MongoDB
- Prompt templates with English and Arabic locale structure

## Architecture

```text
Client / API requests
        |
        v
FastAPI backend
        |
        +--> MongoDB: projects, uploaded assets, text chunks
        |
        +--> Qdrant: vector index and semantic search
        |
        +--> LLM provider: answer generation
             - OpenAI API
             - Ollama through OpenAI-compatible API
```

## Tech Stack

- Python 3.8+
- FastAPI
- MongoDB with Motor
- Qdrant
- LangChain text loaders and splitters
- OpenAI SDK
- Cohere SDK
- Docker Compose

## Setup

Create and activate a Python environment:

```bash
conda create -n mini-rag-app python=3.8
conda activate mini-rag-app
```

Install dependencies:

```bash
cd src
pip install -r requirements.txt
```

Create the app environment file:

```bash
cp .env.example .env
```

Update `.env` with your provider keys and model names.

## MongoDB

Start MongoDB with Docker Compose:

```bash
cd docker
cp .env.example .env
docker compose up -d
```

Example `docker/.env`:

```env
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=password
```

## Local LLM With Ollama

Install Ollama, then pull a model:

```bash
ollama pull dolphin-phi
```

Or use a newer small model:

```bash
ollama pull llama3.2:1b
```

Example generation config in `src/.env`:

```env
GENERATION_BACKEND="OPENAI"
OPENAI_API_KEY="ollama"
OPENAI_API_URL="http://localhost:11434/v1"
GENERATION_MODEL_ID="dolphin-phi:latest"
```

Ollama exposes an OpenAI-compatible API at:

```text
http://localhost:11434/v1
```

## Embeddings

The default example uses Cohere embeddings:

```env
EMBEDDING_BACKEND="COHERE"
COHERE_API_KEY="your_cohere_api_key"
EMBEDDING_MODEL_ID="embed-multilingual-light-v3.0"
EMBEDDING_MODEL_SIZE=384
```

If you change the embedding model, make sure `EMBEDDING_MODEL_SIZE` matches the model output dimension.

## Run The API

From the `src` directory:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 5000
```

## Basic Workflow

1. Upload a document:

```text
POST /api/v1/data/upload/{project_id}
```

2. Process the document into chunks:

```text
POST /api/v1/data/process/{project_id}
```

Recommended body:

```json
{
  "chunk_size": 500,
  "overlap_size": 100,
  "do_reset": 1
}
```

3. Push chunks into the vector database:

```text
POST /api/v1/nlp/index/push/{project_id}
```

Recommended body:

```json
{
  "do_reset": 1
}
```

4. Search similar chunks:

```text
POST /api/v1/nlp/index/search/{project_id}
```

Example body:

```json
{
  "text": "Which industry uses Lorem Ipsum?",
  "limit": 5
}
```

5. Generate a RAG answer:

```text
POST /api/v1/nlp/index/answer/{project_id}
```

Example body:

```json
{
  "text": "How did Lorem Ipsum move from print to digital publishing?",
  "limit": 5
}
```

## Notes For GitHub

- Do not commit real `.env` files or API keys.
- Commit `.env.example` only.
- Uploaded files, MongoDB data, and local vector databases are ignored by Git.
- Small local models are useful for testing, but larger API models usually produce better answers.

## Limitations

- This is a learning and portfolio project, not a production-hardened system.
- Error handling is basic and can be improved further.
- No automated test suite is included yet.
- Answer quality depends heavily on chunk size, retrieved context, embedding model, and generation model.

## License

This project is licensed under the Apache License 2.0.
