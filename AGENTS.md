# AGENTS.md

Work this repo **cloud-agent first** on GitHub (Cursor Cloud Agents). Do not assume a local Mac checkout.

**web-4founders** is the corporate landing + interactive guide for 4Founders Studio. Live site: [https://4founders.studio](https://4founders.studio).

## Defaults

| | |
|---|---|
| Default branch | `main` |
| Hosting | Coolify (Docker, `Dockerfile` at repo root) deploys from `main` |
| Merge / deploy | **Never merge a PR or trigger a Coolify deploy** without explicit human OK |

Open a PR to `main`. Leave it unmerged. Asistente Ivrogo will merge.

## Purpose

Static HTML/CSS/JS landing (`public/`) plus a dc-runtime guide (`guia/`), served by a small Express app (`server/`). Forms POST to `/api/contact` and `/api/lead`; the server forwards JSON to n8n webhooks, then Odoo CRM. The browser never talks to n8n or Odoo.

## Stack

- Landing: HTML/CSS/vanilla JS in `public/` (Brand Kit in `docs/brand-kit.html`)
- Guide: dc-runtime in `guia/`, served at `/guia`
- API: Express ESM (`server/index.js`) — static files + `/api/*` + `/health`
- Node 20 Alpine (`Dockerfile`)
- No GitHub Actions CI. No lint or test scripts.

**Do not add** `tweaks-panel`, React, or Babel CDN on the production landing.

## Layout

```
public/          landing, blog, legal, assets, styles.css, app.js
guia/            interactive guide (served at /guia)
server/          Express: static + /api/contact, /api/lead, /api/blog/publish, /health
docs/            brand-kit.html, n8n-workflows.md
Dockerfile       Node 20 Alpine, healthcheck GET /health
```

Register extra static routes **before** the `public/` catch-all in `server/index.js`.

Project skill (read at the start of implementation work): `.cursor/skills/web-4founders-best-practices/SKILL.md`.

Before touching `public/` or `guia/`, also read `.agents/skills/ui-ux-pro-max`, `frontend-design`, and `web-design-guidelines`.

## Commands (Linux / Cloud Agent)

There is no root `package.json`. The server lives in `server/`. There are no `lint` or `test` scripts.

```bash
cd server
npm ci
npm start          # node index.js — http://localhost:3000
```

Dev with reload: `npm run dev` (`node --watch`).

Smoke:

```bash
curl -s http://localhost:3000/health
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/guia/
```

Docker (does not start Coolify):

```bash
docker compose build
```

`docker compose up` expects a root `.env` (see `.env.example`). Do not use Mac-only tools. Do not assume a checkout under `/Users/...`.

## Env

Names only. Never commit values. Copy `.env.example` when present.

- `N8N_CONTACT_WEBHOOK` — contact form → n8n
- `N8N_LEAD_WEBHOOK` — lead magnet → n8n
- `BLOG_PUBLISH_SECRET` — Bearer secret for `POST /api/blog/publish`
- `GITHUB_TOKEN`, `GITHUB_REPO`, `GITHUB_BRANCH` — blog publish commits
- `PORT` — server port (default `3000`)

n8n webhook URLs stay on the server. Never expose them (or Odoo credentials) to the frontend.

## Agent notes

- Prefer small, reviewable PRs. Do not push to `main`.
- Do not change Coolify config, GitHub secrets, workflows, or production unless a human asks.
- If API payloads change, update `docs/n8n-workflows.md`.
- Out of scope without a dedicated issue: auto-download of the guide after the form, embedded Calendly, Next.js migration, first-party database.
- Comments and new code in English; Spanish is OK in docs and PRs. Site copy is Spanish.
