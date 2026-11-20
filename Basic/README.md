# MediaLYT — LYTDesign AI Media Production Studio

Produce YouTube videos and Shorts for the **LYTDesign** channel from real product footage, with AI agents (later) and human approval at every checkpoint.

MediaLYT is a separate application. Its interface is a converted **copy** of BaseLYT V16; the original BaseLYT file is untouched (a read-only backup with its SHA-256 is in `backup/`).

## Run it

```bash
bash scripts/check-host.sh      # read-only check of Node.js, FFmpeg, ffprobe, disk
npm start                       # starts the Media Hub engine
```

Open **http://127.0.0.1:4717/**. The engine serves the interface itself (same origin), listens on loopback only, and stores everything in `data/` (override with `MEDIALYT_DATA_DIR`). No `npm install` is needed — the engine uses only Node.js built-ins (`node:http`, `node:sqlite`, `node:crypto`). Requires Node.js ≥ 22.13 and FFmpeg/ffprobe on `PATH`.

Opening `app/MediaLYT.html` directly as a file works for viewing, but every view will honestly say the engine is not connected.

## Layout

| Path | Purpose |
|---|---|
| `app/MediaLYT.html` | Single-file interface (BaseLYT V16 CSS, theme engine, desktops and components; nine MediaLYT tabs) |
| `engine/server.mjs` | HTTP server, routes, security checks, job worker |
| `engine/db.mjs` | SQLite schema: productions, assets, jobs, approvals, events, brand notes |
| `engine/media.mjs` | Upload streaming + SHA-256, storage layout, FFmpeg/ffprobe detection and probing |
| `engine/workflow.mjs` | Three approval checkpoints, fingerprints, derived production status |
| `engine/agents.mjs` | Six agent role contracts (runners not implemented in V1) |
| `tests/engine.test.mjs` | `npm test` — engine tests (node:test) |
| `tests/ui-verify.cjs`, `tests/ui-e2e.cjs` | Dev-only UI checks (need Playwright + a running engine) |
| `backup/` | Read-only original BaseLYT V16 + checksum |

## What is real in V1

- Engine health (database, storage, FFmpeg/ffprobe detection) shown live; disconnected/loading/error states everywhere.
- Productions: create, edit brief/storyboard/script/captions, archive, delete — persisted in SQLite.
- Media & Brand Library: real uploads with real progress, SHA-256 fingerprints, Range streaming, ffprobe inspection jobs.
- Approvals: creative → production → final. Each stores the fingerprint of exactly what was reviewed; edits, file replacement or on-disk modification invalidate it. Every action is logged.
- Publishing metadata (title, description, tags, privacy, schedule, kids, synthetic-content disclosure) — part of the final fingerprint.

## Not implemented yet (shown as such in the UI)

AI agent runners, AI/TTS provider adapters, FFmpeg edit/render pipeline, YouTube upload. Agent "Run" actions are disabled and the engine returns HTTP 501 instead of simulating work.

## Security

Loopback bind; Host-header check (DNS rebinding); writes need same origin + `X-MediaLYT` header; strict CSP; no credentials in the page, browser storage or repo — future provider keys go in the engine environment only.
