---
name: aihub-dev
description: >-
  Develop aihubproject (Node AI Hub, MySQL, quotas, billing). Use when working
  under aihubproject/, hub APIs, migrations, or docker compose for the hub.
---

# AiHub Dev

Product-agnostic hub. Consumers pass own `usageKey` / `feature`.

## Run

```bash
cd aihubproject
cp .env.example .env   # if needed
npm install
npm run migrate
npm run dev            # default http://localhost:3400
```

Docker: `docker compose up --build`  
Prod-like: `docker compose -f docker-compose.prod.yml up --build -d`

## Auth

| Who | Header |
|-----|--------|
| Project | `Authorization: Bearer <project_api_key>` → `/v1/*` |
| Admin | `Authorization: Bearer <ADMIN_SECRET>` → `/admin/*` |

`ADMIN_SECRET` ≥ 32 chars. `ENCRYPTION_KEY` = 64 hex chars.

## Must

- Do not hardcode product-specific usage keys in hub defaults — keep generic (`text`, `image`) unless registering via admin
- Never commit `.env` or raw project API keys
- Health: `GET /health` (no auth), `GET /v1/health` (auth)
- ThaiSnapAI talks to this hub — coordinate env with `ThaiSnapAI` docs `AI_HUB.md`
