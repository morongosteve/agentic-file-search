# CLAUDE.md

## Project Overview

Agentic file search: an AI-powered document search agent that navigates files dynamically using a three-phase strategy (parallel scan → deep dive → backtrack for cross-references). Unlike RAG, it reasons over documents without pre-computed embeddings.

## Runtime & Package Manager

- Python ≥ 3.10, managed with **uv**
- Package: `fs-explorer` (entry points: `explore`, `explore-ui`)

## Development Commands

```bash
# Install
uv pip install -e ".[dev]"

# Run CLI
uv run explore --task "What is the purchase price in data/test_acquisition/?"

# Run web UI (http://127.0.0.1:8000)
uv run uvicorn fs_explorer.server:app --host 127.0.0.1 --port 8000

# Tests
uv run pytest

# Lint / type-check
uv run ruff check .
uv run ty check
```

## Project Structure

```
src/fs_explorer/
├── agent.py      # Gemini API client, token tracking
├── workflow.py   # LlamaIndex Workflows (event-driven orchestration)
├── fs.py         # File tools: scan_folder, preview_file, parse_file, read, grep, glob
├── models.py     # Pydantic action models
├── main.py       # CLI entry point (Typer)
└── server.py     # FastAPI + WebSocket server
    ui.html       # Single-file web UI
data/
├── test_acquisition/    # 10 interconnected legal docs for testing
└── large_acquisition/   # 25 docs with cross-references
```

## Architecture

The agent uses **LlamaIndex Workflows** (event-driven, not linear chains):

1. `scan_folder` — preview all documents in parallel
2. `parse_file` — full extraction on promising files only
3. Cross-reference backtracking via `grep`/`glob` on references found

LLM: Google Gemini 2.0 Flash (structured JSON output). Document parsing: Docling (local, offline).

## Configuration

Create `.env` in project root:
```
GOOGLE_API_KEY=your_api_key
```

## Key Conventions

- All agent actions return structured `ActionResponse` Pydantic models
- Token usage is tracked per query in `agent.py`; cost is logged after each run
- The web UI uses WebSocket for real-time step streaming — keep server events JSON-serializable
- Docling parsing is local and CPU-bound; avoid calling `parse_file` unnecessarily (use `preview_file` first)
