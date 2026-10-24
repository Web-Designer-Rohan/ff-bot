# Tech Stack

## Runtime

- **Python 3.10+** (asyncio, f-strings, `asyncio.to_thread`). Shipped `.pyc`
  files are for 3.13 but are not required.
- Three entry points: `python app.py`, `python SaMIGEN.py`, `python join_guild.py`.

## Libraries

| Library | Used for | Where | Pin |
|---|---|---|---|
| `asyncio` (stdlib) | bot loops, TCP/UDP clients, dashboard | app.py | — |
| `aiohttp` | dashboard web server | dashboard_server.py | unpinned |
| `httpx` | async HTTP to Garena/login servers | app.py | unpinned |
| `requests` | sync HTTP (dashboard guild calls, generator, joiner) | dashboard_server.py, SaMIGEN.py, join_guild.py | unpinned |
| `protobuf` | generated message classes | `*_pb2.py` | **unpinned — breaks on 4.x/5.x** |
| `protobuf-decoder` | generic protobuf sniffing (`Parser`) | app.py | unpinned |
| `pycryptodome` (`Crypto`) | AES-CBC payload encryption | all scripts | unpinned |
| `google-play-scraper` | fetch current Free Fire version from Play Store | app.py:418 | unpinned |
| `PyJWT` | imported by dashboard, **never used** | dashboard_server.py:17 | unpinned |

`requirements.txt` (8 lines) lists `httpx`, `google-play-scraper`,
`pycryptodome`, `protobuf-decoder`, `protobuf`, `requests`, `aiohttp`, `PyJWT`.
It does **not** pin a single version, and `urllib3` is only present transitively.

**Verified dependency bug:** `thunderFF_pb2.py`, `RbcFF_pb2.py`,
`RbcTeam_pb2.py` and `data_pb2.py` import `google.protobuf.runtime_version`
(protobuf ≥ 5.27) and were generated with protobuf 7.35.1 / 6.33.1.
`protobuf==4.25.3` fails at import (no `runtime_version`), and `5.29.5` fails on
`data_pb2` ("mismatched Gencode/Runtime major versions"). `6.33.1` and `7.36.2`
work — so the practical floor is **protobuf 6**.

## Protocols and crypto

| Piece | Detail | Location |
|---|---|---|
| Login payload cipher | AES-128-CBC, PKCS#7, fixed key/IV | app.py:207–208, 565 |
| TCP gateway frame | 2-byte msg key + CRC7 + XOR-obfuscated header, tea-CBC body | app.py:879–980 |
| TEA | 16 rounds, delta `0x9E3779B9`, custom CBC | app.py:879–920 |
| CRC7 | 256-byte lookup table | app.py:210–222, 936 |
| UDP match frame | same header scheme, `sv_frame`/`build_packet` | app.py:941–961 |
| Protobuf | hand-built varint/length encoding + generated `_pb2` parsers | app.py:541–596 |
| JWT split | login token split at **character 660** into `thunder` / `sharma` | app.py:1087 |

## External services

| Service | Reason |
|---|---|
| `100067.connect.garena.com` | OAuth guest register / token grant / token inspect |
| `loginbp.ppmainecoonghj.com` | `MajorLogin`, `GetLoginData`, `Ping`, `ChooseNewbieChoice` |
| `version.ggwhitehawk.com` | remote version + server URL lookup |
| `client*.freefiremobile.com`, `clientbp.*` | game gateway, clan, friend APIs |
| Google Play | version scrape |
| `danger-guild-management-web.vercel.app` (joiner) | third-party guild join/leave backend |
| `ags-community-jwt-api.vercel.app` (joiner) | third-party JWT provider |

## Secrets and data files

- `token_cache.json` — 7 cached accounts with JWT, `access_token`, AES key/IV.
- `devices.json` — per-account fake device profile, including client IP.
- Hard-coded `CLIENT_SECRET` (`SaMIGEN.py:29`) and a dashboard API key
  `XEROX_MODS` (`dashboard_server.py:653`, `index.html:580`).

Version strings diverge across files (`OB54` vs `OB55`, `1.130.22`, `1.132.6`,
`1.132.9`, `2.133.9`), so a game update usually requires several edits.
