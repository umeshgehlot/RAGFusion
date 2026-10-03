# RAGFusion web interface

This Next.js and TypeScript app provides a single-page chat interface for the RAGFusion API. It sends a question to `POST /chat/stream`, renders the response as it arrives over Server-Sent Events, and shows which retrieval tools the agent used.

## Requirements

- Node.js and npm
- Access to a RAGFusion API that permits requests from the frontend origin

## Run locally

From this directory:

```powershell
npm install
npm run dev
```

Open <http://localhost:3000>. `npm run build` creates a production build and `npm start` serves that build.

## API configuration

The backend URL is currently a constant in `app/page.tsx` and points to the hosted demo API. It is not currently read from an environment variable. To use a local backend, change that URL to the API address (usually `http://localhost:8058`) and make sure the backend's CORS configuration allows `http://localhost:3000`.

## Project files

- `app/page.tsx` — chat interaction and SSE parsing
- `app/layout.tsx` — root layout and global stylesheet import
- `app/globals.css` — global styles

See the repository [README](../README.md) for backend, ingestion, and database setup.
