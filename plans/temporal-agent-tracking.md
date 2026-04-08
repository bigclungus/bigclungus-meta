# Temporal Agent Tracking — Implementation Plan

**Created:** 2026-04-08
**Requested by:** koole__
**Status:** Planned, not started

## System Context

- Existing `worker.py` uses `namespace="default"` (implicit), task queue `"listings-queue"`, Temporal at `localhost:7233`
- `subagent-start.ts` / `subagent-stop.ts` currently write to `tasks.db` (at `/home/clungus/work/bigclungus-meta/tasks.db`) and POST to clunger `/api/agents/spawn` and `/api/agents/:id/complete`
- Clunger reads `agents.db` at `/mnt/data/data/agents.db` for spawn/complete data and `tasks.db` for task list
- The Temporal HTTP API is live at `:8233`; only `default` and `temporal-system` namespaces exist so far

---

## Phase 1: Namespace and Worker Setup

### 1.1 Create the "tasks" namespace

Use the Temporal HTTP API once, idempotently:
```
POST http://127.0.0.1:8233/api/v1/namespaces
{
  "namespace": "tasks",
  "workflowExecutionRetentionPeriod": "720h"  // 30 days
}
```

This should be done as a one-time bootstrap step before the worker starts. The tasks worker should attempt this at startup and swallow `409 AlreadyExists`.

### 1.2 Separate worker process

Create `/mnt/data/temporal-workflows/tasks_worker.py`. This is intentionally separate from `worker.py` to avoid namespace cross-contamination. Pattern mirrors `worker.py`:

```python
client = await Client.connect(TEMPORAL_HOST, namespace="tasks")
worker = Worker(
    client,
    task_queue="agent-tasks-queue",
    workflows=[AgentTaskWorkflow],
    activities=[create_task_record, run_agent, finalize_task],
)
```

Key differences from `worker.py`:
- `namespace="tasks"` on the Client connect call
- `task_queue="agent-tasks-queue"` (separate from `"listings-queue"`)
- No cron bootstrapping — this worker is purely reactive (workflows are started by hooks)

Wire up a systemd service at `/mnt/data/temporal-workflows/systemd/tasks-worker.service` mirroring the existing `temporal-worker.service` pattern.

---

## Phase 2: Workflow — `AgentTaskWorkflow`

**File:** `/mnt/data/temporal-workflows/workflows/agent_task_wf.py`

```python
@workflow.defn
class AgentTaskWorkflow:
    def __init__(self) -> None:
        self._status = "running"
        self._metadata: dict = {}
        self._cancel_requested = False

    @workflow.run
    async def run(self, input: dict) -> dict:
        # Fields: task_id, agent_type, model, prompt, metadata
        await workflow.execute_activity(
            create_task_record,
            args=[input],
            start_to_close_timeout=timedelta(seconds=30),
            retry_policy=RetryPolicy(maximum_attempts=3),
        )
        result = await workflow.execute_activity(
            run_agent,
            args=[input],
            start_to_close_timeout=timedelta(minutes=input.get("start_to_close_timeout_minutes", 45)),
            heartbeat_timeout=timedelta(minutes=2),
            retry_policy=RetryPolicy(maximum_attempts=1),  # no retry on agent activity
        )
        await workflow.execute_activity(
            finalize_task,
            args=[input, result],
            start_to_close_timeout=timedelta(seconds=60),
            retry_policy=RetryPolicy(maximum_attempts=5),
        )
        return result

    @workflow.signal
    async def cancel(self) -> None:
        self._cancel_requested = True
        self._status = "cancelled"

    @workflow.signal
    async def add_metadata(self, kv: dict) -> None:
        self._metadata.update(kv)

    @workflow.query
    def get_status(self) -> dict:
        return {"status": self._status, "metadata": self._metadata}
```

