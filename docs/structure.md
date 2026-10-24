# Structure

```
FF_Bot/
├── GUILD_GLORY_FIXED.zip        main bot package (not extracted in the repo)
├── guild-spam&join_er.zip       standalone guild join / owner-spam CLI
├── SaMIGEN.py                   bulk guest-account generator
├── skills-lock.json             agent-skill lockfile (not part of the bot)
├── .agents/skills/              agent skills (not part of the bot)
└── docs/                        this audit
```

## Inside `GUILD_GLORY_FIXED.zip`

| File / folder | Role |
|---|---|
| `app.py` (2,500 lines) | **Core engine.** Config constants, device spoofing, OAuth/MajorLogin/GetLoginData, TCP gateway client, squad coordinator, matchmaking, UDP match client, movement/anti-AFK, friend/team packet handlers, account loops |
| `dashboard_server.py` (1,022) | aiohttp dashboard server, shared `bot_state`, EXP table, account add/delete/pause, guild XP + clan-join APIs |
| `SAMI_CODEX/index.html` | Single-page dashboard (dark UI, guild card, uptime, logs, add-account modal) |
| `message_ids.py` (1,884) | `MESSAGE_ID_TO_NAME` map: UDP/RUDP command IDs → names |
| `thunderFF_pb2.py` | Generated protobuf: `MajorLoginReq/Res`, `GetLoginDataRes`, `StartMatch` |
| `RbcFF_pb2.py`, `RbcTeam_pb2.py` | **Byte-identical copies** of `thunderFF_pb2.py`; imported but unused |
| `StartMatch_pb2.py` | Generated `StartUDPMatch`; **never imported** |
| `data_pb2.py` | Clan-info response schema (`response`) |
| `ReqCLan_pb2.py` | `MyMessage` for join-clan request |
| `encode_id_clan_pb2.py` | `MyData` for fetch-clan-by-ID request |
| `my_pb2.py`, `output_pb2.py` | Old protobuf schemas; only `output_pb2` is imported (unused) |
| `decode.py` | Standalone hex/UTF-8/strings/JSON/protobuf inspector; hard-coded sample hex |
| `encode-id-clan.py` | Standalone AES encryptor; hard-coded clan ID `3034881538` |
| `requirements.txt` | 8 Python packages, no versions |
| `accounts.json` | Input accounts (shipped as `[]`) |
| `token_cache.json` | **Sensitive** session cache: 7 accounts with JWT, access token, AES key/IV |
| `devices.json` | Per-account fake device fingerprints |
| `guild_exp_cache.json` | Guild XP baselines for "gained" calculation |
| `__pycache__/` | Stale 3.13 bytecode; delete freely |
| `SAMI_CODEX/` | Dashboard template folder |

## Inside `guild-spam&join_er.zip`

| File | Role |
|---|---|
| `join_guild.py` (627) | CLI menu, themed terminal UI, join/leave via third-party API, owner-spam via JWT API + friend-request packets |
| `my_pb2.py`, `output_pb2.py` | Imported in a `try/except` but never used |

## Data flow

```
accounts.json ──► app.py login ──► token_cache.json / devices.json
                     │
                     ├─ TCP gateway (functional_addrs) ──► squad + matchmaking
                     │                                     └─ UDP match (server_ip:port)
                     └─ TCP chat (informational_addrs) ──► keep-alive / social packets
                     │
dashboard_server.bot_state ◄── app.py status/EXP ──► index.html (poll 10 s)
```

## Dead or redundant files

- `RbcFF_pb2.py`, `RbcTeam_pb2.py` — same content as `thunderFF_pb2.py`
  (identical MD5), add import cost and confusion.
- `StartMatch_pb2.py` — correct schema for the match-search packet, but
  `app.py:832` references a non-existent `CguFF_pb2` instead.
- Duplicate `handle_index` in `dashboard_server.py:181` and `:567`.
- `jwt`, `csv`, `my_pb2`, `output_pb2`, `base64`, `Rbc*_pb2` imports unused.
