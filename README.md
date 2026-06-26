# Document Copilot

Document Copilot is a grounded research assistant for analysts built around SEC filing retrieval, grounding, and generative answer streaming. The repo is organized as a React + TypeScript frontend, a Python + FastAPI backend, and Supabase-backed persistence for authentication, documents, embeddings, and chat history.

## Why this project exists

The product is designed to reduce analyst intake work by turning curated SEC filings into a searchable, trusted conversational interface.

- Users ask questions in plain English about filings.
- The system retrieves evidence from a curated corpus of documents.
- Answers are generated only from retrieved source passages.
- Claims are backed by citations and original filing text.

## Tech stack

### Frontend

- React + Vite + TypeScript
- React Router for client routing
- Tailwind CSS with shadcn/ui patterns
- `@supabase/supabase-js` for browser auth and session management
- Vercel AI SDK packages for chat primitives and streaming UI behavior

### Backend

- Python 3.12+
- FastAPI + Uvicorn
- Pydantic v2 + pydantic-settings
- PydanticAI for typed LLM orchestration
- OpenAI SDK for embeddings and generation
- Supabase Python client for database and auth access
- SQLAlchemy + Alembic for schema modeling and migrations
- `pgvector` for vector search and Postgres full-text search for lexical lookup
- `httpx` for outbound network calls
- `structlog` for structured logging

### Persistence and infrastructure

- Supabase Auth for identity
- Supabase Postgres for product state
- Supabase `pgvector` for embedding-powered retrieval
- Railway-ready deployment model with separate frontend and backend services

## Architecture

Document Copilot separates the system into three main layers:

1. Browser UI
   - Handles login, chat UI, and streaming assistant responses.
   - Uses Supabase anon credentials only.
   - Sends authenticated requests to the backend.

2. Backend API
   - Verifies Supabase JWTs.
   - Runs retrieval, grounding, and LLM orchestration.
   - Persists chats, citations, documents, chunks, and metadata.
   - Streams assistant output back to the browser.

3. Data layer
   - Stores source documents, encoded chunks, embeddings, and chat state.
   - Runs hybrid retrieval with semantic and lexical search.
   - Supplies evidence for grounded answers.

### High-level flow

- User signs in through Supabase Auth in the frontend.
- Frontend sends chat requests with `Authorization: Bearer <token>` to FastAPI.
- Backend verifies the token, retrieves relevant document chunks, and calls OpenAI.
- Backend returns a grounded, cited answer and writes chat state to Supabase.
- Frontend renders the answer incrementally.

## Repo layout

- `backend/` — FastAPI service, agent orchestration, retrieval, grounding, and persistence
- `frontend/` — React SPA and UI components
- `data/` — ingestion helpers and document download utilities
- `docs/` — architecture notes, setup guides, and product brief

## Developer workflow

### Backend

1. Copy environment variables and fill secrets:
   ```bash
   cd backend
   cp .env.example .env
   ```
2. Install or sync dependencies:
   ```bash
   uv sync
   ```
3. Run the API locally:
   ```bash
   uv run uvicorn app.main:app --reload
   ```
4. Run migrations:
   ```bash
   uv run alembic upgrade head
   ```
5. Run tests:
   ```bash
   uv run pytest
   ```

### Frontend

1. Install dependencies:
   ```bash
   cd frontend
   pnpm install
   ```
2. Start the dev server:
   ```bash
   pnpm dev
   ```
3. Type check and lint:
   ```bash
   pnpm tsc --noEmit
   pnpm lint
   ```

### Data ingestion

- Use `data/download.py` from the repo root to fetch sample SEC data.
- Ingestion is designed to chunk filings, build embeddings, and store them in Supabase.

## Core concepts

### Grounded answers

The product guarantees trust by requiring the backend to:

- retrieve supporting document passages,
- call the LLM with only grounded context,
- validate citation mappings,
- refuse to answer when evidence is insufficient.

### Hybrid retrieval

Retrieval combines:

- semantic ranking over chunk embeddings,
- lexical ranking via Postgres full-text search,
- reciprocal rank fusion to merge results.

### PydanticAI orchestration

The backend uses a typed agent boundary to define:

- the assistant contract,
- allowed tools and dependencies,
- the answer payload shape,
- citation and source-passage output.

## Recommended reading

- `docs/architecture.md` — system architecture and flow
- `docs/guides/backend-setup.md` — backend developer setup
- `docs/guides/frontend-setup.md` — frontend developer setup
- `docs/client-brief.md` — product and user problem definition

## Deployment notes

- The backend should run as a stateless FastAPI service.
- The frontend is a Vite-built SPA.
- Supabase holds auth and durable product state.
- The frontend must never receive the Supabase service-role key.
- All privileged writes should happen on the backend.

## Testing and validation

- Backend tests live under `backend/tests/`
- Use `uv run pytest` to run the suite.
- Focus on retrieval, grounding, chat orchestration, and LLM output contracts.

## Contributing

- Follow the existing architecture: keep the browser thin and the backend authoritative.
- Add features by extending the backend agent, retrieval, or grounding layers rather than embedding more logic in the frontend.
- Keep configuration centralized in `backend/app/config.py` and `frontend/lib/env.ts`.

---

Document Copilot is built to help analysts trust every answer. Keep the system grounded, test the retrieval layer, and preserve citation integrity as you contribute.
