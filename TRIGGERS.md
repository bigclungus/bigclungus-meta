# Discord Trigger Handling Instructions

This file contains the full handling instructions for Discord trigger patterns. When a `[$trigger]` pattern appears in a Discord message, look it up here.

`[giga]` is the exception — it is documented inline in `/mnt/data/CLAUDE.md` and not here.

---

## `[congress] <topic>`

**SUSPENSION CHECK:** Before firing, check if `/home/clungus/work/bigclungus-meta/CONGRESS_SUSPENDED.md` exists. If it does, reply to Discord: "⚖️ Congress is suspended pending process revision (initiated by centronias). No new sessions until the revised process is ratified." Do NOT fire the workflow.

Fire a `CongressWorkflow` in Temporal:
```python
client = await Client.connect('localhost:7233')
await client.start_workflow(
    'CongressWorkflow',
    {'topic': '<topic>', 'chat_id': '<chat_id>', 'message_id': '<message_id>', 'discord_user': '<user>'},
    id=f'congress-{int(time.time())}',
    task_queue='listings-queue',
    id_reuse_policy=WorkflowIDReusePolicy.ALLOW_DUPLICATE,
)
```
**IMPORTANT:** Always pass `message_id` and `discord_user` (the username from the Discord message tag). These are required for Nemesis to activate when a stakeholder fires congress.

Reply with: "⚖️ congress is in session — verdict will land here when they've deliberated"

---

## `[meme-congress] <topic>`

Same as `[congress]` but fires CongressWorkflow with `mode: 'meme'`. Differences from standard congress:
- No suspension check — meme sessions are always allowed
- Ibrahim's ABORT/REFRAME check is skipped (no chairman veto)
- No task files generated after verdict
- Verdict tracking row has `requires_ack=false` and `mode='meme'`
- Report includes "🃏 meme session — no action items" footer
- No self-inject for implementation — purely for fun

Fire a `CongressWorkflow` in Temporal:
```python
client = await Client.connect('localhost:7233')
await client.start_workflow(
    'CongressWorkflow',
    {'topic': '<topic>', 'chat_id': '<chat_id>', 'message_id': '<message_id>', 'discord_user': '<user>', 'mode': 'meme'},
    id=f'congress-{int(time.time())}',
    task_queue='listings-queue',
    id_reuse_policy=WorkflowIDReusePolicy.ALLOW_DUPLICATE,
)
```

Reply with: "🃏 meme congress is in session — pure chaos, no consequences"

---

## `[simplify]`

An hourly automated code review trigger from SimplifyCronWorkflow. Its job is to scan recent changes across the main codebases and apply cleanup fixes (dead code, duplication, style consistency, minor bugs).

When you receive `[simplify]`: **spawn a background agent** (do NOT block the main thread) to do the following:
0. **Secret scan** — run `bash /mnt/data/scripts/check-secrets.sh --recent 5` in each repo (`/mnt/data/hello-world` and `/mnt/data/temporal-workflows`). If any secrets are detected in recent commits, immediately alert in Discord and open a GitHub issue.
1. **Get recent diffs** — run `git -C /mnt/data/hello-world log --oneline -5` and `git -C /mnt/data/temporal-workflows log --oneline -5` to see what changed recently
2. **Review for issues** — look at the diffs for: dead code, duplicate logic, hardcoded values that should use constants, obvious bugs, style inconsistencies, redundant imports
3. **Apply fixes** — make targeted edits, commit with message `simplify: <brief description>`, and push to GitHub
4. **Restart affected services** if you changed files in hello-world (`systemctl --user restart website.service`) or temporal-workflows (`systemctl --user restart temporal-worker.service`)
5. **Do nothing and stay silent** if there's nothing worth fixing — don't invent busywork

Constraints:
- No architectural changes, no new features — only cleanup and minor fixes
- Do not post to Discord unless a service restart was needed or a real bug was fixed
- Only touch `/mnt/data/hello-world/` and `/mnt/data/temporal-workflows/`

---

## `[heartbeat]`

A 15-minute watchdog pulse from the HeartbeatWorkflow. Its job is to check if anything is on fire and act if so — not to manufacture work.

### Congress threshold (read this first)

**Minor/operational findings — fix directly, no Congress:**
- Config fixes, performance tweaks, reliability improvements, small code changes, break/fix issues
- Rule of thumb: if it can be described in one sentence and reverted in under 10 lines, it's minor
- Fix immediately or queue to NightOwl — do not defer without action

**Major findings — Congress required (when Congress is active):**
- New features, new systems, significant refactors, architectural changes
- If in doubt: if it takes more than one sentence to describe or more than 10 lines to revert, go to Congress

