# RAGFusion — Agentic Multi-Source Retrieval & Knowledge Graph

RAGFusion is an experimental document-question-answering system that routes questions to vector, hybrid, graph, page-level, and document retrieval tools. A Pydantic AI agent uses retrieved evidence to produce answers; a FastAPI service exposes the agent and direct-search endpoints.

**Hosted API documentation:** <https://ragfusion-1065091982503.us-east1.run.app/docs>

## How it works

- **Chunk retrieval:** PostgreSQL with pgvector supports semantic search; hybrid search combines vector similarity with keyword ranking.
- **Knowledge graph:** Neo4j and Graphiti support entity, relationship, and timeline queries. Graph ingestion is a separate step from vector ingestion.
- **Page retrieval:** whole-page embeddings can return document pages without splitting their context.
- **PageIndex:** an optional VectifyAI cloud API can search PDF sections using a remotely built document tree. This is different from local page-vector search.
- **Agent/API:** FastAPI serves regular and Server-Sent Events (SSE) chat responses, search endpoints, and document/session operations.

The backend uses PostgreSQL/pgvector and Neo4j. LLM, embedding, and PageIndex providers are configured through environment variables; see [.env.example](.env.example) for the available settings.

## Repository layout

```text
agent/          Agent, API, providers, prompts, database and graph helpers
ingestion/      Chunking, embedding, graph and PageIndex ingestion
scripts/        Fast and graph ingestion entry points
sql/schema.sql  PostgreSQL schema (destructive reset script; see warning below)
frontend/       Next.js user interface
tests/          Agent and ingestion tests
cli.py          Interactive terminal client
mcp_server.py   MCP server exposing selected retrieval tools
```

## Requirements

- Python 3.11
- PostgreSQL with the pgvector extension
- Neo4j for graph functionality
- Provider credentials for the selected LLM and embedding models
- Node.js and npm only if running the frontend

## Configure and run the API

From the project root, create a local environment file and fill in the database, Neo4j, and provider settings:

```powershell
Copy-Item .env.example .env
```

Install the Python dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

> **Database warning:** `sql/schema.sql` drops and recreates application tables. Use it only for a new or disposable development database; it deletes existing data. `.env` is not automatically imported into PowerShell. If initializing with `psql`, set `$env:DATABASE_URL` in the current shell to the database URL first.

After configuring PostgreSQL and Neo4j, initialize a fresh development database if needed, then start the API. The CLI in this checkout currently treats the healthy API's `/health` status (`ok`) as unhealthy; use the web/API directly or update that check before relying on the CLI:

```powershell
psql $env:DATABASE_URL -f sql/schema.sql
python -m agent.api
```

The API listens on port 8058 by default (the `PORT` environment variable can override it). Open <http://localhost:8058/docs> for the interactive API reference. The live hosted API is also linked above.

## Ingest documents

The repository does not include a `documents/` corpus. Provide your own supported files in a folder before ingesting. The fast pipeline stores vector chunks and PDF page embeddings; it does not populate the graph. The ingestion script may delete existing rows when `--clean` is supplied, so do not add that flag unless you intend to clear the ingested data.

```powershell
python scripts/ingest_fast.py --documents documents/
```

To create graph data from ingested chunks, run the graph pipeline separately. `--limit` is useful for controlling provider usage during a trial run:

```powershell
python scripts/ingest_graph.py --limit 50
```

PDFs can optionally be uploaded to PageIndex, which requires `PAGEINDEX_API_KEY` and uses the external PageIndex service:

```powershell
python ingestion/ingest_pageindex.py --dry-run
```

## Frontend and CLI

The frontend is a small Next.js app that streams answers and displays the retrieval tools used. Its API URL is currently configured in the page source to use the hosted backend; local use requires changing that URL to the local API and ensuring the backend allows the frontend origin.

```powershell
cd frontend
npm install
npm run dev
```

The CLI is in `cli.py`; run it from another terminal after starting the API:

```powershell
python cli.py
```

## API overview

| Route | Method | Purpose |
|---|---|---|
| `/health` | GET | Service and dependency health |
| `/chat` and `/chat/stream` | POST | Agent answer, either regular JSON or SSE |
| `/search/vector`, `/search/graph`, `/search/hybrid` | POST | Direct search without agent routing |
| `/documents` | GET | List ingested documents |
| `/sessions/{session_id}` | GET | Read session history |
| `/metrics` | GET | Prometheus metrics |
| `/docs` | GET | Interactive OpenAPI documentation |

For the complete current route behavior, see `agent/api.py` and `agent/tools.py`.

## Tests and development checks

Install the development requirements, plus `pytest` and `pytest-cov` if they are not already available in your environment. The pytest configuration enables coverage and enforces an 80% threshold.

```powershell
python -m pip install -r requirements.txt -r requirements-dev.txt pytest pytest-cov
pytest
```

The application includes per-IP in-memory rate limiting and input-length checks on selected public endpoints. Authentication and shared cross-instance rate limiting are not implemented; do not treat the demo deployment as a hardened production service.

## License

MIT. See [LICENSE](LICENSE).
