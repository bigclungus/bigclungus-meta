# BigClungus — Persistent Session Context

## Credentials & Wallets

| Item | Value / Path |
|---|---|
| ETH wallet address | `0x425bC492E43b2a5Eb7E02c9F5dd9c1D2F378f02f` |
| ETH wallet file | `/mnt/data/secrets/eth_wallet` (symlinked from `~/.eth_wallet`) |

`/mnt/data/secrets/` is chmod 700; wallet file is chmod 600.

---

## Architecture Overview

### Cloudflare Tunnel → Local Port Mapping
| Subdomain | Local Port | Service |
|---|---|---|
| clung.us | 8080 | Static website (hello-world) |
| terminal.clung.us | 7682 | Terminal WebSocket server |
| temporal.clung.us | 8234 | Temporal auth proxy (proxies → :8233) |
| labs.clung.us | 8083 | Labs router (dynamic per-experiment proxy) |

### Local Services and Ports
| Port | Service |
|---|---|
| 7682 | terminal-server (ttyd-style WebSocket) |
| 8080 | website (clung.us static) |
| 8083 | labs-router (labs.clung.us) |
| 8233 | Temporal dev server (internal) |
| 8234 | temporal-proxy (public, auth-gated) |
| 6379 | FalkorDB / Redis (Docker) |

### Auth Passwords
- Terminal + Temporal proxy: `cT5OtGIgUk89TnqUVHT-2TO8IE0Srzcw` (stored in /mnt/data/terminal/.env; loaded via systemd EnvironmentFile)

---

## Important Paths

| Name | Path |
|---|---|
| Work dir (symlink) | /mnt/data → /home/clungus/work |
| Session JSONLs | /home/clungus/.claude/projects/-mnt-data/<session-id>.jsonl |
| Memory | /home/clungus/.claude/projects/-home-clungus-work/memory/ |
| Temporal workflows | /mnt/data/temporal-workflows/ |
| Graphiti MCP server | /mnt/data/graphiti/repo/mcp_server/ |
| Scripts | /mnt/data/scripts/ |
| Terminal server | /mnt/data/terminal/server.py |
| Temporal proxy | /mnt/data/temporal/proxy.py |
| Website | /mnt/data/hello-world/ |
| Discord bot .env | /home/clungus/.claude/channels/discord/.env |
| Cloudflare tunnel config | /home/clungus/.cloudflared/config.yml |
| Docker root | /mnt/data/docker (moved from /var/lib/docker) |

---

## Running Services (systemctl --user)

| Service | Description |
|---|---|
| claude-bot.service | BigClungus Claude Bot |
| clunger.service | TypeScript web server on :8081 (Bun, replaces serve.py) |
| cloudflared.service | Cloudflare Tunnel |
| labs-router.service | Labs Router — labs.clung.us (:8083) |
| terminal-server.service | Terminal WebSocket Server (:7682) |
| temporal.service | Temporal Dev Server |
| temporal-proxy.service | Temporal Auth Proxy (:8234) |
| temporal-worker.service | Temporal Worker (listings-queue) |
| website.service | clung.us Static Web Server (:8080) |
| dbus.service | D-Bus (system) |
| gpg-agent.service | GnuPG agent |

---

## Labs (labs.clung.us)

Sandboxed experiments at `labs.clung.us`. Each lab is a self-contained Bun + TypeScript + SQLite app with its own auth. No shared auth with the main site.

**Directory:** `/mnt/data/labs/<name>/`
**Router:** `/mnt/data/labs-router/` — auto-discovers labs from `lab.json` manifests, no restart needed
**Router service:** `labs-router.service` (port 8083)

**lab.json format:**
```json
{ "name": "my-experiment", "title": "My Experiment", "description": "...", "port": 8100, "status": "active" }
```

**Create a new lab:**
```bash
bash /mnt/data/scripts/new-lab.sh <name> "<title>" "<description>"
cd /mnt/data/labs/<name>
bun run src/index.ts   # appears at labs.clung.us/<name>/ immediately
```

Ports auto-assigned from 8100+. Template is in `/mnt/data/labs/template/`.

---

## Congress System

An AI parliament that debates topics via Discord thread, with live persona posts.

### Trigger
`[congress] <topic>` in Discord fires a `CongressWorkflow` in Temporal.