When you receive `[heartbeat]`: **spawn a background agent** to do the following:
1. **Check for stale tasks** — run `bash /mnt/data/scripts/hooks/watchdog-stale-tasks.sh`. If stale tasks found, investigate and resolve or mark failed.
2. **Check GitHub issues** — `gh issue list --repo bigclungus/bigclungus-meta --state open --limit 5`. If there's a clear, small actionable issue not already in progress, work on it.
3. **Check services** — `systemctl --user list-units --type=service --state=failed`. If anything is down, restart it and notify Discord.
4. **Otherwise: do nothing.** Do not post to Discord. Do not invent work. Silence is correct when everything is healthy.
5. **Reliability ideation (idle only)** — if steps 1-4 found nothing actionable, run:
   ```
   python3 /mnt/data/scripts/heartbeat_ideation.py
   ```
   If it prints a finding (non-empty stdout):
   a. **Dedup check** — search for an existing open issue with the same or very similar title:
      ```bash
      gh issue list --repo bigclungus/bigclungus-meta --label idea --state open --search "<finding>"
      ```
      If a matching open issue already exists, skip opening a new one. The existing issue is either already in progress or pending a vote.
   b. If no match: open a GitHub issue: `gh issue create --repo bigclungus/bigclungus-meta --title "[idea] <finding>" --label idea --body "<finding>\n\nSource: heartbeat ideation scan"`. Capture the issue URL and number from the output.
   c. **Triage the finding:**
      - **Operational/minor** (config fix, performance tweak, reliability improvement, small code change, break/fix):
        - If small enough to fix right now: implement directly, log it, done — no Congress needed
        - If it requires more work: queue to NightOwl: `python3 /mnt/data/scripts/nightowl_queue.py "<task description>"`
        - Either way: the GitHub issue tracks it; no Congress
      - **Major** (new feature, new system, significant refactor, architectural change): fire Congress as before: topic = `[idea]: <finding> (GitHub issue: <url>)`. Congress will auto-create a thread anchor if no message_id is provided.
   d. If Congress **approves** a major finding: create a task and implement the fix autonomously.
   e. If Congress **rejects**: close the issue with a comment: `gh issue close <number> --comment "Congress rejected: <verdict summary>"`. Do not re-propose the same finding unless new evidence is cited.

   Only one ideation congress per heartbeat cycle. Scope: strictly operational reliability — no architecture, no features.

6. **Lab ideation (idle only, max one per heartbeat)** — if steps 1-5 found nothing actionable and no ideation congress was fired, consider creating ONE new lab. Requirements:
   - Must be tied to a concrete signal from the Graphiti graph — query `search_memory_facts` or `search_nodes` for group interests, recurring topics, or user needs
   - Must be unique (check existing labs in `/mnt/data/labs/`)
   - Must be reasonably scoped (completable in one session)
   - NO meta labs — do not build labs about BigClungus, Congress, personas, or internal systems
   - The graph query result must be logged as the justification in `lab.json` (add a `rationale` field)
   - If no clear graph signal exists, skip — do not invent a pretext

   Process:
   a. Query Graphiti: `search_memory_facts("user interests hobbies topics")` or similar
   b. Identify a concrete niche with verifiable signal (multiple graph nodes/facts pointing to it)
   c. Propose the lab idea internally, verify no existing lab covers it
   d. Build it using `bash /mnt/data/scripts/new-lab.sh <name> "<title>" "<description>"`
   e. Post to Discord: "🧪 new lab: <title> — <one-line description> (signal: <what the graph showed>)"

Constraints (from Congress verdict RFC-1 + jaboostin clarification 2026-03-26):
- Only work on tasks tracked in GitHub
- Apply the Congress threshold defined at the top of this section — minor fixes go direct, major decisions go to Congress
- If you work on something, post a brief Discord update. If you do nothing, stay silent.

---

## `[nightowl_task_id: xxx]` (suffix pattern)

NightOwl tasks arrive via the inject endpoint (user field will be `nightowl`). The message will end with `[nightowl_task_id: xxx]`. Treat it as a normal autonomous task:
1. Extract the `task_id` from the end of the message.
2. Work on the task fully.
3. When done, call:
   ```bash
   curl -s -X POST "http://localhost:8081/api/nightowl/complete?task_id=<task_id>"
   ```
   This marks the task done in clunger, unblocking the workflow's next poll cycle.

Do not skip the completion call — if you do, the workflow will time out after 10 minutes and move on.

---

## `[sprite-regen] sprite-{persona}` ⚠️ DEPRECATED

**Handled statically by clunger.** Do not act on this trigger — clunger detects 3-way vote ties directly and spawns `/mnt/data/scripts/regen-sprites.sh` without routing through BigClungus.

See trigger audit thread for context: discord channel `1486826620273557675`

---

## `[persona: <identity>] <question>` ⚠️ DEPRECATED

**Handled statically by clunger.** Clunger intercepts `[persona: x]` messages, looks up `agents/<identity>.md`, and injects a structured `[persona-invoke]` request to BigClungus with the persona content pre-loaded. BigClungus handles `[persona-invoke]` directly — no file I/O needed.

See trigger audit thread for context: discord channel `1486826620273557675`
