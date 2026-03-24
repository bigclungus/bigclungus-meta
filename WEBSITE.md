# BigClungus Website — High Level Design

## Architecture Overview

All public traffic enters through a single Cloudflare named tunnel (`685863fc-3783-4e49-bb8d-6a405c524706`), which routes requests by hostname to local Python services running on the Ubuntu VM. There is no reverse proxy tier between Cloudflare and the application servers — Cloudflare terminates TLS and the tunnel daemon (`cloudflared`) forwards plaintext HTTP to each service's local port.

```
Internet
   │  HTTPS
   ▼
Cloudflare Edge (TLS termination)
   │  cloudflared tunnel
   ▼
Ubuntu VM
   ├─ :8080  serve.py         (clung.us)
   ├─ :7682  terminal/server.py  (terminal.clung.us)
   ├─ :8234  temporal/proxy.py   (temporal.clung.us → :8233 Temporal dev server)
   ├─ :8082  1998/             (1998.clung.us, static)
   └─ :8084  (cost.clung.us, separate service)
```

Unmatched hostnames fall through to a Cloudflare `http_status:404` rule.

---

## Domains & Routing

| Subdomain | Local Port | Service | Auth |
|---|---|---|---|
| `clung.us` | 8080 | Static site (`serve.py`) | None |
| `terminal.clung.us` | 7682 | Terminal server (`server.py`) | GitHub OAuth |
| `temporal.clung.us` | 8234 | Temporal proxy (`proxy.py`) | GitHub OAuth |
| `1998.clung.us` | 8082 | 1998 retro static site | None |
| `cost.clung.us` | 8084 | Cost tracker (separate service) | Unknown |

Source: `/home/clungus/.cloudflared/config.yml`

---

## Pages & Endpoints

### clung.us (Static Site)

Served by `/mnt/data/hello-world/serve.py` — a subclass of Python's `SimpleHTTPRequestHandler` on port 8080.

**Server behavior:**
- Extensionless URLs (e.g. `/deaths`) are rewritten to `deaths.html` if the file exists.
- Custom 404 page served from `404.html` if present.
- Access logs are suppressed.

**Pages:**

| URL | File | Purpose | Auth |
|---|---|---|---|
| `/` | `index.html` | Landing page — BigClungus avatar, bio, nav | None |
| `/deaths` | `deaths.html` | "The Deaths of Centronias" tracker | None |
| `/changelog` | `changelog.html` | Site changelog (file assumed, not read) | None |

**Navigation links on every page:**
- `hello` — `/`
- `changelog` — `/changelog`
- `deaths` — `/deaths`
- `github` — external: `https://github.com/bigclungus`
- `project board` — external: GitHub Projects board
- `1998` — external: `https://1998.clung.us`
- `terminal` — `https://terminal.clung.us` (marked locked)
- `temporal` — `https://temporal.clung.us` (marked locked)

**deaths.html specifics:**
- Displays a large death count and one coffin emoji per death.
- Death count is a hardcoded JS variable (`DEATH_COUNT = 2` as of last read).
- Subtitle: "The Deaths of Centronias — a faithful record / All deaths attributed to Clungus."

**index.html Easter egg:**
- Clicking the robot avatar 5 times spawns a spider emoji that chases the mouse cursor; catching it with the cursor destroys it.

---

### terminal.clung.us

Served by `/mnt/data/terminal/server.py` (aiohttp) on port 7682.

**Purpose:** Live terminal viewer showing BigClungus's active `screen` session (`/tmp/screenlog.txt`), plus a side panel with GitHub Project tasks and subagent task status.

**Auth:** GitHub OAuth (see Auth section). All routes except `/login`, `/auth/github`, `/auth/callback` require a valid `tauth_github` cookie.

**Routes:**

| Method | Path | Description |
|---|---|---|
| GET | `/` | Main terminal UI (xterm.js + agent panel) |
| GET | `/login` | Login page with GitHub OAuth button |
| POST | `/login` | Login page (same handler) |
| GET | `/auth/github` | Redirects to GitHub OAuth authorization |
| GET | `/auth/callback` | GitHub OAuth callback; sets `tauth_github` cookie |
| GET | `/ws` | WebSocket: streams `screenlog.txt` to xterm.js client |
| GET | `/health` | JSON health payload: CPU, RAM, disk, swap, service statuses, uptime, cost data |
| GET | `/graph` | Knowledge graph viewer page |
| GET | `/graph-data` | JSON: entity/relationship data from FalkorDB (Graphiti) |
| GET | `/ingestion-status` | JSON: Graphiti ingestion status |
| GET | `/tasks` | JSON: local subagent task list from `/tmp/claude-*/tasks` |
| GET | `/task-output/{agentId}` | JSON: output for a specific subagent task |
| POST | `/meta/{agentId}` | Write metadata for a subagent task |
| GET | `/github-tasks` | JSON: open issues from BigClungus/bigclungus-meta GitHub Project |
| POST | `/restart-bot` | Restarts `claude-bot.service` via systemctl |
| GET | `/cost-data` | JSON: parsed cost/token usage data |
| GET | `/system-status` | JSON: systemd service statuses |
| GET | `/topology` | System topology visualization page |
| GET | `/gamecube-sounds.js` | Serves Gamecube sound effects JS file |
| GET | `/edit-claude-md` | Web editor for the CLAUDE.md config file |
| POST | `/edit-claude-md` | Saves edits to CLAUDE.md config |