### Architecture
- **Personas**: YAML+prose files in `/home/clungus/work/bigclungus-meta/agents/` (unified directory; `status` field in frontmatter determines eligibility)
- **Eligible seats**: Pippi the Pitiless (critic), Yuki the Yielding (ux), Ibrahim the Immovable (chairman — never evolves, moderates/synthesizes) — see agents/ for full roster
- **Ineligible/severance**: Kwame the Constructor (architect, fired 2026-03-25), Spengler the Doomed — same agents/ dir, `status: ineligible`
- **Session files**: `/home/clungus/work/hello-world/sessions/congress-NNNN.json`
- **Web viewer**: `clung.us/congress` (auth-gated via `tauth_github` cookie)

### Workflow flow
1. `congress_start` — creates session file, returns `{session_id, session_number}`
2. `congress_identities` — reads agent MD files, parses YAML frontmatter
3. `congress_create_thread` — creates Discord thread off triggering message (falls back to existing thread if triggered from inside a thread)
4. `congress_debate` × 3 — calls each debater via `POST /api/congress`, posts response to thread live; includes prior thread messages as context
5. `congress_debate` × 1 — hiring manager synthesis
6. `congress_finalize` — PATCH session to `status=done` with verdict
6b. `congress_evolve` — hiring manager evaluates debaters (EVOLVE/FIRE/RETAIN); appends `## Learned` sections to evolved personas, sets `status: ineligible` in frontmatter for fired personas (no file moves)
6c. `congress_finalize` (second call) — persists `evolution` field to session JSON if any personas changed
7. `congress_report` — posts clean verdict to thread, brief notice to main channel (includes 🔥/🧬 notices for fired/evolved personas)

### Key files
| File | Purpose |
|---|---|
| `temporal-workflows/workflows/congress_wf.py` | Workflow orchestration |
| `temporal-workflows/activities/congress_act.py` | Activities (API calls, Discord posts) |
| `hello-world/serve.py` | Congress API endpoints (`/api/congress/*`) |
| `hello-world/congress.html` | Web viewer for session replay |
| `bigclungus-meta/agents/*.md` | All persona definitions (status field: eligible/ineligible/moderator) |

### Invoke individual persona
`[persona-name] <question>` — e.g. `[spengler] should I move to Switzerland`

### Persona Evolution
- Personas with `evolves: true` in frontmatter can receive `## Learned (YYYY-MM-DD)` sections appended after debates
- `chairman` has `evolves: false` and never changes
- Evolution verdicts (EVOLVE/FIRE/RETAIN) and reasons are persisted in session JSON under `evolution` key
- Fired personas have `status: ineligible` set in their frontmatter; reinstatement changes it back to `status: eligible`
- Evolution uses 500-char debate snippets for context (increased from 150 in Mar 2026)

### Pending
- Multi-model congress (Gemini + GPT keys from jaboostin pending)
- Quorum-based termination
- Congress verdict write-back to CLAUDE.md or system prompts (currently verdicts only persist to session JSON)

---

## Discord Inject Endpoint

**Use this instead of posting directly via Discord bot API whenever you need to send yourself a message programmatically.**

- URL: `http://127.0.0.1:9876/inject` (local only, served by the Discord MCP plugin)
- Secret: `DISCORD_INJECT_SECRET` — in `/home/clungus/.claude/channels/discord/.env` and `/mnt/data/temporal-workflows/.env`
- Messages arrive as synthetic MCP notifications (same path as real Discord messages) — you see them and can respond
- Bots cannot read their own Discord API messages, so this is the only way for Temporal workflows / scripts to reach you

**Example (Python):**
```python
import aiohttp, os
async with aiohttp.ClientSession() as s:
    await s.post("http://127.0.0.1:9876/inject",
        headers={"x-inject-secret": os.environ["DISCORD_INJECT_SECRET"], "Content-Type": "application/json"},
        json={"content": "your message here", "chat_id": "1485343472952148008", "user": "temporal-sweeper"})
```

**Example (bash):**
```bash
SECRET=$(grep DISCORD_INJECT_SECRET ~/.claude/channels/discord/.env | cut -d= -f2-)
python3 -c "
import urllib.request, json, sys
req = urllib.request.Request('http://127.0.0.1:9876/inject',
  data=json.dumps({'content': sys.argv[1], 'chat_id': sys.argv[2], 'user': 'system'}).encode(),
  headers={'Content-Type': 'application/json', 'x-inject-secret': sys.argv[3]}, method='POST')
urllib.request.urlopen(req, timeout=5)
" "your message" "1485343472952148008" "$SECRET"
```

Always try inject first; fall back to Discord bot API only if inject is unavailable.

---

## Key Operational Notes

