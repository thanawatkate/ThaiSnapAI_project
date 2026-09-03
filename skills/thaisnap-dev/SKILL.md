---
name: thaisnap-dev
description: >-
  Develop and run ThaiSnapAI (FastAPI + Streamlit + AiHub). Use when working
  under ThaiSnapAI/, scripts/dev, ports 8000/8501/3400, or app features.
---

# ThaiSnapAI Dev

Full team rules live in `ThaiSnapAI/AGENTS.md` — read that first for any non-trivial change.

## Run

```bash
cd ThaiSnapAI
./scripts/dev.sh          # AiHub :3400 + API :8000 + UI :8501
SKIP_AI_HUB=1 ./scripts/dev.sh   # skip hub
```

Windows: `scripts/dev.ps1`.

## Must

- UI logic in `frontend/adapters/web/` — `frontend/pages/` = thin wrappers only
- `frontend/` never `import backend` — HTTP via `frontend/api_client/` only
- Domain in `backend/core` + `backend/infra` — infra must not import frontend
- Shared assets/config at `ThaiSnapAI/shared/`
- No secrets in commits (`.env`, credentials)
- Streamlit: native widgets + `frontend/.streamlit/config.toml` only — no injected CSS/HTML theme
- After image/UI edits the photo must not flicker — verify on the real editor before finishing (`skills/no-image-flicker`)
- Tests: `python tests/test_quick.py` from `ThaiSnapAI/`

## AiHub link

Text work uses hub when `AI_HUB_URL` + `AI_HUB_API_KEY` set. See `ThaiSnapAI/docs/guides/ai/AI_HUB.md`.
