# Hermes maz-lite One-Shot Fix — 2026-07-08

## What happened
Built a clean, lightweight Hermes profile (`maz-lite`) to replace the flat SOUL/MEMORY/USER export as Maz's daily-driver memory system, without touching the existing default profile or the existing canonical `03-MEMORY` vault system.

## Verified facts used
- Hermes source: `NousResearch/hermes-agent`, cloned locally at `C:\Users\manaz\AppData\Local\hermes\hermes-agent`, real `hermes profile` CLI + skills system (not invented).
- GitHub repos: `gh repo list manazoid4` (15 repos, see `PROJECTS_INDEX.md` / `GITHUB_REPOS_INDEX.md` in the maz-lite profile).
- Broken cron job found: `6071da785394` "Daily Hermes session HTML email" — already `enabled:false`/`state:paused` due to repeated `RuntimeError: Connection error`. Renamed to mark legacy, left disabled. New v2 markdown-first job added (`6071da785395`), also disabled by default.

## Full report
See the assistant's final chat message in this session for the complete "Hermes maz-lite One-Shot Fix Report" with backups, verification results, and exact next commands.

## Not done (explicit safety rules)
- Did not switch the active profile to maz-lite (reversible next step for Maz).
- Did not send any real email.
- Did not push/commit anything to GitHub (including this vault).
- Did not close or touch any other Hermes session.
- Did not rebase/update the hermes-agent repo itself (it's 28 commits behind upstream — separate task if wanted).