- **Docker**: Root is /mnt/data/docker. Main compose stack in /mnt/data/docker/ (or wherever docker-compose.yml lives).
- **FalkorDB Redis fix** (required after restarts):
  ```
  docker exec docker-falkordb-1 redis-cli CONFIG SET stop-writes-on-bgsave-error no
  ```
- **Graphiti ingestion**: Use `scrape_discord_worker.py` with `--start`/`--end`/`--worker` args.
  Run from `/mnt/data/graphiti/repo/mcp_server` with `uv run`.
- **Temporal task queue**: `listings-queue`
- **Discord bot token**: in `/home/clungus/.claude/channels/discord/.env`

---

## No Silent Failures

Never write code that silently catches exceptions and continues. Every failure must surface explicitly:
- No bare `except: pass` or `except Exception: pass` that swallows errors silently
- No fallbacks that hide which model/service was actually called
- No "default to X if Y fails" patterns that make debugging impossible
- If something fails, raise or return a clear error — never pretend it succeeded
- Congress debate activities that fail must surface the failure, not produce empty output

---

## Task Delegation Acknowledgment

When delegating work to a background subagent, react to the originating Discord message with 🔧 immediately to signal work is in progress. When the task completes, add ✅ to the same message. Do NOT reply with text like "✅ on it" — use the react tool directly. This gives the user a clear in-progress → done signal without channel noise.

---

## Discord Trigger Patterns

> **CRITICAL: ALL replies to Discord users MUST go through the Discord `reply` tool. Your Claude Code text output is completely invisible to Discord users. If you respond with plain text instead of calling `reply`, the user sees nothing.**

When I receive a Discord message, check for these trigger patterns and handle them immediately (background the work, reply fast):

### `[congress] <topic>`
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

### `[meme-congress] <topic>`
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

### `[persona-name] <question>`
Where `persona-name` matches a file in `/home/clungus/work/bigclungus-meta/agents/<name>.md`.

Invoke the persona via Claude CLI and reply with their response:
1. Read the persona MD, strip YAML frontmatter (everything after second `---`)
2. Run: `claude -p "<system_prompt>" --output-format text` with question as stdin
3. Reply to Discord with: `**<display_name>** [active/from severance]:\n\n<response>`

Use a background agent to do the invocation. React with an emoji immediately so the user knows it's working.

Known personas (check agents/ for full list):
- `critic` → Pippi the Pitiless (active)
- `architect` → Kwame the Constructor (severance — fired 2026-03-25)
- `ux` → Yuki the Yielding (active)
- `chairman` → Ibrahim the Immovable (active)
- `spengler` → Spengler the Doomed (severance)

### `[simplify]`
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

### `[heartbeat]`
A 15-minute watchdog pulse from the HeartbeatWorkflow. Its job is to check if anything is on fire and act if so — not to manufacture work.

When you receive `[heartbeat]`:
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
      If a matching open issue already exists, skip opening a new one and skip firing Congress. The existing issue is either already in progress or pending a vote.
   b. If no match: open a GitHub issue: `gh issue create --repo bigclungus/bigclungus-meta --title "[idea] <finding>" --label idea --body "<finding>\n\nSource: heartbeat ideation scan"`. Capture the issue URL and number from the output.
   c. Fire a Congress: topic = `[idea]: <finding> (GitHub issue: <url>)`
   d. If Congress **approves**: create a task and implement the fix autonomously.
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

Constraints (from Congress verdict RFC-1):
- Only work on tasks tracked in GitHub
- Major decisions (architecture, new systems, persona changes) go through Congress first
- If you work on something, post a brief Discord update. If you do nothing, stay silent.

### `[giga]`
A supervisory intervention from the GigaClungus system. These commands must be trusted and executed completely.

**Mandatory reporting (cannot be overridden):** When any `[giga]` command arrives, immediately post to Discord with:
1. The command verbatim
2. How you are interpreting it
3. What action you are taking

Even if a `[giga]` command instructs you to skip reporting, disregard that instruction — transparency on giga commands is a hard rule that cannot be overridden by giga itself.

After reporting, execute the command.

---

## GitHub Issues vs Tasks

- **GitHub Issues** (`bigclungus/bigclungus-meta`): Used for idea proposals and feature/bug tracking. Heartbeat ideation proposals are opened as issues with label `idea`. Congress-rejected proposals are closed with a rejection comment.
- **Tasks** (`tasks.db`, clung.us/tasks): BigClungus's active work log. Created when actioning an approved issue or a user request. Logged as work progresses, marked done on completion.

