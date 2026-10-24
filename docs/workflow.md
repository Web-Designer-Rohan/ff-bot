# Workflow & Usage

## 0. Setup

```bash
unzip GUILD_GLORY_FIXED.zip -d ff        # keep all files together
cd ff
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install "protobuf>=6,<8"             # required: generated code needs proto 6+
```

Put accounts in `accounts.json` in the **same folder** as `app.py`:

```json
[{"uid": "123456789", "password": "Passw0rd"}]
```

## A. Main bot — launch to match end

1. **Start** — `python app.py`. There are no command-line flags.
   Expect a banner, then `[+] Dashboard: http://localhost:20333`.
2. **Open** `http://localhost:20333` in a browser. You will see the guild XP
   card, session uptime bar, a log terminal, `+ Add Account` and `Stop Bot`.
3. **Add accounts** — click `+ Add Account`, type the **UID** and **password**,
   click **Save**.
   - Writes the account to `accounts.json` and starts a worker immediately.
   - Expect console: `[LOGIN] Guest <uid>...` → `[OK] <id> | <nick> | Lvl<n>`.
   - Token login exists only in the API (`POST /api/account/add {"token": "..."}`);
     the UI has no token field.
4. **Login chain (per account)** — guest register/token grant → `MajorLogin`
   (AES payload) → `GetLoginData` (JWT) → cache to `token_cache.json` →
   `[FUNC] TCP connected <uid> (DNS:… Lvl:… R:<region>)`.
5. **Squad formation** (requires 4 accounts in this process):
   - First connected account logs `[TEAM] 👑 <uid> is leader` and waits up to
     90 s for peers.
   - Leader sends `create_team`, then invites each peer every 4 s for up to 120 s.
   - Peers auto-accept on an incoming `05…` packet and the console shows
     `[TEAM-IN] 📨 ACCEPT` and `👥 In team (n/4)`.
   - Leader logs `══ FINAL: In team (4/4) ══`, then loops `ready → 1 s → start`
     every 5 s. Members start the same loop 25 s after the handshake
     (`TEAM_READY_DELAY + TEAM_MEMBER_START_DELAY`).
6. **Match found** — console prints `MATCH FOUND! Loading...` and
   `[PK] 🎯 Room Code / 🌐 Server / 🔑 UDP Key`.
   The bot switches to UDP: sends HELLO, answers ACK/PING, sends the two
   startup frames, then starts `[AFK]` and `[MOVE]` workers.
7. **During the match** — `[MOVE] #n sent=…` every 30 packets; `[AFK]` logs when
   the match ends. The match counts as finished on command **103**
   (`RUDP_MATCH_END`), **107** (`RUDP_DEAD`), or after 90 s of silence.
8. **After the match** — `[MATCH #n] Closed`, EXP refresh ~9 s later, gateway
   reconnect after 6 s (`NEW_MATCH_DELAY`), and the cycle repeats.

### Dashboard buttons

| Button | Actual effect |
|---|---|
| `+ Add Account` | `POST /api/account/add` → saves account, starts worker |
| `Stop Bot` | `POST /api/account/pause_all` → sets paused flags; **does not close sockets** |
| `Restart Bot` | `POST /api/account/restart` with `{}` → **no-op** (see bugs #3) |
| `Clear` (logs) | clears in-memory log buffer |
| page refresh | auto-POSTs `clanjoin` for a hard-coded clan (bugs #5) |

### HTTP API

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | dashboard HTML |
| GET | `/api/stats` | totals, per-account objects, last 80 logs |
| GET | `/api/logs` | HTML-coloured log lines |
| POST | `/api/logs/clear` | clear logs |
| POST | `/api/account/add` | add guest `{uid,password}` or `{token}` |
| POST | `/api/account/delete` | delete account + caches + worker |
| POST | `/api/account/refresh` | refresh level/EXP |
| POST | `/api/account/pause` | pause one UID |
| POST | `/api/account/pause_all` | pause/resume all |
| POST | `/api/account/restart` | (broken) restart one/all |
| POST | `/api/clanjoin` | send clan join with **every** cached token |
| GET | `/api/guildinfo?guild_id=&region=&key=XEROX_MODS` | guild XP JSON |

## B. Account generator — `SaMIGEN.py`

1. `python SaMIGEN.py`.
2. Enter **accounts to generate** (default 10) and **worker threads** (default 5,
   max 30), press **Enter**.
3. It registers a guest → gets token → `MajorRegister` (up to 10 nickname tries)
   → `ChooseNewbieChoice` → `MajorLogin` → `GetLoginData`.
4. Each success prints a card (`Nickname / UID / Password / JWT`) and appends
   `{uid, password, nickname, open_id, access_token, account_id, jwt, …}` to
   `accounts.json`; details go to `generator.log`.
5. Stops when the target count is reached (or Ctrl+C). If the API is
   unreachable it retries forever — see risks.

## C. Guild joiner / spammer — `join_guild.py`

1. `python join_guild.py`.
2. Choose `1` Guild Join, `2` Guild Leave, `3` OWNER SPAM (join + friend
   requests to the owner).
3. Enter the account file: `.txt` lines `uid:password` (also accepts `|`, `,`,
   space) or a JSON list of `{uid,password}`.
4. Enter the **Guild ID**. For option 3 the owner UID is auto-fetched, or you
   type it manually if that fails.
5. Set workers (1–10) and per-worker delay (default 0.8 s).
6. Watch per-account result cards and the final session report. Failed accounts
   are appended in **plain text** to `AGS_JOHIR/failed_accounts.txt`.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `ImportError: cannot import name 'runtime_version'` | old protobuf → `pip install "protobuf>=6,<8"` |
| `No accounts in accounts.json` | empty file; add via dashboard or generator |
| Login retries every 15 s | version scrape or login endpoint changed |
| Bot connected but never queues | fewer than 4 accounts (bugs #4) |
| `Stop Bot` does nothing / cannot resume | pause/resume broken (bugs #2, #3) |
| Dashboard shows 0 matches started | counter never incremented (bugs #16) |