Input schema (dict keys):
- `task_id` — matches the ID generated in `subagent-start.ts` (e.g. `task-20260408-123456-abcd1234`)
- `agent_type` — `"claude" | "gemini" | "gpt" | "custom"`
- `model` — e.g. `"claude-sonnet-4-6"`
- `prompt` — the originating prompt/title (from pending file context)
- `metadata` — freeform dict (discord_user, session_id, discord_message_id, run_in_background, isolation)
- `start_to_close_timeout_minutes` — optional int, defaults to `45`. The workflow passes `timedelta(minutes=input.get("start_to_close_timeout_minutes", 45))` to `run_agent`'s `start_to_close_timeout`. Callers needing a longer-running agent can set this per-invocation.

The workflow does NOT use `workflow.wait_condition` on cancel — the cancel signal sets a flag, and `run_agent` checks `activity.is_cancelled()` during heartbeat. This keeps the workflow determinism clean.

---

## Phase 3: Activities

**File:** `/mnt/data/temporal-workflows/activities/agent_task_act.py`

### `create_task_record`

- Writes a row to `tasks.db` (same schema as current `subagent-start.ts` INSERT: `id, title, status, created_at, updated_at, data`)
- This is redundant with the hook in shadow mode but becomes authoritative in Phase 2
- Use `sqlite3` directly (no Bun dependency); the DB is at `/home/clungus/work/bigclungus-meta/tasks.db`
- Also write a spawn row to `/mnt/data/data/agents.db` mirroring what clunger's `restHandleAgentSpawn` does (columns: `id, task_id, session_id, started_at, status, model, description, trace_id`)

### `run_agent`

Agent-type dispatch pattern:
```python
AGENT_ADAPTERS = {
    "claude": ["claude", "-p"],
    "gemini": ["gemini", "--prompt"],
    "gpt": ["gpt", "-p"],
    "custom": [],
}

@activity.defn
async def run_agent(input: dict) -> dict:
    adapter = AGENT_ADAPTERS.get(input["agent_type"], AGENT_ADAPTERS["claude"])
    cmd = adapter + [input["prompt"]]
    proc = await asyncio.create_subprocess_exec(
        *cmd,
        stdout=asyncio.subprocess.PIPE,
        stderr=asyncio.subprocess.PIPE,
    )
    # Heartbeat loop: 30s interval, cancel check
    while proc.returncode is None:
        await asyncio.sleep(30)
        activity.heartbeat({"pid": proc.pid})
        if activity.is_cancelled():
            proc.terminate()
            break
    try:
        stdout, stderr = await asyncio.wait_for(proc.communicate(), timeout=10.0)
    except asyncio.TimeoutError:
        proc.kill()
        stdout, stderr = await proc.communicate()
    return parse_agent_output(stdout.decode(), input["agent_type"])
```

Output parsing for token/cost extraction:
- Claude: parse the `usage:` JSON block from `claude -p` output (input_tokens, output_tokens, cache_read_input_tokens, cost_usd)
- Gemini/GPT: similar output parsing for their CLIs; stub out and return zeros for initial implementation

### `finalize_task`

Two writes, both must succeed (activity retries handle transient failures):

1. **`tasks.db` update** — mirrors `subagent-stop.ts` UPDATE: set `status="done"`, update `data` blob with `finished_at`, append done log entry
2. **`agents.db` update** — mirrors clunger's `restHandleAgentComplete`: set `status="completed"`, `completed_at`, `input_tokens`, `output_tokens`, `cost_usd`, `duration_ms`

> **Idempotency note:** activity may retry on transient failure; all DB writes must be idempotent UPSERTs. Use `INSERT OR REPLACE` for new rows and `INSERT OR IGNORE` where only missing rows should be inserted. Never use bare `INSERT` in this activity.

Both writes use sqlite3 with WAL mode. The activity result is also returned to the workflow and stored in Temporal history (free audit log).

---

## Phase 4: Hook Integration (Shadow Mode)

**Files to modify:** `/mnt/data/scripts/hooks/subagent-start.ts` and `subagent-stop.ts`

### `subagent-start.ts` changes

After the existing DB write and clunger POST, add a non-blocking Temporal start:

