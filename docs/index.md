# FF_Bot — Code Audit Docs

Beginner-friendly documentation for the Free Fire automation tools in this folder.

**Audit scope:** the shipped archives `GUILD_GLORY_FIXED.zip`, `guild-spam&join_er.zip`
and `SaMIGEN.py`. They were unpacked read-only into a temp folder and traced line by
line. Nothing in the tool was changed.

| Doc | Read this to learn |
|---|---|
| [overview.md](overview.md) | What the tool does and its key features |
| [tech-stack.md](tech-stack.md) | Languages, libraries, protocols, external APIs |
| [workflow.md](workflow.md) | Step-by-step setup and use, with expected output |
| [structure.md](structure.md) | Every file/folder and its job |
| [bugs.md](bugs.md) | Confirmed bugs vs. risks, severity, fixes |
| [upgrades.md](upgrades.md) | High-impact improvements and priorities |
| [skills.md](skills.md) | Installed coding skills and how they were verified |

**Short version:** this is a Python bot that logs into Free Fire guest accounts,
builds a 4-player squad, queues Battle Royale, then keeps the UDP match session
alive with fake movement and anti-AFK fire while a local web dashboard shows
guild XP and logs. Two extra scripts generate accounts and send guild join /
friend-request spam.

> ⚠️ It automates a live game account, breaks the game's terms of service, and
> ships with real-looking session tokens inside `token_cache.json`. Treat the
> whole folder as compromised until those credentials are rotated.
