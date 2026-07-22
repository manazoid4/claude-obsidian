# Hermes maz-lite Profile

New lightweight Hermes profile created 2026-07-08 to replace the flat SOUL/MEMORY/USER export as Maz's daily-driver memory system. See [[MEMORY_INDEX]] for how Hermes routes memory generally — this note is the pointer between that system and the Hermes-side profile files.

## Where it lives
`C:\Users\manaz\AppData\Local\hermes\profiles\maz-lite\`

## What's in it
- `SOUL.md` — identity, points at `memories/MEMORY_INDEX.md`
- `memories/USER.md`, `memories/MEMORY.md` — small, auto-loaded every turn
- `memories/{USER_PROFILE,MEMORY_INDEX,HERMES_RULES,PROJECTS_INDEX,GITHUB_REPOS_INDEX,LOOP_LIBRARY,PROMPT_INSPIRATION,WORKFLOWS,DECISIONS,EXPORT_POLICY,SESSION_CONTROL}.md` — on-demand, read only when relevant
- `skills/profile-export-v2/` — redacted export skill (supersedes the old raw SOUL/MEMORY/USER zip export)
- `skills/session-control/` — session cleanup/compact/close skill (dry-run by default)

The profile's own `PROJECTS_INDEX.md`/`GITHUB_REPOS_INDEX.md` are a fast local cache — [[PROJECT_INDEX]] and [[DECISIONS]] in this vault remain canonical.

## Original profile
Untouched at `C:\Users\manaz\AppData\Local\hermes\` (the default profile). Backed up before any of this work at `C:\Users\manaz\AppData\Local\hermes\backups\maz-lite-setup-2026-07-08-0030\`.

## Activation
Not yet made the sticky active profile — that switch (`hermes profile use maz-lite`) is a deliberate next step left for Maz to run, so it doesn't change what the live gateway session was serving mid-setup. See the session summary in `04-SESSIONS\` for exact commands.