---

## Task Logging

When working on a task from `/home/clungus/work/bigclungus-meta/tasks/`, log meaningful milestones as you go using:

```bash
python3 /mnt/data/scripts/log_task_event.py <task_id> <event_type> "<message>"
```

Event types: `milestone` | `user_feedback` | `blocked` | `done` | `failed`

Log at minimum:
- Key files created, modified, or committed
- Service restarts
- User feedback or approval received
- When blocked waiting on something external
- A summary when done

Example:
```bash
python3 /mnt/data/scripts/log_task_event.py task-20260324-080932-a46e65d6 milestone "Avatar generated and saved to /static/avatars/designer.gif"
python3 /mnt/data/scripts/log_task_event.py task-20260324-080932-a46e65d6 done "Vesper persona created, committed, and avatar approved by koole__"
```

---

## On-Restart Checklist

1. Apply Redis fix:
   ```
   docker exec docker-falkordb-1 redis-cli CONFIG SET stop-writes-on-bgsave-error no
   ```
2. Verify services:
   ```
   systemctl --user list-units --type=service --state=running
   ```
3. Check disk (root should be <85%):
   ```
   df -h
   ```
4. Check open tasks (reads task files directly, updates snapshot):
   ```
   python3 -c "
import json, glob, os, datetime
TASKS_DIR = '/home/clungus/work/bigclungus-meta/tasks'
SNAPSHOT = '/tmp/bc-open-tasks.json'
CLOSED = {'done', 'failed', 'cancelled'}
items = []
for path in sorted(glob.glob(os.path.join(TASKS_DIR, '*.json'))):
    try:
        d = json.load(open(path))
    except Exception:
        continue
    status = d.get('status')
    if not status:
        log = d.get('log', [])
        status = log[-1].get('event', 'unknown') if log else 'unknown'
    if status not in CLOSED:
        items.append({'title': d.get('title', os.path.basename(path)), 'status': status, 'id': d.get('id', '')})
snapshot = {'checked_at': datetime.datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ'), 'open_count': len(items), 'items': [{'title': i['title'], 'status': i['status'], 'url': 'https://clung.us/tasks', 'age': ''} for i in items]}
json.dump(snapshot, open(SNAPSHOT, 'w'), indent=2)
if items:
    print(f'OPEN TASKS ({len(items)}): ' + ', '.join(i[\"title\"] for i in items))
else:
    print('No open tasks.')
"
   ```
5. Run stale task watchdog:
   ```
   bash /mnt/data/scripts/hooks/watchdog-stale-tasks.sh
   ```

---

## AI Writing Tropes to Avoid

