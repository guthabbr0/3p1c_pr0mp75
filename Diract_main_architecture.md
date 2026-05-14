# Implementation Prompt — Discord Self-Bot + Ollama Cloud + Web Control Panel

**Target audience:** the coding agent (and any human reviewer) who will implement this system.
**Authority:** this document is the source of truth. If something here conflicts with your training-time intuitions, this document wins. If something is ambiguous, ask before improvising.

---

## 0. Mission, in one paragraph

Build a single Python script (`bot.py`) that runs (a) a Discord self-bot account using `discord.py-self`, (b) an HTTP + WebSocket control panel served by FastAPI on the same asyncio event loop, and (c) a bridge to Ollama Cloud (https://docs.ollama.com/cloud) for LLM responses. The bot responds when pinged or replied-to in any channel or DM, with conversation context attached. Everything — messages, commands, LLM I/O, errors — is persisted to a local SQLite database with FTS5 indexes. The control panel is the operator's cockpit: live logs, configuration, permissions, per-scope system prompts, memory inspection, command issuing, and a private operator-to-bot webchat.

---

## 1. Non-negotiable hard constraints

These are not preferences. Violating any of these is a defect.

1. **Single Python script.** All Python code lives in `bot.py`. All HTML, CSS, and JS for the control panel is embedded as triple-quoted strings in `bot.py`. The only external files are: the SQLite database, a `.env` or `config.toml` for secrets, the test files, `AGENTS.md`, and `README.md`.
2. **No `!` command from an unauthorized user ever reaches the LLM.** Not as part of context, not as part of a summary, not anywhere. The authorization filter is the first thing every inbound message hits. Authorized commands are also never sent to the LLM — they are interpreted by deterministic code. The LLM only ever sees natural-language conversation content from authorized scopes.
3. **The bot only "wakes" on direct ping or reply.** It does not respond to ambient chatter. When it wakes, it gathers everything posted in that channel since its last wake (or since `!clear`/`!compact`/`!summary`), prepends the active system prompt and relevant memory, and sends one Ollama request.
4. **All Discord messages and all LLM I/O are persisted.** Including messages from non-authorized users (we need the audit trail), including messages the bot chose not to respond to.
5. **The database must remain queryable under load.** WAL mode, sensible indexes, FTS5 for text search, and a single writer pattern (all writes go through one asyncio queue).
6. **ISO-8601 UTC timestamps everywhere.** In the DB (`TEXT` column, format `YYYY-MM-DDTHH:MM:SS.sssZ`), in the UI, in logs, in API responses. Decimal point separator, space as digit grouping.
7. **The control panel is the same surface area as Discord.** Anything a user can do from Discord (issue a command, talk to the bot) the operator can do from the panel. Commands from the panel are logged identically with `source='panel'` instead of `source='discord'`.

---

## 2. Risk and compliance — must be in README

`discord.py-self` automates a user account, which is not covered by Discord's ToS and risks account limitation. The README must lead with this notice, and the configuration must require the operator to acknowledge it (add `i_understand_selfbot_risk = true` line in config that the bot refuses to start without). The implementation should be structured so - keep `discord.py-self`-specific behavior isolated to the client initialization and login flow.

---

## 3. Technology stack

| Layer | Choice | Why |
|---|---|---|
| Language | Python 3.11+ | `asyncio.TaskGroup`, `tomllib`, structural pattern matching |
| Discord client | `discord.py-self` | Maintained self-bot fork |
| Web framework | `fastapi` + `uvicorn` (programmatic) | Async-native, mounts cleanly into our loop |
| HTTP client (Ollama) | `httpx` (async) | Streaming support, connection pooling |
| Database | `sqlite3` stdlib + `aiosqlite` | FTS5 built-in, WAL, zero ops |
| Templating | None — embedded strings | Single-script constraint |
| Frontend JS | Alpine.js (CDN) + HTMX (CDN) | No build step, declarative, tiny |
| Frontend CSS | Hand-written, embedded | Aesthetic control, no Tailwind bloat |
| Auth (panel) | Single shared secret token, cookie session | Single-operator; keep it boring |
| Tests | `pytest` + `pytest-asyncio` + `httpx` test client | Standard async testing |

Locked dependency list (`requirements.txt`):
```
discord.py-self>=2.0
fastapi>=0.110
uvicorn[standard]>=0.27
httpx>=0.27
aiosqlite>=0.20
pydantic>=2.6
python-multipart>=0.0.9
pytest>=8.0
pytest-asyncio>=0.23
```

No other runtime deps. Resist the urge.

---

## 4. File layout

```
project-root/
├── bot.py                  # ALL Python and embedded frontend
├── config.toml             # Operator-edited; not committed
├── config.example.toml     # Committed template
├── bot.sqlite              # Created on first run; not committed
├── tests/
│   ├── conftest.py
│   ├── test_permissions.py
│   ├── test_commands.py
│   ├── test_context.py
│   ├── test_memory.py
│   ├── test_db.py
│   └── test_panel_api.py
├── AGENTS.md               # Instructions for future coding agents
├── README.md               # Operator-facing docs + risk notice
└── requirements.txt
```

---

## 5. Database schema

SQLite, WAL mode, `PRAGMA foreign_keys=ON`, `PRAGMA journal_mode=WAL`, `PRAGMA synchronous=NORMAL`. All writes funnel through a single `asyncio.Queue` consumed by one writer task — this avoids `database is locked` under concurrent ingest and panel writes.

### 5.1 Core tables

```sql
-- Every Discord message we observe, in any channel we have access to.
CREATE TABLE messages (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    discord_msg_id  TEXT UNIQUE NOT NULL,
    scope_type      TEXT NOT NULL CHECK (scope_type IN ('dm','group','guild')),
    scope_id        TEXT NOT NULL,           -- channel_id or DM partner user_id
    guild_id        TEXT,
    author_id       TEXT NOT NULL,
    author_name     TEXT NOT NULL,
    content         TEXT NOT NULL,
    is_command      INTEGER NOT NULL DEFAULT 0,
    is_authorized   INTEGER NOT NULL DEFAULT 0,  -- author whitelisted for this scope
    triggered_bot   INTEGER NOT NULL DEFAULT 0,  -- ping/reply that woke the bot
    reply_to_id     TEXT,
    timestamp_utc   TEXT NOT NULL
);
CREATE INDEX idx_messages_scope_ts ON messages(scope_type, scope_id, timestamp_utc);
CREATE INDEX idx_messages_author_ts ON messages(author_id, timestamp_utc);

-- FTS5 mirror for full-text search across the panel.
CREATE VIRTUAL TABLE messages_fts USING fts5(
    content, author_name, scope_id,
    content='messages', content_rowid='id',
    tokenize='unicode61 remove_diacritics 2'
);
-- Triggers keep FTS in sync with messages (insert/delete/update). MUST be created.

-- Scoped system prompts.
CREATE TABLE prompts (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    scope_type    TEXT NOT NULL CHECK (scope_type IN ('global','dm','group','guild')),
    scope_id      TEXT,                    -- NULL for global
    body          TEXT NOT NULL,
    updated_at    TEXT NOT NULL,
    updated_by    TEXT NOT NULL,           -- user_id or 'panel'
    UNIQUE (scope_type, scope_id)
);

-- Authorization whitelist. Separate rows per scope so a user can be admin in DMs
-- but not in groups, etc.
CREATE TABLE permissions (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id       TEXT NOT NULL,
    scope_type    TEXT NOT NULL CHECK (scope_type IN ('global','dm','group','guild')),
    scope_id      TEXT,                    -- NULL for global
    level         TEXT NOT NULL CHECK (level IN ('admin','user','blocked')),
    added_at      TEXT NOT NULL,
    UNIQUE (user_id, scope_type, scope_id)
);

-- Audit trail for every command attempted, accepted or rejected.
CREATE TABLE commands_log (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    source        TEXT NOT NULL CHECK (source IN ('discord','panel')),
    user_id       TEXT NOT NULL,
    scope_type    TEXT NOT NULL,
    scope_id      TEXT,
    command       TEXT NOT NULL,
    args          TEXT,
    accepted      INTEGER NOT NULL,
    reason        TEXT,                    -- e.g. 'unauthorized', 'ok', 'bad_args'
    timestamp_utc TEXT NOT NULL
);

-- Ollama request/response audit. One row per call.
CREATE TABLE ollama_log (
    id              INTEGER PRIMARY KEY AUTOINCREMENT,
    scope_type      TEXT NOT NULL,
    scope_id        TEXT,
    model           TEXT NOT NULL,
    prompt_tokens   INTEGER,
    completion_tokens INTEGER,
    latency_ms      INTEGER,
    request_json    TEXT NOT NULL,         -- full body sent
    response_json   TEXT,                  -- full body received (or error)
    error           TEXT,
    timestamp_utc   TEXT NOT NULL
);

-- Bot-level logs (errors, info, debug).
CREATE TABLE bot_logs (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    level         TEXT NOT NULL CHECK (level IN ('debug','info','warn','error')),
    component     TEXT NOT NULL,           -- 'discord','ollama','panel','db','context'
    message       TEXT NOT NULL,
    detail_json   TEXT,
    timestamp_utc TEXT NOT NULL
);
CREATE INDEX idx_bot_logs_level_ts ON bot_logs(level, timestamp_utc);

-- Per-scope active context window and accumulated summaries.
CREATE TABLE context_state (
    scope_type      TEXT NOT NULL,
    scope_id        TEXT NOT NULL,
    last_wake_utc   TEXT,                  -- last time bot answered here
    rolling_summary TEXT,                  -- compacted history
    last_message_id INTEGER,               -- id from `messages` table
    PRIMARY KEY (scope_type, scope_id)
);

-- Bot's memory: persistent remarks about users and channels.
-- The LLM can read and write these via tool calls.
CREATE TABLE memories (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    target_type   TEXT NOT NULL CHECK (target_type IN ('user','channel')),
    target_id     TEXT NOT NULL,
    note          TEXT NOT NULL,
    tags          TEXT,                    -- comma-separated; e.g. 'hobby,music'
    confidence    REAL NOT NULL DEFAULT 0.7 CHECK (confidence BETWEEN 0.0 AND 1.0),
    created_at    TEXT NOT NULL,
    created_by    TEXT NOT NULL,           -- 'bot' or operator user_id
    superseded_by INTEGER,                 -- for soft-edits; points to newer row
    FOREIGN KEY (superseded_by) REFERENCES memories(id)
);
CREATE INDEX idx_memories_target ON memories(target_type, target_id, superseded_by);

-- Operator webchat — separate from the Discord bridge entirely.
CREATE TABLE panel_chat (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    role          TEXT NOT NULL CHECK (role IN ('operator','assistant','system')),
    content       TEXT NOT NULL,
    tool_calls_json TEXT,
    timestamp_utc TEXT NOT NULL
);
```

Add the FTS5 sync triggers (`ai_messages`, `ad_messages`, `au_messages`) in the schema bootstrap. The implementing agent: do this correctly, test it, do not skip.

---

## 6. Command specification

Prefix: `!`. Parser: deterministic, regex-based, runs **before** any LLM logic.

### 6.1 Syntax

```
!<command> [args...]            applies to the current scope (channel or DM)
!<command> [args...] *          applies globally
!<command> [args...] @<id>      applies to a specific scope by ID (panel only, typically)
```

Quoted strings (`'…'` or `"…"`) preserve whitespace. Escape with `\`.

### 6.2 Commands (all admin-only unless noted)

| Command | Args | Effect |
|---|---|---|
| `!prompt` | `'<body>'` `[*\|@id]` | Set system prompt for scope |
| `!prompt` | (no arg) | Show current prompt for scope |
| `!compact` | — | Summarize older half of context, keep recent half verbatim |
| `!summary` | — | Generate full summary, persist to `context_state.rolling_summary`, clear working buffer |
| `!clear` | — | Wipe both working buffer and `rolling_summary` for this scope |
| `!whitelist` | `add\|remove <user_id> [*\|@id]` | Manage permissions |
| `!memory` | `add user\|channel <id> '<note>' [tags=t1,t2]` | Manual memory entry |
| `!memory` | `show user\|channel <id>` | List memories |
| `!model` | `<model_name> [*\|@id]` | Override Ollama model for scope |
| `!status` | — | Bot reports uptime, model, context size, last error |
| `!help` | — | List available commands (per requester's permission level) |

### 6.3 Authorization filter (THE critical security boundary)

Pseudocode for every inbound Discord message:

```
on_message(msg):
    record_to_db(msg)                          # always persist
    if msg.content.startswith('!'):
        parsed = parse_command(msg.content)
        is_authorized = check_permission(msg.author.id, msg.scope, level='admin')
        log_command_attempt(msg, parsed, is_authorized)
        if not is_authorized:
            return                              # HARD STOP. Never goes to LLM.
        execute_command(parsed, msg.scope)
        return                                  # commands are not LLM inputs either.
    if bot_was_pinged_or_replied(msg):
        wake_and_respond(msg.scope)
```

`wake_and_respond` reads from `messages` table — and the assembly step **MUST filter out** rows where `is_command=1`. Tests in `test_permissions.py` will assert this invariant aggressively. See §13.

---

## 7. Context management

### 7.1 What "context" means here

For each scope (DM, group channel, guild channel), the bot maintains:
- **Working buffer**: all messages in that scope since the last wake or `!clear`, filtered to exclude commands.
- **Rolling summary**: a string produced by `!compact` or `!summary`, prepended to the working buffer on the next wake.
- **Active system prompt**: most specific match wins (scope-specific > global).
- **Relevant memories**: pulled from `memories` for the channel + each speaking user (top-N by recency × confidence).

### 7.2 Assembly order on wake

```
[ system_prompt_for_scope ]
[ memory_block ]                # "What you know about this channel and its participants:"
[ rolling_summary ]             # if present
[ working_buffer messages ]     # chronological, command-filtered
[ trigger message ]             # the ping/reply that woke us
```

### 7.3 `!compact` algorithm

1. Take the oldest 50% of the working buffer.
2. Send to Ollama with a dedicated compaction prompt ("Summarize the following conversation preserving names, decisions, and unresolved questions. Be terse.").
3. Append the new summary to `rolling_summary` (or replace if it gets too long — cap at ~2000 tokens, summarize the summary if needed).
4. Drop the compacted half from the working buffer (in practice: update `context_state.last_message_id` to point past it).

### 7.4 `!summary` algorithm

Same as compact but operates on the entire working buffer + existing `rolling_summary`, replacing `rolling_summary` with the new one and resetting `last_message_id` to the most recent message.

### 7.5 `!clear` algorithm

Set `rolling_summary = NULL`, `last_message_id = <current latest>`. The DB row in `messages` is not deleted — clearing affects only context assembly, not the audit log.

### 7.6 Token budgeting

Configurable per-model max context. On assembly, if the assembled prompt exceeds 80% of the budget, automatically trigger `!compact` before sending. Log this as `auto_compact` in `bot_logs`.

---

## 8. Memory system

### 8.1 Behavior

Memory is the bot's long-term recollection of people and places. It is separate from conversation context. The LLM has tool access to read/write it. When assembling context (§7.2), we pre-load the top-N memories for the channel and each recent speaker so the model has them without needing a tool call.

### 8.2 Tools exposed to the LLM

Implement as Ollama tool-calling functions. Schemas:

```json
{
  "name": "memory_add",
  "description": "Record a durable note about a user or channel.",
  "parameters": {
    "target_type": {"enum": ["user","channel"]},
    "target_id": "string",
    "note": "string",
    "tags": "string (comma-separated, optional)",
    "confidence": "number 0.0-1.0 (optional, default 0.7)"
  }
}

{
  "name": "memory_search",
  "description": "Find existing memories.",
  "parameters": {
    "target_type": {"enum": ["user","channel","any"]},
    "target_id": "string (optional)",
    "query": "string (FTS, optional)",
    "limit": "integer (default 10)"
  }
}

{
  "name": "memory_update",
  "description": "Refine or correct an existing memory. Marks old row superseded.",
  "parameters": {
    "memory_id": "integer",
    "new_note": "string",
    "new_tags": "string (optional)",
    "new_confidence": "number (optional)"
  }
}

{
  "name": "log_search",
  "description": "Search past Discord messages (FTS).",
  "parameters": {
    "query": "string",
    "scope_id": "string (optional)",
    "author_id": "string (optional)",
    "since_utc": "string (ISO-8601, optional)",
    "limit": "integer (default 20)"
  }
}
```

### 8.3 Proactive memory ingestion

Periodically (configurable, e.g. every N bot responses or every M hours), kick off a background "reflection" task per active scope:
1. Pull the last K messages from a scope.
2. Run an Ollama prompt: "Identify durable facts about participants (interests, expertise, recurring topics, expressed preferences). Output as JSON array of memory_add calls."
3. Execute the resulting `memory_add` calls (with audit logging).

This makes the bot "proactive" without making it chatty. The operator can toggle this from the panel and review proposed memories before commit (a `memories_pending` flag column or a separate table — implementer's call; prefer adding `created_by='bot_pending'` and approving from the panel).

---

## 9. Ollama Cloud integration

### 9.1 Configuration

Read from `config.toml`:
```toml
[ollama]
endpoint = "https://ollama.com"      # or whatever the cloud base URL resolves to
api_key  = "..."
default_model = "..."
request_timeout_s = 120.0

[discord]
token = "..."
i_understand_selfbot_risk = true

[panel]
host = "127.0.0.1"
port = 8765
auth_token = "..."                   # operator's bearer token

[bot]
trigger_on = ["ping", "reply"]       # both by default
auto_compact_threshold = 0.80
proactive_memory_enabled = true
proactive_memory_interval_minutes = 60
```

### 9.2 Calls

Use the `/api/chat` style endpoint with tool support per Ollama's API. Always send `stream=false` for simplicity (latency is acceptable at this scale). Persist every request and response to `ollama_log` — full bodies, not truncated. Yes, this grows the DB; at <few MB/day of text that's still fine for a year.

### 9.3 Error handling

Timeout → log, post a tasteful "I'm temporarily unable to reach the model" message in the original scope, retry with exponential backoff (max 3 attempts) only for the first response to a given trigger.

---

## 10. Web control panel — specification

### 10.1 Aesthetic direction

Developer cockpit. Dark by default (light toggle). Dense. Monospace for log and chat content; sans-serif (system stack) for UI chrome. No rounded-corner overload. No oversized hero spacing. Think `btop`, `k9s`, the Vercel dashboard's dark mode — but cleaner. Single-page app, tab-switched panels, persistent left-side log tail.

Color palette suggestion (the implementing agent may refine, but stay in this register):
```
--bg:        #0e0f12
--bg-elev:   #15171c
--border:    #23262d
--fg:        #d7dae0
--fg-dim:    #7c828d
--accent:    #6ea8fe
--ok:        #5ec27b
--warn:      #e1b46a
--err:       #e16a6a
--mono: ui-monospace, "JetBrains Mono", "SF Mono", Menlo, monospace
--sans: system-ui, -apple-system, "Inter", sans-serif
```

Layout:
```
┌─────────────────────────────────────────────────────────────────────┐
│ topbar: bot status · uptime · model · scope filter · search · ⚙     │
├──────────┬──────────────────────────────────────────────────────────┤
│ side nav │  active tab content                                      │
│ ──────── │                                                          │
│ Logs     │                                                          │
│ Channels │                                                          │
│ Prompts  │                                                          │
│ Perms    │                                                          │
│ Memory   │                                                          │
│ WebChat  │                                                          │
│ Commands │                                                          │
│ Config   │                                                          │
│          │                                                          │
├──────────┴──────────────────────────────────────────────────────────┤
│ footer: live tail (last 5 log lines, scrollable, colorized by level) │
└─────────────────────────────────────────────────────────────────────┘
```

### 10.2 Tabs

**Logs.** Three sub-views: Discord messages, Ollama I/O, bot logs. Each is a virtualized table (no DOM thrash on 10 000+ rows). Columns: timestamp · scope · author/component · content (truncated, expandable). FTS search box at top. Time-range filter. Level filter for bot logs. Export-to-JSON button per filtered view.

**Channels.** List of every scope the bot has seen, with: last activity, message count (24 h), authorized user count, active model, "view conversation" link (paginated message history with the same render the bot would see), "edit prompt" inline, "edit permissions" inline.

**Prompts.** Editor for global prompt and a list of scope-specific overrides. Monaco-style textarea is overkill; a plain `<textarea>` with monospace font, line numbers via CSS counter, and a "diff against previous version" view from history. Save creates a new row in `prompts` (don't UPDATE, INSERT and supersede — or keep a side `prompts_history` table; implementer's call, but versioning must exist).

**Perms.** Table of whitelist entries. Add/remove rows. Bulk import via paste (one `user_id,level,scope_type,scope_id` per line).

**Memory.** Browse memories grouped by target (user / channel). Inline edit. "Pending" tab for bot-proposed memories awaiting approval. Search box (FTS over notes).

**WebChat.** Single-pane chat UI. Operator on the right, assistant on the left. Persisted in `panel_chat`. This chat runs through the same Ollama integration and has the same tool access as the bot — so the operator can ask "what did user X talk about last week" and the bot answers using `log_search`. Crucially, this chat is **not** bridged to Discord. It's the operator's private console.

**Commands.** Form: target scope dropdown (any seen scope, plus "global"), command dropdown, args field, submit. Effect is identical to issuing the command in Discord, with `source='panel'` in `commands_log`.

**Config.** Edit `config.toml` values from the UI (the implementing agent should save back to the file on apply, with a backup of the previous version, e.g. `config.toml.bak`). Restart-required values are clearly flagged. Test buttons: "test Ollama connection", "test Discord token".

### 10.3 Auth

Single bearer token in `config.panel.auth_token`. On first hit to `/`, redirect to `/login` (a single password field). On submit, set an httpOnly cookie `session=<token>`. All API endpoints check this cookie. No user accounts, no roles — single operator. Document in README that the panel must be behind a reverse proxy with TLS if exposed beyond localhost.

### 10.4 Real-time updates

One WebSocket at `/ws`. Server pushes JSON events:
```
{"type":"message","data":{...}}        # new Discord message
{"type":"ollama","data":{...}}         # new Ollama call
{"type":"log","data":{...}}            # new bot log line
{"type":"command","data":{...}}        # command executed
{"type":"stat","data":{...}}           # periodic stat snapshot
```
Frontend subscribes once on page load, dispatches to whichever tab is rendering.

---

## 11. HTTP API surface (consumed by the panel JS)

All routes require auth cookie. JSON in, JSON out. Paginate with `?limit=&before_id=` (cursor over `id DESC`).

```
GET    /api/stats
GET    /api/messages?scope_type=&scope_id=&q=&since=&limit=
GET    /api/ollama-log?scope_id=&limit=
GET    /api/bot-logs?level=&component=&limit=
GET    /api/commands-log?limit=
GET    /api/scopes                              # list of all seen scopes with metadata
GET    /api/prompts
PUT    /api/prompts                             # { scope_type, scope_id, body }
GET    /api/permissions
POST   /api/permissions                         # add
DELETE /api/permissions/{id}
GET    /api/memories?target_type=&target_id=&q=&pending=
POST   /api/memories
PUT    /api/memories/{id}
DELETE /api/memories/{id}
POST   /api/memories/{id}/approve               # for bot-pending
GET    /api/panel-chat
POST   /api/panel-chat                          # { message } -> streams or returns assistant reply
POST   /api/command                             # { scope_type, scope_id, command, args }
GET    /api/config                              # redacted (no secrets in response, mask token)
PUT    /api/config
POST   /api/config/test-ollama
POST   /api/config/test-discord
GET    /ws                                       # WebSocket
```

---

## 12. Concurrency model

One asyncio event loop, three primary task families:

1. **Discord client** — `discord.py-self` runs its own coroutines; we hook `on_message`, `on_ready`, `on_error`.
2. **Web server** — `uvicorn.Server(config).serve()` awaited as a task.
3. **Background workers**:
   - DB writer (consumes a single `asyncio.Queue`, sole writer to SQLite).
   - WebSocket broadcaster (consumes an internal pub/sub queue, fans out to connected sockets).
   - Proactive memory reflector (periodic).
   - Auto-compaction watcher (checks token budget before each LLM call — synchronous to the call, not a separate task).

Use `asyncio.TaskGroup` to supervise. On any unhandled exception in a critical task, log to `bot_logs` with level `error`, broadcast over WS, and either restart the task (for transient failures) or shut down gracefully (for fatal config errors).

---

## 13. Testing strategy

Tests are not optional and must be written **alongside** the code, not after. `pytest` + `pytest-asyncio`. All tests use a fresh in-memory SQLite per test (or a tmpfile, depending on whether FTS5 works in `:memory:` — verify early).

### 13.1 `test_permissions.py` — the critical suite

These are the tests that protect the system's integrity. They must all pass before any deploy.

- `test_unauthorized_command_never_reaches_llm`: simulate inbound `!prompt 'evil'` from non-whitelisted user; assert (a) command logged as rejected, (b) no Ollama call made (mocked client records zero invocations), (c) no row added to active context buffer.
- `test_authorized_command_executes_but_not_sent_to_llm`: same but authorized; assert command executes, still zero Ollama calls, command not in context assembly output.
- `test_command_in_message_history_excluded_from_context`: pre-populate `messages` with mixed commands and regular text; call context assembler; assert no command lines in output.
- `test_per_scope_authorization`: user authorized in DM scope, unauthorized in group scope; verify both behaviors.
- `test_blocked_user_messages_not_in_context`: blocked-level user; messages logged but excluded from assembly.
- `test_global_vs_scoped_permission_resolution`: global admin should work everywhere; scoped admin only in scope.

### 13.2 `test_commands.py`

- Parser tests: quoted args, escaped quotes, trailing `*`, trailing `@id`, malformed inputs.
- Each command's effect on the DB state.
- Idempotency where it matters (e.g. `!clear` twice should be safe).

### 13.3 `test_context.py`

- Assembly order matches §7.2.
- `!compact` halves the buffer and produces a summary (mock Ollama returns a fixed string).
- `!summary` replaces `rolling_summary`.
- `!clear` resets `last_message_id` but does not delete DB rows.
- Auto-compaction triggers at the configured threshold.

### 13.4 `test_memory.py`

- `memory_add` writes and is retrievable by FTS.
- `memory_update` marks old row superseded and chains correctly.
- Proactive reflection produces pending rows, approve flips them to active.

### 13.5 `test_db.py`

- Schema bootstraps cleanly on empty file.
- FTS triggers keep the virtual table in sync (insert, update, delete a message; query FTS).
- WAL mode is set.
- Single-writer queue serializes (write 1 000 messages concurrently from 10 producers; final count is exactly 1 000, no duplicates, no losses).

### 13.6 `test_panel_api.py`

- Auth: no cookie → 401; wrong token → 401; right token → 200.
- Each route happy path with FastAPI's test client.
- `/api/command` from panel writes `source='panel'` in `commands_log`.
- WebSocket broadcasts on message ingest.

### 13.7 Coverage gate

`pytest --cov=bot --cov-fail-under=80` in CI. The permission and command modules should be at 95 %+ specifically — they are the security surface.

---

## 14. `AGENTS.md` — required content

Generate this file first. It will be read by every future agent that touches this repo. Mandatory sections:

```markdown
# AGENTS.md — instructions for coding agents working on this repo

## Ground rules
- Single Python script. All Python in `bot.py`. Do not split.
- Embed HTML/CSS/JS as triple-quoted strings in `bot.py`. Do not create a `static/` directory.
- ISO-8601 UTC for all timestamps. Decimal dot, space digit grouping.
- Never weaken the permission filter (§13.1 in IMPLEMENTATION_PROMPT.md).
- Add or update tests for every change. Run `pytest` before declaring done.
- Update README.md and AGENTS.md when behavior changes.

## Architecture pointers
- Read IMPLEMENTATION_PROMPT.md before making non-trivial changes.
- Schema changes require a migration block at the top of `bootstrap_db()` and a test in `test_db.py`.
- New commands: parser + handler + permission check + test in `test_commands.py` + entry in `!help`.
- New panel routes: handler + auth check + test in `test_panel_api.py` + frontend wiring.

## Conventions
- All DB writes go through the writer queue. Never call `db.execute(...)` directly from a request handler for INSERT/UPDATE/DELETE.
- All LLM calls go through `OllamaClient.chat()`. Never construct httpx requests inline.
- All log lines for the operator go to `bot_logs` AND get broadcast via the WS pub/sub.

## How to run
- Install: `pip install -r requirements.txt`
- Configure: copy `config.example.toml` to `config.toml` and fill in.
- Run: `python bot.py`
- Test: `pytest`
- Lint: project doesn't enforce a linter, but match existing style.

## What NOT to do
- Don't add dependencies. The list in requirements.txt is closed.
- Don't add a build step for the frontend.
- Don't bypass the permission filter for "convenience".
- Don't log secrets (token, API key) anywhere — they must be masked in API responses and never written to bot_logs.
```

---

## 15. `README.md` — required content

Generate this file first too. Mandatory sections, in this order:

1. **Risk notice** — bold, top of file. ToS warning. The `i_understand_selfbot_risk` config flag.
2. **What it does** — three-paragraph plain-language overview.
3. **Quick start** — copy config, run, open panel.
4. **Configuration reference** — every key in `config.toml`, defaults, units.
5. **Command reference** — table from §6.2.
6. **Permission model** — how scopes work, examples.
7. **Database** — schema overview, where the file lives, how to back it up (`sqlite3 .backup`).
8. **Panel screenshot placeholders** — markdown image refs even if images don't exist yet.
9. **Tests** — `pytest` invocation, what `--cov-fail-under` enforces.
10. **Roadmap / known limitations** — explicit list. At minimum: no message edit handling yet, no attachment handling yet, single operator only.
11. **License** — implementer's choice; MIT.

---

## 16. Implementation milestones

Do them in this order. Don't skip ahead.

1. **Skeleton + DB bootstrap.** `bot.py` opens the DB, applies schema, exits. `test_db.py` passes.
2. **Config loading.** Parse `config.toml`, validate, refuse to start if `i_understand_selfbot_risk` is false. Unit test the loader.
3. **DB writer queue.** Single-writer pattern. `test_db.py` concurrent test passes.
4. **Discord client ingest.** Connect, log every message to DB, no responses yet. Manual smoke test.
5. **Permission filter + command parser.** All of §6 and §13.1. Full test suite for permissions passes.
6. **Command handlers.** `!prompt`, `!whitelist`, `!status`, `!help`. Tests.
7. **Ollama client.** `OllamaClient.chat()` + logging to `ollama_log`. Mock-based tests.
8. **Wake & respond loop.** Ping/reply detection → context assembly → Ollama call → reply post. Tests with mocked Ollama and mocked Discord client.
9. **Context commands.** `!compact`, `!summary`, `!clear`. Tests.
10. **Memory system + tools.** Tool-calling integration, manual `!memory`, proactive reflector. Tests.
11. **FastAPI server scaffold.** Auth, static index, WebSocket echo. Tests.
12. **Panel API endpoints.** All of §11. Tests.
13. **Panel frontend.** Embedded HTML/CSS/JS. All tabs functional.
14. **Operator webchat.** Reuses Ollama client and memory tools.
15. **Polish.** Auto-compaction, error broadcasting, config editing from panel.

After each milestone: run full test suite, update `AGENTS.md` if conventions evolved, update `README.md` if user-facing behavior changed.

---

## 17. Acceptance criteria

The system is "done" when all of the following are true:

- `pytest` passes with coverage ≥ 80 % overall, ≥ 95 % on permission and command modules.
- A non-whitelisted user issuing `!anything` results in: row in `commands_log` with `accepted=0`, zero `ollama_log` rows, zero changes to any prompt/permission/memory/context table.
- The bot wakes only on ping or reply, never on ambient chatter.
- `!compact`, `!summary`, `!clear` behave per §7.
- The panel loads at `http://127.0.0.1:8765`, requires the auth token, and exposes every tab in §10.2.
- Sending a command from the panel produces the same DB state as sending it from Discord, with `source='panel'`.
- The operator can chat with the bot in the WebChat tab; the bot can answer questions about Discord logs by calling `log_search`.
- Stopping and restarting the bot loses no data and resumes cleanly.
- README and AGENTS.md are present, current, and self-consistent with the code.

---

## 18. Notes for the implementing agent

- **Resist scope creep.** Streaming responses, attachment handling, voice — explicit non-goals.
- **Resist dependency creep.** The list in §3 is closed unless you have a strong reason and document it.
- **Resist abstraction creep.** This is a single-file, single-operator tool. A 1 5000-line `bot.py` is fine. A 6-file plugin architecture is not what was asked for.
- **When uncertain, ask.** Especially about: schema changes, dependency additions, anything touching the permission filter, anything that changes the public API.
- **Read this whole document before writing code.** Then read it again after the skeleton is in place. Then once more before declaring done.

End of spec.