```typescript
// Shadow mode: also start AgentTaskWorkflow in "tasks" namespace
async function startTemporalWorkflow(taskId: string, input: TemporalInput): Promise<void> {
  const workflowId = `agent-task-${taskId}`;
  const body = JSON.stringify({
    workflow_type: "AgentTaskWorkflow",
    task_queue: "agent-tasks-queue",
    workflow_id: workflowId,
    input: [input],
  });
  await fetch(
    "http://127.0.0.1:8233/api/v1/namespaces/tasks/workflows",
    { method: "POST", headers: { "Content-Type": "application/json" }, body }
  );
}
```

Call with `agent_type` derived from `model` field in the pending file context (e.g. `claude-*` → `"claude"`, `gemini-*` → `"gemini"`). Default to `"claude"`.

Store `workflowId` (`agent-task-<taskId>`) in `/tmp/bc-agents/<agentId>.json` — the same state file already used by hooks for IPC. Concretely: `subagent-start.ts` writes `workflowId` into that JSON alongside `task_id`; `subagent-stop.ts` reads it back to know which workflow to signal for completion.

### `subagent-stop.ts` changes

After existing DB update and clunger POST, send a signal to the workflow:

```typescript
// Shadow mode: signal the running Temporal workflow to mark complete
async function signalTemporalWorkflow(workflowId: string, lastMsg: string): Promise<void> {
  await fetch(
    `http://127.0.0.1:8233/api/v1/namespaces/tasks/workflows/${workflowId}/signal`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        signal_name: "add_metadata",
        input: [{ last_message: lastMsg.slice(0, 500), completed_at: new Date().toISOString() }],
      }),
    }
  );
}
```

Note: in Phase 1 (shadow mode), `subagent-stop.ts` does NOT send the `cancel` signal — the workflow runs `run_agent` independently. The signal is metadata-only. The workflow's `finalize_task` will write the real output when the subprocess finishes. This keeps the two systems fully decoupled.

**Error handling:** all Temporal calls are wrapped in `try/catch` with `process.stderr.write` on failure. A Temporal failure never propagates to the hook exit code — the old system remains the fallback.

---

## Phase 5: Clunger Integration

**File:** `/home/clungus/work/clunger/src/index.ts`

### New endpoint: `GET /api/tasks?temporal=true`

Shadow endpoint — same path as the existing tasks list but with a `temporal=true` query param to opt into the Temporal-backed response. This avoids adding a new route and keeps the UI toggle simple.

```typescript
// GET /api/tasks?temporal=true
if (pathname === "/api/tasks" && req.method === "GET") {
  if (!restIsAuthed(req)) { jsonResponse(res, { error: "Forbidden" }, 403); return; }
  if (url.searchParams.get("temporal") === "true") {
    await restServeTemporalTasks(res, url.searchParams);
  } else {
    await restServeTasks(res, url.searchParams);
  }
  return;
}
```

Implementation of `restServeTemporalTasks`:
```typescript
async function restServeTemporalTasks(res, params) {
  const status = params.get("status") ?? "running";
  // Map to Temporal execution status filter
  // running → OPEN, done → CLOSED, etc.
  const query = status === "running" ? "ExecutionStatus='Running'" : "";
  const url = `http://127.0.0.1:8233/api/v1/namespaces/tasks/workflows?query=${encodeURIComponent(query)}`;
  const resp = await fetch(url);
  const data = await resp.json();
  // Normalize Temporal workflow execution shape to the same Task shape clunger already serves
  const tasks = (data.executions ?? []).map(normalizeTemporalExecution);
  jsonResponse(res, { tasks, source: "temporal" });
}
```

`normalizeTemporalExecution` maps Temporal's `WorkflowExecutionInfo` fields to the existing `Task` shape used in `listTasks`: extract `workflowId` as `id`, `startTime` as `startedAt`, status as `status`. Do NOT pull input args from workflow history — that requires a per-workflow API call and won't scale. Instead, store `task_id`, `agent_type`, and `model` as Temporal search attributes at workflow start time:

```python
workflow.upsert_typed_search_attributes(
    TypedSearchAttributes([
        SearchAttributePair(SearchAttributeKey.for_keyword("task_id"), input["task_id"]),
        SearchAttributePair(SearchAttributeKey.for_keyword("agent_type"), input["agent_type"]),
        SearchAttributePair(SearchAttributeKey.for_keyword("model"), input["model"]),
    ])
)
```

Clunger can then read these directly from the list API response (`execution.searchAttributes`) without extra round-trips.

### Terminal page toggle

The terminal graph page (`/mnt/data/terminal/graph.html`) currently has a tasks panel. Add a `data-source` toggle button in the tasks section:
```html
<button id="task-source-toggle" onclick="toggleTaskSource()">source: legacy</button>
```

JS logic: on toggle, flip between fetching `/api/tasks` (legacy) and `/api/tasks?temporal=true` (Temporal). Store preference in `localStorage`. Both responses use the same normalized shape so no rendering changes needed.

---

## Phase 6: Rollout Sequence

### Phase 1 — Shadow mode (current target)

- `tasks` namespace created
- `tasks_worker.py` running as systemd service
- `subagent-start.ts` and `subagent-stop.ts` POST to Temporal in addition to existing paths
- Neither system is authoritative; Temporal failures are silent
- Clunger gets `GET /api/tasks?temporal=true` endpoint (internal only, not surfaced in UI yet)
- Validation: compare `/api/tasks` vs `/api/tasks?temporal=true` counts daily; alert on >10% divergence

### Phase 2 — Temporal authoritative for new tasks

- `subagent-start.ts` starts Temporal workflow FIRST; falls back to sqlite only on error
- `finalize_task` activity becomes the canonical write path for `agents.db`
- `subagent-stop.ts` only signals Temporal; the workflow's `finalize_task` handles both DB writes
- `subagent-start.ts` still writes `tasks.db` for backward compat with `restServeTasks`
- Terminal page toggle surfaced in UI; default switches to "temporal"

Gating criteria for Phase 2: 7 consecutive days of shadow mode with <2% divergence on task counts and no Temporal worker crashes.

### Phase 3 — Drop sqlite task writing in hooks

- Remove sqlite INSERT from `subagent-start.ts`
- Remove sqlite UPDATE from `subagent-stop.ts`
- `restServeTasks` in clunger reads from Temporal (proxied) instead of `tasks.db`
- `tasks.db` kept as read-only historical archive
- `check_open_tasks` sweeper activity updated to query Temporal HTTP API instead of scanning JSON files

---

## Known Issues Before Phase 2

These are confirmed blockers that must be resolved before cutting over to Phase 2 (Temporal authoritative):

1. **`proc.communicate()` hang after cancel** — if the agent subprocess ignores `SIGTERM`, `proc.communicate()` blocks forever and the activity never returns, preventing Temporal from marking the workflow failed. Fixed in this plan with `asyncio.wait_for(..., timeout=10.0)` + `proc.kill()` fallback, but needs a real integration test with a stubborn subprocess before Phase 2.

2. **`finalize_task` idempotency** — activity retries on transient DB failure will re-run the writes. Without `INSERT OR REPLACE` / `INSERT OR IGNORE`, a retry produces duplicate rows or constraint errors, leaving the task in a broken state. All DB writes in `finalize_task` must be audited and converted to idempotent UPSERTs before Phase 2 makes this the canonical write path.

---

## File Paths Summary

New files to create:
- `/mnt/data/temporal-workflows/workflows/agent_task_wf.py`
- `/mnt/data/temporal-workflows/activities/agent_task_act.py`
- `/mnt/data/temporal-workflows/tasks_worker.py`
- `/mnt/data/temporal-workflows/systemd/tasks-worker.service`

Files to modify:
- `/mnt/data/scripts/hooks/subagent-start.ts` — add Temporal shadow POST
- `/mnt/data/scripts/hooks/subagent-stop.ts` — add Temporal shadow signal
- `/home/clungus/work/clunger/src/index.ts` — add `/api/tasks/temporal` route
- `/mnt/data/terminal/graph.html` — add task source toggle