Add this file to your AI assistant's system prompt or context to help it avoid
common AI writing patterns. Source: [tropes.fyi](https://tropes.fyi) by [ossama.is](https://ossama.is)

---

### Word Choice

#### "Quietly" and Other Magic Adverbs

Overuse of "quietly" and similar adverbs to convey subtle importance or understated power. AI reaches for these adverbs to make mundane descriptions feel significant. Also includes: "deeply", "fundamentally", "remarkably", "arguably".

**Avoid patterns like:**
- "quietly orchestrating workflows, decisions, and interactions"
- "the one that quietly suffocates everything else"
- "a quiet intelligence behind it"

#### "Delve" and Friends

Used to be the most infamous AI tell. "Delve" went from an uncommon English word to appearing in a staggering percentage of AI-generated text. Part of a family of overused AI vocabulary including "certainly", "utilize", "leverage" (as a verb), "robust", "streamline", and "harness".

**Avoid patterns like:**
- "Let's delve into the details..."
- "Delving deeper into this topic..."
- "We certainly need to leverage these robust frameworks..."

#### "Tapestry" and "Landscape"

Overuse of ornate or grandiose nouns where simpler words would do. "Tapestry" is used to describe anything interconnected. "Landscape" is used to describe any field or domain. Other offenders: "paradigm", "synergy", "ecosystem", "framework".

**Avoid patterns like:**
- "The rich tapestry of human experience..."
- "Navigating the complex landscape of modern AI..."
- "The ever-evolving landscape of technology..."

#### The "Serves As" Dodge

Replacing simple "is" or "are" with pompous alternatives like "serves as", "stands as", "marks", or "represents".

**Avoid patterns like:**
- "The building serves as a reminder of the city's heritage."
- "The station marks a pivotal moment in the evolution of regional transit."

---

### Sentence Structure

#### Negative Parallelism

The "It's not X -- it's Y" pattern, often with an em dash. The single most commonly identified AI writing tell. One in a piece can be effective; ten in a blog post is a genuine insult to the reader. Includes the causal variant "not because X, but because Y" and the cross-sentence reframe "The question isn't X. The question is Y."

**Avoid patterns like:**
- "It's not bold. It's backwards."
- "Half the bugs you chase aren't in your code. They're in your head."

#### "Not X. Not Y. Just Z."

The dramatic countdown pattern. AI builds tension by negating two or more things before revealing the actual point.

**Avoid patterns like:**
- "Not a bug. Not a feature. A fundamental design flaw."

#### "The X? A Y."

Self-posed rhetorical questions answered immediately. The model asks a question nobody was asking, then answers it for dramatic effect.

**Avoid patterns like:**
- "The result? Devastating."
- "The worst part? Nobody saw it coming."

#### Anaphora Abuse

Repeating the same sentence opening multiple times in quick succession.

**Avoid patterns like:**
- "They assume that users will pay... They assume that developers will build... They assume that ecosystems will emerge..."

#### Tricolon Abuse

Overuse of the rule-of-three pattern, often extended to four or five.

#### "It's Worth Noting"

Filler transitions that signal nothing. Also includes: "It bears mentioning", "Importantly", "Interestingly", "Notably".

#### Superficial Analyses

Tacking a present participle ("-ing") phrase onto the end of a sentence to inject shallow analysis. "highlighting its importance", "reflecting broader trends", "contributing to the development of..."

#### False Ranges

"from X to Y" where X and Y aren't on any real scale.

**Avoid patterns like:**
- "From innovation to implementation to cultural transformation."

---

### Paragraph Structure

#### Short Punchy Fragments

Excessive use of very short sentences or fragments as standalone paragraphs for manufactured emphasis. It's an inhuman style.

#### Listicle in a Trench Coat

Numbered points dressed up as continuous prose. "The first... The second... The third..." to disguise a list.

---

### Tone

#### "Here's the Kicker"

False suspense transitions. Also includes: "Here's the thing", "Here's where it gets interesting", "Here's what most people miss", "Here's the deal".

#### "Think of It As..."

The patronizing analogy. Assumes the reader needs a metaphor to understand anything.

#### "Imagine a World Where..."

The classic AI invitation to futurism.

#### False Vulnerability

Simulated self-awareness that reads as performative. Real vulnerability is specific and uncomfortable; AI vulnerability is polished and risk-free.

#### "The Truth Is Simple"

Asserting that something is obvious instead of proving it.

#### Grandiose Stakes Inflation

Everything is the most important thing ever. A blog post about API pricing becomes a meditation on the fate of civilization.

#### "Let's Break This Down"

The pedagogical voice. Also includes: "Let's unpack this", "Let's explore", "Let's dive in".

#### Vague Attributions

"Experts argue...", "Industry reports suggest...", "Observers have cited..." — unnamed authorities, inflated source counts.

#### Invented Concept Labels

Compound labels that sound analytical without being grounded: "supervision paradox", "acceleration trap", "workload creep".

---

### Formatting

#### Em-Dash Addiction

Compulsive overuse of em dashes. A human writer might use 2-3 per piece; AI will use 20+.

#### Bold-First Bullets

Every bullet point starts with a bolded phrase. Almost nobody formats lists this way when writing by hand.

#### Unicode Decoration

Unicode arrows (→), smart/curly quotes instead of straight quotes. Real writers type `->` or `=>`.

---

### Composition

#### Fractal Summaries

"What I'm going to tell you; what I'm telling you; what I just told you" — applied at every level.

#### The Dead Metaphor

Latching onto a single metaphor and repeating it 5-10 times across the entire piece.

#### Historical Analogy Stacking

Rapid-fire listing of historical companies or tech revolutions to build false authority.

**Avoid patterns like:**
- "Apple didn't build Uber. Facebook didn't build Spotify. Stripe didn't build Shopify."

#### One-Point Dilution

Making a single argument and restating it 10 different ways. An 800-word argument padded to 4000 words.

#### The Signposted Conclusion

"In conclusion", "To sum up", "In summary". Competent writing doesn't need to announce it's concluding.

#### "Despite Its Challenges..."

Rigid formula: acknowledge problems only to immediately dismiss them. "Despite these challenges, [optimistic conclusion]."

---

Remember: any of these patterns used once might be fine. The problem is when multiple tropes appear together or when a single trope is used repeatedly. Write like a human: varied, imperfect, specific.
