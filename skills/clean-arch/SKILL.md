---
name: clean-arch
description: >-
  Clean Architecture boundaries for ThaiSnapAI backend/frontend. Use when
  adding use cases, ports, adapters, API routes, or moving code between layers.
---

# Clean Architecture

Canonical docs: `ThaiSnapAI/docs/ARCHITECTURE.md`, `ThaiSnapAI/AGENTS.md`.

## Layers

| Layer | Path | May depend on |
|-------|------|----------------|
| Core | `ThaiSnapAI/backend/core/` | nothing outside core |
| Infra | `ThaiSnapAI/backend/infra/` | core (implements ports) |
| API | `ThaiSnapAI/backend/api/` | core + infra wiring |
| UI | `ThaiSnapAI/frontend/` | HTTP API only |

## Must

1. New business logic → `core/usecases/` + ports in `core/ports/`
2. External I/O (DB, scrape, render, HTTP clients) → `infra/` implementing ports
3. Entities in `core/entities/` stay free of Streamlit/FastAPI/MoviePy types
4. `frontend/` never imports `backend`
5. `infra` never imports `frontend`
6. Prefer edit existing adapters over bypassing ports from API/UI

## Smell test

If `core/` gains `import streamlit`, `import fastapi`, or scraper HTML parsers — wrong layer. Move out.
