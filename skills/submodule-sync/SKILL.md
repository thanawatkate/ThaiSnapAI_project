---
name: submodule-sync
description: >-
  Sync git submodules for this monorepo via script/sync.sh. Use when updating
  ThaiSnapAI or aihubproject submodules, fixing submodule URLs, or after
  .gitmodules changes.
---

# Submodule Sync

## Run

```bash
cd script && ./sync.sh
```

From WSL/Debian preferred. Needs `git` + network auth to GitHub.

## Rules

1. Source of truth for paths/URLs is root `.gitmodules` only.
2. Do not delete dirty submodule work — script backs up to `.sync-backup/`.
3. If mid-rebase/merge in a submodule, stop and fix that submodule first.
4. Prefer HTTPS via `gh` auth when SSH keys missing (Debian WSL).
5. Do not force-push submodule remotes. Do not edit `.git/config` submodule URLs by hand — re-run sync.
6. After sync, check `git submodule status` at repo root.

## Layout

| Path | Role |
|------|------|
| `ThaiSnapAI/` | App (video / Streamlit / API) |
| `aihubproject/` | AI Hub (Node / MySQL) |
| `script/sync.sh` | Register, clone, pull default branch |