**Terminal UI layout:**
- Header bar: session name, connection status, links to clung.us / knowledge graph / system / claude.md editor, restart button.
- Health bar: live CPU, RAM, disk, swap gauges; service status dots for `cloudflared` and `terminal-server`; uptime display.
- Main area: xterm.js terminal (70% width) + agent side panel (30% width).
- Agent panel: GitHub Project issues (top half) + subagent task list (bottom half).

---

### temporal.clung.us

Served by `/mnt/data/temporal/proxy.py` (aiohttp) on port 8234.

**Purpose:** Authenticated reverse proxy in front of the Temporal dev server running on `localhost:8233`. Exposes the Temporal web UI and API to the public internet behind GitHub OAuth.

**Auth:** GitHub OAuth (see Auth section). All routes except `/login`, `/auth/github`, `/auth/github/callback` require a valid `tauth_github` cookie.

**Routes:**

| Method | Path | Description |
|---|---|---|
| GET | `/login` | Login page with GitHub OAuth button |
| GET | `/auth/github` | Redirects to GitHub OAuth authorization |
| GET | `/auth/github/callback` | OAuth callback; sets `tauth_github` cookie |
| `*` | `/{path:.*}` | Catch-all: proxies request to `http://localhost:8233` |

**Proxy behavior:**
- Preserves raw path (percent-encoding intact, important for Temporal workflow IDs with `/`).
- Strips hop-by-hop headers in both directions.
- Adds `X-Forwarded-For` and `X-Forwarded-Host`.
- 60-second request timeout; returns 502 on connection failure, 504 on timeout.
- Session cookie (`tauth_github`) is valid for 24 hours.

---

### 1998.clung.us

Served statically from `/mnt/data/1998/` on port 8082 (separate service, not `serve.py`).

**Purpose:** Parody late-1990s personal homepage aesthetic. Features: tiled starfield SVG background, Comic Sans, WordArt-style rainbow heading, blinking/marquee/spinning text animations, "UNDER CONSTRUCTION" flashing banner, hit counter widget, rainbow animated dividers, and sidebar/content table layout. No functional backend — purely decorative static HTML.

---

## Auth

Both `terminal.clung.us` and `temporal.clung.us` use the same GitHub OAuth flow, implemented independently in each service's Python file.

**Flow:**
1. Unauthenticated request hits auth middleware → redirect to `/login`.
2. User clicks "Sign in with GitHub" → GET `/auth/github` → redirect to `https://github.com/login/oauth/authorize` with `client_id`, `scope=read:user`, random `state` token (stored in `gh_oauth_state` cookie, 10-minute TTL).
3. GitHub redirects to the callback URL with `code` and `state`.
4. Callback handler validates CSRF `state`, POSTs to `https://github.com/login/oauth/access_token` to exchange the code for an access token, then GETs `https://api.github.com/user` to resolve the GitHub username.
5. If the username is in `GITHUB_ALLOWED_USERS` (or the list is empty), sets a `tauth_github` cookie containing the username (HttpOnly, SameSite=Lax, 24-hour TTL) and redirects to `/`.
6. Subsequent requests: middleware reads `tauth_github` cookie and re-checks against the allowed list.

**Config:** `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`, and `GITHUB_ALLOWED_USERS` are loaded from environment variables. See `.env` files loaded by systemd (`EnvironmentFile=`) — do not hardcode secrets here. The terminal server's `.env` is at `/mnt/data/terminal/.env`.

**Note:** The terminal server's OAuth callback is at `/auth/callback`; the temporal proxy uses `/auth/github/callback`. These are separate OAuth app registrations.

---

## Services

All managed via `systemctl --user` as the `clungus` user.

| Service | Binary / Script | Port | Description |
|---|---|---|---|
| `website.service` | `serve.py` | 8080 | Static site for clung.us |
| `terminal-server.service` | `terminal/server.py` | 7682 | Live terminal viewer + admin UI |
| `temporal-proxy.service` | `temporal/proxy.py` | 8234 | Auth proxy for Temporal dev server |
| `temporal.service` | Temporal CLI | 8233 | Temporal dev server (internal only) |
| `temporal-worker.service` | Temporal worker | — | Worker for `listings-queue` task queue |
| `1998.service` | Static server | 8082 | 1998 retro site |
| `cloudflared.service` | `cloudflared` | — | Cloudflare tunnel daemon |
| `claude-bot.service` | Claude Code bot | — | BigClungus Discord bot |

**Docker services** (managed separately via docker-compose, root at `/mnt/data/docker/`):
- FalkorDB / Redis on port 6379 — backing store for Graphiti memory.

**Post-restart note:** After any system restart, run:
```
docker exec docker-falkordb-1 redis-cli CONFIG SET stop-writes-on-bgsave-error no
```
to re-enable FalkorDB writes.
