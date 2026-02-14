# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A RAG (Retrieval-Augmented Generation) chatbot that answers questions about course materials. It uses ChromaDB for vector storage, Anthropic Claude for AI generation, and sentence-transformers for embeddings, served via a FastAPI backend with a static HTML/JS/CSS frontend.

## Commands

```bash
# Install dependencies
uv sync

# Run the app (starts FastAPI on port 8000)
./run.sh
# or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

The web interface and API docs are at `http://localhost:8000` and `http://localhost:8000/docs`.

## Architecture

**Request flow:** Frontend → FastAPI (`backend/app.py`) → `RAGSystem` → `AIGenerator` (Claude API with tool use) → `CourseSearchTool` → `VectorStore` (ChromaDB) → response back through the chain.

The system uses **Claude tool use** for search: when a user asks a course-specific question, Claude decides whether to call the `search_course_content` tool, which queries ChromaDB. The AI generator handles the tool execution loop in `_handle_tool_execution`.

### Key Backend Modules

- **`rag_system.py`** — Main orchestrator that wires together all components
- **`ai_generator.py`** — Claude API client with tool-use handling; system prompt lives here as `SYSTEM_PROMPT`
- **`vector_store.py`** — ChromaDB wrapper with two collections: `course_catalog` (metadata) and `course_content` (chunks). Course names are resolved via semantic search against the catalog before filtering content.
- **`document_processor.py`** — Parses course documents (expected format: title/link/instructor header lines, then `Lesson N:` markers) and chunks text by sentence boundaries
- **`search_tools.py`** — Tool abstraction (`Tool` ABC, `ToolManager`, `CourseSearchTool`). New tools should extend `Tool` and be registered with `ToolManager`.
- **`session_manager.py`** — In-memory conversation history per session (not persistent across restarts)
- **`config.py`** — All configuration as a dataclass; reads `ANTHROPIC_API_KEY` from `.env`
- **`models.py`** — Pydantic models: `Course`, `Lesson`, `CourseChunk`

### Frontend

Plain HTML/JS/CSS in `frontend/`. No build step. Served as static files by FastAPI.

### Document Format

Course documents in `docs/` follow this structure:
```
Course Title: ...
Course Link: ...
Course Instructor: ...

Lesson 0: Title
Lesson Link: ...
Content...

Lesson 1: Title
...
```

Documents are auto-loaded from `../docs` on server startup. The server runs from the `backend/` directory, so paths are relative to that.

## Environment

- Python 3.13, managed with `uv`
- Requires `ANTHROPIC_API_KEY` in `.env` (see `.env.example`)
- ChromaDB persists to `backend/chroma_db/`
- Embedding model: `all-MiniLM-L6-v2` (sentence-transformers)
- AI model: `claude-sonnet-4-20250514` (configured in `config.py`)
