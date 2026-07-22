# Memory Index
*First file Hermes should check when searching Obsidian memory.*

## Canonical Notes
- **[[CURRENT_TASKS]]**: Current open loops, tasks, and blockers.
- **[[DECISIONS]]**: Durable decisions with date, reason, and scope.
- **[[PROJECT_INDEX]]**: Active projects and canonical project note paths.
- **[[USER_PROFILE]]**: Stable user preferences, paths, and system rules.
- **[[HERMES_EXTERNAL_AGENT_SOURCES]]**: MAZos agent references and sources.
- **[[HERMES_MAZ_LITE_PROFILE]]**: New lightweight Hermes profile (maz-lite) — clean SOUL/memory split, redacted export, session-control skill.

## Fast-Mode & Anti-Hallucination Rules
- Use current context first.
- Read this index before searching the whole vault.
- Read only one relevant note at a time.
- Prefer repo files over memory for code truth.
- Prefer `search_files` over full file reads.
- Do not load broad skills by default; use only when explicitly matched or requested.
- Do not spawn agents unless explicitly approved.
- Never claim files, repos, cron jobs, memories, commands, emails, or deployments succeeded unless verified by tool output.
- When using Obsidian memory, cite the exact note path.
- If uncertain, state "not verified".
