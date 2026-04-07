# Agent Metadata Unification Plan

## Goal

Consolidate task tracking, subagent lifecycle data, and cost information into a single SQLite database (`agents.db`). The `/tasks` page gains inline cost display. The `/costs` page is retired.

---

## Current State

| Component | Backing Store |
|---|---|
| `/tasks` | `bigclungus-meta/tasks/*.json` flat files |
| Terminal subagent grid | `/tmp/*.jsonl` output files scanned live |
| `/costs` | `token-usage.db` SQLite (separate DB) |

Pain points:
- Three separate data sources with no foreign keys between them
- Cost is not visible in the task view
- Subagent discovery is a filesystem glob — fragile and stateless
- Task JSONs are the only durable backup; no indexing or query capability

---

## Target State

- **One DB:** `agents.db` at `/mnt/data/data/agents.db`
- **`/tasks`:** reads from `agents.db`, shows per-task cost and session total in header
- **Terminal subagent grid:** unchanged — still shows live agent status (already good)
- **`/costs`:** retired; data folded into `/tasks`
- **`token-usage.db`:** retired after migration

---

## Schema

```sql
-- One row per Claude agent invocation (subagent or main session)
CREATE TABLE agents (
    id              TEXT PRIMARY KEY,          -- matches task_id or a generated UUID
    task_id         TEXT,                      -- FK → tasks.id (nullable for orphaned agents)
    session_id      TEXT,                      -- Claude session/JSONL file identifier
    started_at      INTEGER,                   -- unix timestamp
    completed_at    INTEGER,
    status          TEXT DEFAULT 'in_progress',-- in_progress | done | failed | stale
    input_tokens    INTEGER DEFAULT 0,
    output_tokens   INTEGER DEFAULT 0,
    cost_usd        REAL DEFAULT 0.0,
    model           TEXT,
    output_file     TEXT                       -- /tmp path for SSE tail (nullable)
);

-- Replaces bigclungus-meta/tasks/*.json
CREATE TABLE tasks (
    id              TEXT PRIMARY KEY,          -- task-YYYYMMDD-HHMMSS-<hash>
    title           TEXT NOT NULL,
    description     TEXT,
    status          TEXT DEFAULT 'open',       -- open | in_progress | done | failed
    created_at      INTEGER,
    updated_at      INTEGER,
    source          TEXT                       -- discord | nightowl | manual
);

-- Replaces task event arrays inside JSON files
CREATE TABLE task_events (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    task_id         TEXT NOT NULL,
    event_type      TEXT NOT NULL,             -- milestone | user_feedback | blocked | done | failed
    message         TEXT,
    created_at      INTEGER
);

CREATE INDEX idx_agents_task_id   ON agents(task_id);
CREATE INDEX idx_agents_status    ON agents(status);
CREATE INDEX idx_task_events_task ON task_events(task_id);
```

---

## Milestones

### 1 — Migration script
**File:** `scripts/migrate_to_agents_db.py`

- Create `agents.db` with schema above
- Read all `bigclungus-meta/tasks/*.json` → insert into `tasks` + `task_events`
- Read `token-usage.db` rows → insert into `agents`, joining by session_id/task_id where possible
- Unmatched token rows land in `agents` with `task_id = NULL`
- Run once; idempotent (skip already-inserted IDs)

### 2 — Instrument agent spawn / complete
**Files:** `scripts/launch-claude.py`, `scripts/log_task_event.py`

- On spawn: `INSERT INTO agents (id, task_id, session_id, started_at, status, output_file)`
- On complete: `UPDATE agents SET completed_at, status, input_tokens, output_tokens, cost_usd, model`
- Remove writes to `token-usage.db`; keep writing task JSON files until phase 5 (safety net)

### 3 — Rewrite clunger API endpoints
**File:** `clunger/src/services/tasks.ts`

- `GET /api/tasks` — query `tasks` JOIN `agents` for cost aggregation; return task list with `cost_usd` field
- `GET /api/tasks/:id` — return task detail + events + associated agent runs
- `GET /api/subagents` — query `agents WHERE status = 'in_progress'` (replaces `/tmp` glob)
- Keep SSE endpoint (`/api/subagents/:id/stream`) wired to `output_file` path — no change here

### 4 — Add cost display to `/tasks`
**File:** `clunger/src/public/tasks.html` (or equivalent frontend)

- Per-task row: show `$0.0032` next to task status badge
- Session total: sticky header or footer showing sum of all cost_usd for current view
- No new page — cost lives inside tasks view inline

### 5 — Delete legacy data stores
After phase 3+4 are verified in prod:

- `rm bigclungus-meta/tasks/*.json` (keep directory, update `.gitignore`)
- Remove `log_task_event.py` filesystem write path (keep CLI interface, rewrite to write DB)
- Retire `token-usage.db` and drop `/costs` clunger route
- Remove `/tmp` JSONL glob code from subagent discovery

### 6 — Stale agent detection (optional)
On `claude-bot.service` restart:

- `UPDATE agents SET status = 'stale' WHERE status = 'in_progress' AND started_at < :cutoff`
- Terminal grid can show stale rows grayed out
- Cutoff = restart timestamp; log a milestone event per staled task

---

## What Dies

| Item | Fate |
|---|---|
| `bigclungus-meta/tasks/*.json` | Deleted after phase 5 |
| `token-usage.db` | Deleted after phase 5 |
| `/tmp` JSONL glob for subagent discovery | Removed in phase 3 |
| `/costs` clunger page | Route deleted in phase 5 |

## What Stays

| Item | Notes |
|---|---|
| SSE stream for live agent output | Still tails `output_file`; path now comes from DB |
| Task IDs (`task-YYYYMMDD-...` format) | Same IDs, now PKs in `tasks` table |
| `log_task_event.py` CLI interface | Rewritten internally to write DB instead of JSON |
| `tasks.html` frontend | Extended with cost columns; no structural rewrite |

---

## Risks

| Risk | Mitigation |
|---|---|
| Join gaps between old token rows and task logs | Orphaned agents land with `task_id = NULL`; surfaced in migration report |
| Ghost `in_progress` rows after crash (no phase 6) | Phase 6 fixes this; skip only if restarts are rare |
| Flat JSONs as only durable backup | Keep JSON writes alive through phase 4; only delete after DB is confirmed healthy |
| Schema mismatch between agent spawner and clunger | Use a shared schema file or migration version table |

---

## Open Questions

- Does `log_task_event.py` need to stay a standalone script (for Temporal workers), or can it become a thin HTTP call to clunger?
- Should `agents.db` live in `/mnt/data/data/` (alongside `discord-history.db`) or in `bigclungus-meta/`?
- Retention policy: keep all agent rows forever, or purge after N days?
