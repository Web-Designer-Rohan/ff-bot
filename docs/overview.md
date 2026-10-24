# Overview

## Purpose

FF_Bot is a private game-automation toolkit for Free Fire (Android package
`com.dts.freefireth`). It does three separate jobs:

| Program | Job |
|---|---|
| `GUILD_GLORY_FIXED.zip` → `app.py` | Logs in many guest accounts, forms a 4-person squad, queues Battle Royale, plays out the UDP match, and reports EXP/guild progress on a web dashboard |
| `SaMIGEN.py` | Bulk-creates new guest accounts and appends them to `accounts.json` |
| `guild-spam&join_er.zip` → `join_guild.py` | Makes accounts join/leave a guild and spams friend requests at the guild owner |

## Key features

- **Two login modes** — guest UID + password, or a Garena access token.
- **Session cache** — `token_cache.json` avoids re-login for 75 minutes
  (`TOKEN_CACHE_TTL = 4500`, `app.py:43`).
- **Device fingerprint per account** — random phone model/GPU/IP stored in
  `devices.json` (`app.py:126`).
- **Squad automation** — one account becomes leader, creates the team, invites
  the others, everyone sends "ready" + "start match".
- **Battle Royale only** — hard-coded mode/map IDs (`BR_MODE_ID = 1`,
  `BR_MAP_ID = 1`, `app.py:46`). The "Lone Wolf" label in the dashboard is dead
  code.
- **UDP match client** — custom OICQ/tea-CBC framing, HELLO/ACK/PING replies,
  anti-AFK fire packets and fake movement (`app.py:1012–1272`).
- **Web dashboard** — `http://<host>:20333` with guild XP, uptime, live logs,
  add-account, pause buttons and hidden clan/friend endpoints.
- **EXP tracking** — polls `GetLoginData` every ~30 s and computes level
  progress from a built-in 1–100 EXP table (`dashboard_server.py:162`).
- **Guild tools** — HTTP APIs for guild XP and mass clan-join using every
  cached token (`dashboard_server.py:578–703`).

## What it is not

- No tests, no README, no config file, no CLI flags — every setting is a
  constant you edit in the source.
- Not a single-account tool in practice: the squad flow requires **4 accounts in
  the same process** or matchmaking never starts (see `bugs.md` #4).
- The dashboard does not display the account list, even though `/api/stats`
  returns it.
